# Error Handling Guide

## Overview

This guide provides comprehensive information about error handling in the SpendWise API, including error formats, status codes, and best practices for handling errors in your application.

## Error Response Format

All error responses follow a consistent JSON structure to make error handling predictable and straightforward.

### Single Error Format
```json
{
  "error": "Error message description",
  "status": 400
}
```

### Validation Errors Format
```json
{
  "errors": {
    "field_name": ["error message 1", "error message 2"],
    "another_field": ["error message"]
  },
  "status": 422
}
```

### Authentication Error Format
```json
{
  "status": {
    "code": 401,
    "message": "You need to sign in or sign up before continuing."
  }
}
```

## HTTP Status Codes

| Status Code | Name | Description | Common Causes |
|-------------|------|-------------|---------------|
| 200 | OK | Request succeeded | Successful GET, PATCH requests |
| 201 | Created | Resource created successfully | Successful POST requests |
| 204 | No Content | Request succeeded, no content to return | Successful DELETE requests |
| 400 | Bad Request | Invalid request format | Malformed JSON, missing required headers |
| 401 | Unauthorized | Missing or invalid authentication | No token, expired token, invalid credentials |
| 403 | Forbidden | Not authorized for this resource | Trying to access another user's data |
| 404 | Not Found | Resource doesn't exist | Invalid ID, deleted resource |
| 422 | Unprocessable Entity | Validation failed | Invalid field values, constraint violations |
| 500 | Internal Server Error | Server error occurred | Unexpected server errors, database issues |

## Error Categories

### 1. Authentication Errors (401)

#### Missing Token
```json
{
  "error": "You need to sign in or sign up before continuing.",
  "status": 401
}
```

**Cause:** No Authorization header provided  
**Solution:** Include `Authorization: Bearer <token>` header

**Example:**
```javascript
// ❌ Wrong
fetch('/api/v1/budgets');

// ✅ Correct
fetch('/api/v1/budgets', {
  headers: {
    'Authorization': `Bearer ${token}`
  }
});
```

#### Invalid Credentials
```json
{
  "status": {
    "code": 401,
    "message": "Invalid email or password."
  }
}
```

**Cause:** Wrong email or password during login  
**Solution:** Verify credentials and try again

#### Expired Token
```json
{
  "error": "Signature has expired",
  "status": 401
}
```

**Cause:** JWT token has expired  
**Solution:** Login again to get a new token

#### Invalid Token Signature
```json
{
  "error": "Signature verification failed",
  "status": 401
}
```

**Cause:** Token was tampered with or uses wrong secret  
**Solution:** Login again to get a valid token

#### Revoked Token
```json
{
  "error": "Token has been revoked",
  "status": 401
}
```

**Cause:** User logged out, invalidating the token  
**Solution:** Login again

---

### 2. Authorization Errors (403)

#### Unauthorized Access
```json
{
  "error": "You are not authorized to access this resource",
  "status": 403
}
```

**Cause:** Trying to access another user's data  
**Solution:** Only access your own resources

**Example Scenario:**
- User A trying to access User B's budget
- User trying to modify admin-only resources

---

### 3. Not Found Errors (404)

#### Resource Not Found
```json
{
  "error": "Budget not found",
  "status": 404
}
```

**Common Scenarios:**
```json
{
  "error": "Transaction not found",
  "status": 404
}
```

```json
{
  "error": "Category not found",
  "status": 404
}
```

**Causes:**
- Invalid ID provided
- Resource was deleted
- Resource doesn't belong to the user

**Solution:** Verify the ID exists and belongs to authenticated user

---

### 4. Validation Errors (422)

#### User Registration Errors
```json
{
  "errors": {
    "email": [
      "can't be blank",
      "has already been taken"
    ],
    "password": [
      "is too short (minimum is 6 characters)"
    ],
    "name": [
      "can't be blank"
    ]
  },
  "status": 422
}
```

#### Budget Validation Errors
```json
{
  "errors": {
    "name": ["can't be blank"],
    "financial_goal": ["must be greater than 0"]
  },
  "status": 422
}
```

#### Transaction Validation Errors
```json
{
  "errors": {
    "amount": ["must be greater than 0"],
    "date": ["can't be blank"],
    "category_id": ["must exist"],
    "budget_id": ["must exist"]
  },
  "status": 422
}
```

#### Category Validation Errors
```json
{
  "errors": {
    "name": [
      "can't be blank",
      "has already been taken"
    ]
  },
  "status": 422
}
```

