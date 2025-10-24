# Authentication Guide

## Overview

SpendWise API uses **JWT (JSON Web Token)** authentication powered by the Devise gem with JWT extension. This guide covers everything you need to know about authentication.

## How It Works

1. **Register/Login**: User sends credentials to `/signup` or `/login`
2. **Receive Token**: Server validates credentials and returns a JWT token
3. **Include Token**: Client includes token in `Authorization` header for subsequent requests
4. **Token Validation**: Server validates token on each protected endpoint

## Authentication Flow

```
┌─────────┐                          ┌─────────┐
│ Client  │                          │ Server  │
└────┬────┘                          └────┬────┘
     │                                    │
     │  POST /login                       │
     │  {email, password}                 │
     ├───────────────────────────────────>│
     │                                    │
     │                    Validate User   │
     │                    Generate JWT    │
     │                                    │
     │  200 OK                            │
     │  {user, token}                     │
     │<───────────────────────────────────┤
     │                                    │
     │  Store Token Locally               │
     │                                    │
     │  GET /api/v1/budgets               │
     │  Authorization: Bearer <token>     │
     ├───────────────────────────────────>│
     │                                    │
     │                    Verify Token    │
     │                    Return Data     │
     │                                    │
     │  200 OK                            │
     │  {budgets: [...]}                  │
     │<───────────────────────────────────┤
     │                                    │
```

## JWT Token Structure

A JWT token consists of three parts separated by dots:

```
eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJleHAiOjE2OTI4MDAwMDB9.signature
│                    │                                        │
│     Header         │          Payload                      │  Signature
```

### Header
Contains the algorithm used (HS256):
```json
{
  "alg": "HS256"
}
```

### Payload
Contains user information and expiration:
```json
{
  "user_id": 1,
  "exp": 1692800000,
  "jti": "unique-token-id"
}
```

### Signature
Cryptographic signature to verify token authenticity.

## Registration (Signup)

### Endpoint
```
POST /signup
```

### Request Example
```bash
curl -X POST http://localhost:3000/signup \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "email": "john@example.com",
      "password": "SecurePass123!",
      "name": "John Doe"
    }
  }'
```

### Response (Success)
```json
{
  "status": {
    "code": 201,
    "message": "Signed up successfully."
  },
  "data": {
    "id": 1,
    "email": "john@example.com",
    "name": "John Doe",
    "role": "user"
  },
  "token": "eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxfQ.signature"
}
```

### Password Requirements
- Minimum 6 characters
- No specific complexity requirements (configurable in Devise)

## Login

### Endpoint
```
POST /login
```

### Request Example
```bash
curl -X POST http://localhost:3000/login \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "email": "john@example.com",
      "password": "SecurePass123!"
    }
  }'
```

### Response (Success)
```json
{
  "status": {
    "code": 200,
    "message": "Logged in successfully."
  },
  "data": {
    "id": 1,
    "email": "john@example.com",
    "name": "John Doe",
    "role": "user"
  },
  "token": "eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxfQ.signature"
}
```

### Response (Failure)
```json
{
  "status": {
    "code": 401,
    "message": "Invalid email or password."
  }
}
```

## Using the Token

### Including Token in Requests

All protected endpoints require the JWT token in the `Authorization` header:

```
Authorization: Bearer <your-jwt-token>
```

### Example Request
```bash
curl -X GET http://localhost:3000/api/v1/budgets \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxfQ.signature"
```

### JavaScript Example
```javascript
// Store token after login
const token = response.data.token;
localStorage.setItem('jwt_token', token);

// Use token in subsequent requests
fetch('http://localhost:3000/api/v1/budgets', {
  headers: {
    'Authorization': `Bearer ${localStorage.getItem('jwt_token')}`,
    'Content-Type': 'application/json'
  }
})
.then(response => response.json())
.then(data => console.log(data));
```

### React Example with Axios
```javascript
import axios from 'axios';

// Configure axios instance
const api = axios.create({
  baseURL: 'http://localhost:3000',
  headers: {
    'Content-Type': 'application/json'
  }
});

// Add token to all requests
api.interceptors.request.use(config => {
  const token = localStorage.getItem('jwt_token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// Usage
api.get('/api/v1/budgets')
  .then(response => console.log(response.data))
  .catch(error => console.error(error));
```

## Logout

### Endpoint
```
DELETE /logout
```

### Request Example
```bash
curl -X DELETE http://localhost:3000/logout \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxfQ.signature"
```

### Response (Success)
```json
{
  "status": {
    "code": 200,
    "message": "Logged out successfully."
  }
}
```