#### Delete Constraint Error
```json
{
  "error": "Cannot delete category with associated transactions",
  "status": 422
}
```

**Cause:** Trying to delete a category that has transactions  
**Solution:** Delete or reassign transactions first

---

### 5. Bad Request Errors (400)

#### Malformed JSON
```json
{
  "error": "Invalid JSON format",
  "status": 400
}
```

**Cause:** Invalid JSON in request body  
**Example:**
```javascript
// ❌ Wrong - missing quotes
{"name": missing}

// ✅ Correct
{"name": "value"}
```

#### Missing Content-Type
```json
{
  "error": "Content-Type must be application/json",
  "status": 400
}
```

**Solution:**
```javascript
fetch('/api/v1/budgets', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${token}`
  },
  body: JSON.stringify(data)
});
```

---

### 6. Server Errors (500)

#### Internal Server Error
```json
{
  "error": "Internal server error",
  "status": 500
}
```

**Common Causes:**
- Database connection issues
- Unexpected code errors
- Configuration problems

**What to do:**
1. Check server logs
2. Verify database connectivity
3. Report to development team if persistent

---

## Error Handling Best Practices

### 1. Client-Side Error Handling

#### Using Fetch API
```javascript
fetch('/api/v1/budgets', {
  headers: {
    'Authorization': `Bearer ${token}`
  }
})
.then(response => {
  if (!response.ok) {
    return response.json().then(err => {
      throw new Error(err.error || 'Request failed');
    });
  }
  return response.json();
})
.then(data => {
  // Handle success
  console.log(data);
})
.catch(error => {
  // Handle error
  console.error('Error:', error.message);
  displayError(error.message);
});
```

#### Using Axios
```javascript
import axios from 'axios';

const api = axios.create({
  baseURL: 'http://localhost:3000'
});