### Client-Side Logout
```javascript
// Remove token from storage
localStorage.removeItem('jwt_token');

// Optionally call logout endpoint
fetch('http://localhost:3000/logout', {
  method: 'DELETE',
  headers: {
    'Authorization': `Bearer ${token}`
  }
});
```

## Token Expiration

### Default Expiration
JWT tokens typically expire after a set period (e.g., 24 hours, 7 days). The expiration is configured in the Rails application.

### Handling Expired Tokens

When a token expires, the API returns:
```json
{
  "error": "Signature has expired",
  "status": 401
}
```

### Recommended Approach
```javascript
api.interceptors.response.use(
  response => response,
  error => {
    if (error.response?.status === 401) {
      // Token expired or invalid
      localStorage.removeItem('jwt_token');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);
```

## Security Best Practices

### 1. Token Storage
**Browser Applications:**
- ✅ **Recommended**: `localStorage` or `sessionStorage`
- ⚠️ **Avoid**: Cookies (unless httpOnly and secure)

**Mobile Applications:**
- ✅ Use secure storage (Keychain on iOS, Keystore on Android)

### 2. Token Transmission
- ✅ Always use HTTPS in production
- ✅ Never log or expose tokens in URLs
- ✅ Include tokens only in Authorization header

### 3. Token Handling
```javascript
// Good ✅
const token = localStorage.getItem('jwt_token');
if (token) {
  headers.Authorization = `Bearer ${token}`;
}

// Bad ❌
const url = `http://api.example.com/data?token=${token}`;
```

### 4. Logout Cleanup
```javascript
// Complete logout
function logout() {
  // 1. Remove token from storage
  localStorage.removeItem('jwt_token');
  
  // 2. Call logout endpoint
  api.delete('/logout');
  
  // 3. Clear any user state
  clearUserState();
  
  // 4. Redirect to login
  window.location.href = '/login';
}
```

## Common Authentication Errors

### 1. Missing Token
**Error:**
```json
{
  "error": "You need to sign in or sign up before continuing.",
  "status": 401
}
```
**Solution:** Include `Authorization: Bearer <token>` header

### 2. Invalid Token Format
**Error:**
```json
{
  "error": "Invalid token format",
  "status": 401
}
```
**Solution:** Ensure token format is `Bearer <token>` (note the space)

### 3. Expired Token
**Error:**
```json
{
  "error": "Signature has expired",
  "status": 401
}
```
**Solution:** Login again to get a new token

### 4. Invalid Signature
**Error:**
```json
{
  "error": "Signature verification failed",
  "status": 401
}
```
**Solution:** Token was tampered with or incorrect secret key

### 5. Revoked Token
**Error:**
```json
{
  "error": "Token has been revoked",
  "status": 401
}
```
**Solution:** User logged out, login again

## Testing Authentication

### Using Postman

1. **Login Request:**
   - Send POST to `/login` with credentials
   - Copy the `token` from response

2. **Set Environment Variable:**
   - Create variable `jwt_token`
   - Paste the token value

3. **Configure Collection:**
   - Set Auth type to "Bearer Token"
   - Use `{{jwt_token}}` as the token value

4. **Automatic Token Storage:**
   ```javascript
   // Add to Login request "Tests" tab
   if (pm.response.code === 200) {
       var jsonData = pm.response.json();
       pm.environment.set("jwt_token", jsonData.token);
   }
   ```

### Using cURL

```bash
# 1. Login and save token
TOKEN=$(curl -s -X POST http://localhost:3000/login \
  -H "Content-Type: application/json" \
  -d '{"user":{"email":"user@example.com","password":"password"}}' \
  | jq -r '.token')

# 2. Use token in subsequent requests
curl -X GET http://localhost:3000/api/v1/budgets \
  -H "Authorization: Bearer $TOKEN"
```

## Environment-Specific Configuration

### Development
```ruby
# config/environments/development.rb
config.jwt_expiration_hours = 24
```

### Production
```ruby
# config/environments/production.rb
config.jwt_expiration_hours = 168 # 7 days
config.force_ssl = true
```

### Environment Variables
```bash
# .env
JWT_SECRET_KEY=your-super-secret-key-here
```

## Troubleshooting Checklist

- [ ] Token is present in Authorization header
- [ ] Header format is `Bearer <token>` (with space)
- [ ] Token is not expired
- [ ] Using correct base URL
- [ ] CORS is properly configured
- [ ] Content-Type header is set
- [ ] Using HTTPS in production

## Additional Resources

- [JWT.io](https://jwt.io/) - Decode and verify JWT tokens
- [Devise Documentation](https://github.com/heartcombo/devise)
- [Devise-JWT Documentation](https://github.com/waiting-for-dev/devise-jwt)