// Response interceptor for error handling
api.interceptors.response.use(
  response => response,
  error => {
    const status = error.response?.status;
    const data = error.response?.data;
    
    switch(status) {
      case 401:
        // Handle authentication errors
        handleAuthError(data);
        break;
      case 403:
        // Handle authorization errors
        handleForbidden(data);
        break;
      case 404:
        // Handle not found
        handleNotFound(data);
        break;
      case 422:
        // Handle validation errors
        handleValidationErrors(data.errors);
        break;
      case 500:
        // Handle server errors
        handleServerError();
        break;
      default:
        handleGenericError(data);
    }
    
    return Promise.reject(error);
  }
);
```

### 2. Validation Error Display

#### React Example
```javascript
function BudgetForm() {
  const [errors, setErrors] = useState({});
  
  const handleSubmit = async (data) => {
    try {
      await api.post('/api/v1/budgets', { budget: data });
      // Success
    } catch (error) {
      if (error.response?.status === 422) {
        setErrors(error.response.data.errors);
      }
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input name="name" />
      {errors.name && (
        <div className="error">
          {errors.name.join(', ')}
        </div>
      )}
      
      <input name="financial_goal" type="number" />
      {errors.financial_goal && (
        <div className="error">
          {errors.financial_goal.join(', ')}
        </div>
      )}
    </form>
  );
}
```

### 3. Global Error Handler

```javascript
class ApiError extends Error {
  constructor(status, message, errors = null) {
    super(message);
    this.status = status;
    this.errors = errors;
  }
}

function handleApiError(error) {
  if (error.response) {
    const { status, data } = error.response;
    
    switch(status) {
      case 401:
        // Redirect to login
        localStorage.removeItem('jwt_token');
        window.location.href = '/login';
        break;
        
      case 403:
        showNotification('You do not have permission to perform this action', 'error');
        break;
        
      case 404:
        showNotification('Resource not found', 'error');
        break;
        
      case 422:
        // Display validation errors
        if (data.errors) {
          Object.entries(data.errors).forEach(([field, messages]) => {
            showNotification(`${field}: ${messages.join(', ')}`, 'error');
          });
        }
        break;
        
      case 500:
        showNotification('Server error. Please try again later.', 'error');
        break;
        
      default:
        showNotification(data.error || 'An error occurred', 'error');
    }
  } else if (error.request) {
    // Network error
    showNotification('Network error. Please check your connection.', 'error');
  } else {
    showNotification('An unexpected error occurred', 'error');
  }
}

// Usage
api.post('/api/v1/budgets', data)
  .then(response => handleSuccess(response))
  .catch(error => handleApiError(error));
```

### 4. User-Friendly Error Messages

Map technical errors to user-friendly messages:

```javascript
const ERROR_MESSAGES = {
  // Authentication
  'Invalid email or password': 'The email or password you entered is incorrect. Please try again.',
  'Signature has expired': 'Your session has expired. Please log in again.',
  'You need to sign in': 'Please log in to continue.',
  
  // Validation
  "can't be blank": 'This field is required.',
  'has already been taken': 'This value is already in use.',
  'is too short': 'This value is too short.',
  'must be greater than 0': 'Please enter a positive number.',
  
  // Not Found
  'Budget not found': 'The budget you\'re looking for doesn\'t exist or has been deleted.',
  'Transaction not found': 'The transaction you\'re looking for doesn\'t exist.',
  'Category not found': 'The category you\'re looking for doesn\'t exist.',
  
  // Authorization
  'not authorized': 'You don\'t have permission to perform this action.',
  
  // Server
  'Internal server error': 'Something went wrong on our end. Please try again later.'
};

function getUserFriendlyMessage(error) {
  for (const [key, message] of Object.entries(ERROR_MESSAGES)) {
    if (error.includes(key)) {
      return message;
    }
  }
  return error; // Fallback to original message
}
```

---

## Complete Error Handling Example

### React Component with Full Error Handling

```javascript
import React, { useState, useEffect } from 'react';
import axios from 'axios';

const api = axios.create({
  baseURL: 'http://localhost:3000',
  headers: {
    'Content-Type': 'application/json'
  }
});

// Add token to requests
api.interceptors.request.use(config => {
  const token = localStorage.getItem('jwt_token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

function BudgetManager() {
  const [budgets, setBudgets] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [validationErrors, setValidationErrors] = useState({});

  // Fetch budgets with error handling
  const fetchBudgets = async () => {
    setLoading(true);
    setError(null);
    
    try {
      const response = await api.get('/api/v1/budgets');
      setBudgets(response.data.data);
    } catch (err) {
      handleError(err);
    } finally {
      setLoading(false);
    }
  };

  // Create budget with validation error handling
  const createBudget = async (budgetData) => {
    setLoading(true);
    setError(null);
    setValidationErrors({});
    
    try {
      const response = await api.post('/api/v1/budgets', {
        budget: budgetData
      });
      setBudgets([...budgets, response.data.data]);
      return true;
    } catch (err) {
      handleError(err);
      return false;
    } finally {
      setLoading(false);
    }
  };

  // Delete budget with error handling
  const deleteBudget = async (id) => {
    setLoading(true);
    setError(null);
    
    try {
      await api.delete(`/api/v1/budgets/${id}`);
      setBudgets(budgets.filter(b => b.id !== id));
      return true;
    } catch (err) {
      handleError(err);
      return false;
    } finally {
      setLoading(false);
    }
  };

  // Centralized error handler
  const handleError = (err) => {
    if (!err.response) {
      setError('Network error. Please check your connection.');
      return;
    }

    const { status, data } = err.response;

    switch(status) {
      case 401:
        localStorage.removeItem('jwt_token');
        window.location.href = '/login';
        break;
        
      case 403:
        setError('You do not have permission to perform this action.');
        break;
        
      case 404:
        setError('The resource you\'re looking for was not found.');
        break;
        
      case 422:
        if (data.errors) {
          setValidationErrors(data.errors);
          setError('Please fix the validation errors.');
        } else {
          setError(data.error || 'Validation failed.');
        }
        break;
        
      case 500:
        setError('Server error. Please try again later.');
        break;
        
      default:
        setError(data.error || 'An error occurred.');
    }
  };

  useEffect(() => {
    fetchBudgets();
  }, []);

  if (loading) return <div>Loading...</div>;

  return (
    <div>
      {error && (
        <div className="error-banner" role="alert">
          {error}
        </div>
      )}
      
      <BudgetForm 
        onSubmit={createBudget}
        validationErrors={validationErrors}
      />
      
      <BudgetList 
        budgets={budgets}
        onDelete={deleteBudget}
      />
    </div>
  );
}

export default BudgetManager;
```

---

## Error Logging and Monitoring

### Client-Side Logging

```javascript
function logError(error, context = {}) {
  const errorLog = {
    timestamp: new Date().toISOString(),
    message: error.message,
    status: error.response?.status,
    url: error.config?.url,
    method: error.config?.method,
    context,
    userAgent: navigator.userAgent
  };
  
  // Log to console in development
  if (process.env.NODE_ENV === 'development') {
    console.error('Error Log:', errorLog);
  }
  
  // Send to error tracking service (e.g., Sentry)
  // Sentry.captureException(error, { contexts: { custom: errorLog } });
  
  // Send to your backend for logging
  // api.post('/api/v1/error-logs', errorLog).catch(() => {});
}

// Usage
api.get('/api/v1/budgets')
  .catch(error => {
    logError(error, { 
      action: 'fetch_budgets',
      userId: currentUser.id
    });
    handleError(error);
  });
```

---

## Testing Error Scenarios

### Manual Testing Checklist

#### Authentication Errors
- [ ] Request without token
- [ ] Request with expired token
- [ ] Request with invalid token
- [ ] Login with wrong credentials
- [ ] Signup with existing email

#### Authorization Errors
- [ ] Access another user's budget
- [ ] Modify another user's transaction
- [ ] Delete another user's category

#### Validation Errors
- [ ] Create budget without name
- [ ] Create budget with negative goal
- [ ] Create transaction without amount
- [ ] Create transaction with invalid date
- [ ] Create transaction with non-existent category

#### Not Found Errors
- [ ] Get budget with invalid ID
- [ ] Update non-existent transaction
- [ ] Delete non-existent category

---

## Debugging Tips

### 1. Check Request Headers
```bash
curl -v http://localhost:3000/api/v1/budgets \
  -H "Authorization: Bearer token"
```

### 2. Verify Token
- Decode JWT at [jwt.io](https://jwt.io)
- Check expiration time
- Verify signature

### 3. Inspect Response
```javascript
axios.get('/api/v1/budgets')
  .catch(error => {
    console.log('Status:', error.response?.status);
    console.log('Data:', error.response?.data);
    console.log('Headers:', error.response?.headers);
  });
```

### 4. Check Rails Logs
```bash
# In development
tail -f log/development.log

# Look for:
# - SQL queries
# - Controller actions
# - Error stack traces
```

---

## Error Prevention

### 1. Input Validation
```javascript
function validateBudgetData(data) {
  const errors = {};
  
  if (!data.name || data.name.trim() === '') {
    errors.name = ['Name is required'];
  }
  
  if (!data.financial_goal || data.financial_goal <= 0) {
    errors.financial_goal = ['Must be greater than 0'];
  }
  
  return {
    isValid: Object.keys(errors).length === 0,
    errors
  };
}

// Use before API call
const { isValid, errors } = validateBudgetData(formData);
if (!isValid) {
  setValidationErrors(errors);
  return;
}

// Proceed with API call
createBudget(formData);
```

### 2. Token Management
```javascript
// Check token expiration before requests
function isTokenExpired(token) {
  try {
    const payload = JSON.parse(atob(token.split('.')[1]));
    return payload.exp * 1000 < Date.now();
  } catch {
    return true;
  }
}

// Refresh token if needed
api.interceptors.request.use(async config => {
  const token = localStorage.getItem('jwt_token');
  
  if (token && isTokenExpired(token)) {
    // Redirect to login
    window.location.href = '/login';
    return Promise.reject('Token expired');
  }
  
  config.headers.Authorization = `Bearer ${token}`;
  return config;
});
```

### 3. Defensive Coding
```javascript
// Always check if data exists
const budget = response.data?.data;
if (!budget) {
  throw new Error('Invalid response format');
}

// Use optional chaining
const budgetName = budget?.attributes?.name ?? 'Unnamed Budget';

// Handle edge cases
const transactions = budget.relationships?.transactions?.data || [];
```

---

## Quick Reference

### Status Code Quick Lookup
```
401 → Check authentication (token)
403 → Check authorization (permissions)
404 → Check resource ID
422 → Check input validation
500 → Check server logs
```

### Common Fixes
```
Missing token → Add Authorization header
Expired token → Login again
Wrong credentials → Verify email/password
Validation error → Fix input data
Not found → Verify resource ID
Server error → Contact support
```

---

## Additional Resources

- [HTTP Status Codes](https://httpstatuses.com/)
- [REST API Error Handling Best Practices](https://www.baeldung.com/rest-api-error-handling-best-practices)
- [Rails Error Handling](https://guides.rubyonrails.org/active_record_validations.html)
- [Axios Error Handling](https://axios-http.com/docs/handling_errors)