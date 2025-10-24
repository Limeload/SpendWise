# SpendWise API Documentation

## Overview

The SpendWise API is a RESTful API built with Ruby on Rails that provides endpoints for managing personal finances, including budgets, transactions, and categories.

**Base URL:** `http://localhost:3000` (development)  
**API Version:** v1  
**Authentication:** JWT (JSON Web Token)

## Table of Contents

1. [Authentication](#authentication)
2. [Endpoints](#endpoints)
   - [Auth Endpoints](#auth-endpoints)
   - [Budget Endpoints](#budget-endpoints)
   - [Transaction Endpoints](#transaction-endpoints)
   - [Category Endpoints](#category-endpoints)
3. [Error Handling](#error-handling)
4. [Rate Limiting](#rate-limiting)

---

## Authentication

The API uses JWT (JSON Web Token) for authentication. After successful login or signup, you'll receive a token that must be included in subsequent requests.

### Including the Token

Include the JWT token in the `Authorization` header:

```
Authorization: Bearer <your-jwt-token>
```

### Token Expiration

Tokens expire after a set period. When a token expires, you'll receive a `401 Unauthorized` response and need to login again.

---

## Endpoints

### Auth Endpoints

#### 1. User Registration (Signup)

**Endpoint:** `POST /signup`  
**Authentication:** Not required

**Request Body:**
```json
{
  "user": {
    "email": "user@example.com",
    "password": "securePassword123",
    "name": "John Doe"
  }
}
```

**Success Response (201 Created):**
```json
{
  "status": {
    "code": 201,
    "message": "Signed up successfully."
  },
  "data": {
    "id": 1,
    "email": "user@example.com",
    "name": "John Doe",
    "role": "user"
  },
  "token": "eyJhbGciOiJIUzI1NiJ9..."
}
```

**Error Response (422 Unprocessable Entity):**
```json
{
  "status": {
    "code": 422,
    "message": "User couldn't be created successfully."
  },
  "errors": {
    "email": ["has already been taken"],
    "password": ["is too short (minimum is 6 characters)"]
  }
}
```

**cURL Example:**
```bash
curl -X POST http://localhost:3000/signup \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "email": "user@example.com",
      "password": "securePassword123",
      "name": "John Doe"
    }
  }'
```

---

#### 2. User Login

**Endpoint:** `POST /login`  
**Authentication:** Not required

**Request Body:**
```json
{
  "user": {
    "email": "user@example.com",
    "password": "securePassword123"
  }
}
```

**Success Response (200 OK):**
```json
{
  "status": {
    "code": 200,
    "message": "Logged in successfully."
  },
  "data": {
    "id": 1,
    "email": "user@example.com",
    "name": "John Doe",
    "role": "user"
  },
  "token": "eyJhbGciOiJIUzI1NiJ9..."
}
```

**Error Response (401 Unauthorized):**
```json
{
  "status": {
    "code": 401,
    "message": "Invalid email or password."
  }
}
```

**cURL Example:**
```bash
curl -X POST http://localhost:3000/login \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "email": "user@example.com",
      "password": "securePassword123"
    }
  }'
```

---

#### 3. User Logout

**Endpoint:** `DELETE /logout`  
**Authentication:** Required

**Request Headers:**
```
Authorization: Bearer <your-jwt-token>
```

**Success Response (200 OK):**
```json
{
  "status": {
    "code": 200,
    "message": "Logged out successfully."
  }
}
```

**cURL Example:**
```bash
curl -X DELETE http://localhost:3000/logout \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9..."
```

---

### Budget Endpoints

#### 1. List All Budgets

**Endpoint:** `GET /api/v1/budgets`  
**Authentication:** Required

**Request Headers:**
```
Authorization: Bearer <your-jwt-token>
```

**Success Response (200 OK):**
```json
{
  "data": [
    {
      "id": "1",
      "type": "budget",
      "attributes": {
        "name": "Monthly Groceries",
        "financial_goal": 500.00,
        "total_spent": 325.50,
        "remaining": 174.50,
        "user_id": 1,
        "created_at": "2024-08-20T10:30:00.000Z",
        "updated_at": "2024-08-20T10:30:00.000Z"
      }
    },
    {
      "id": "2",
      "type": "budget",
      "attributes": {
        "name": "Entertainment",
        "financial_goal": 200.00,
        "total_spent": 150.00,
        "remaining": 50.00,
        "user_id": 1,
        "created_at": "2024-08-20T11:00:00.000Z",
        "updated_at": "2024-08-20T11:00:00.000Z"
      }
    }
  ]
}
```

**cURL Example:**
```bash
curl -X GET http://localhost:3000/api/v1/budgets \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9..."
```

---

#### 2. Create Budget

**Endpoint:** `POST /api/v1/budgets`  
**Authentication:** Required

**Request Headers:**
```
Authorization: Bearer <your-jwt-token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "budget": {
    "name": "Vacation Fund",
    "financial_goal": 2000.00
  }
}
```

**Success Response (201 Created):**
```json
{
  "data": {
    "id": "3",
    "type": "budget",
    "attributes": {
      "name": "Vacation Fund",
      "financial_goal": 2000.00,
      "total_spent": 0.00,
      "remaining": 2000.00,
      "user_id": 1,
      "created_at": "2024-08-21T09:15:00.000Z",
      "updated_at": "2024-08-21T09:15:00.000Z"
    }
  }
}
```

**Error Response (422 Unprocessable Entity):**
```json
{
  "errors": {
    "name": ["can't be blank"],
    "financial_goal": ["must be greater than 0"]
  }
}
```

**cURL Example:**
```bash
curl -X POST http://localhost:3000/api/v1/budgets \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9..." \
  -H "Content-Type: application/json" \
  -d '{
    "budget": {
      "name": "Vacation Fund",
      "financial_goal": 2000.00
    }
  }'
```

---

#### 3. Get Specific Budget

**Endpoint:** `GET /api/v1/budgets/:id`  
**Authentication:** Required

**URL Parameters:**
- `id` (integer) - Budget ID

**Success Response (200 OK):**
```json
{
  "data": {
    "id": "1",
    "type": "budget",
    "attributes": {
      "name": "Monthly Groceries",
      "financial_goal": 500.00,
      "total_spent": 325.50,
      "remaining": 174.50,
      "user_id": 1,
      "created_at": "2024-08-20T10:30:00.000Z",
      "updated_at": "2024-08-20T10:30:00.000Z"
    }
  }
}
```

**Error Response (404 Not Found):**
```json
{
  "error": "Budget not found"
}
```

**cURL Example:**
```bash
curl -X GET http://localhost:3000/api/v1/budgets/1 \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9..."
```

---

#### 4. Update Budget

**Endpoint:** `PATCH /api/v1/budgets/:id`  
**Authentication:** Required

**URL Parameters:**
- `id` (integer) - Budget ID

**Request Body:**
```json
{
  "budget": {
    "name": "Monthly Groceries - Updated",
    "financial_goal": 600.00
  }
}
```

**Success Response (200 OK):**
```json
{
  "data": {
    "id": "1",
    "type": "budget",
    "attributes": {
      "name": "Monthly Groceries - Updated",
      "financial_goal": 600.00,
      "total_spent": 325.50,
      "remaining": 274.50,
      "user_id": 1,
      "created_at": "2024-08-20T10:30:00.000Z",
      "updated_at": "2024-08-21T14:20:00.000Z"
    }
  }
}
```

**cURL Example:**
```bash
curl -X PATCH http://localhost:3000/api/v1/budgets/1 \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9..." \
  -H "Content-Type: application/json" \
  -d '{
    "budget": {
      "financial_goal": 600.00
    }
  }'
```

---

#### 5. Delete Budget

**Endpoint:** `DELETE /api/v1/budgets/:id`  
**Authentication:** Required

**URL Parameters:**
- `id` (integer) - Budget ID

**Success Response (204 No Content):**
```
No response body
```

**Error Response (404 Not Found):**
```json
{
  "error": "Budget not found"
}
```

**cURL Example:**
```bash
curl -X DELETE http://localhost:3000/api/v1/budgets/1 \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9..."
```

---

### Transaction Endpoints

#### 1. List Budget Transactions

**Endpoint:** `GET /api/v1/budgets/:budget_id/transactions`  
**Authentication:** Required

**URL Parameters:**
- `budget_id` (integer) - Budget ID

**Success Response (200 OK):**
```json
{
  "data": [
    {
      "id": "1",
      "type": "transaction",
      "attributes": {
        "amount": 45.50,
        "description": "Whole Foods grocery shopping",
        "date": "2024-08-20",
        "budget_id": 1,
        "category_id": 2,
        "category_name": "Groceries",
        "created_at": "2024-08-20T15:30:00.000Z",
        "updated_at": "2024-08-20T15:30:00.000Z"
      }
    }
  ]
}
```

**cURL Example:**
```bash
curl -X GET http://localhost:3000/api/v1/budgets/1/transactions \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9..."
```

---

#### 2. Create Transaction

**Endpoint:** `POST /api/v1/budgets/:budget_id/transactions`  
**Authentication:** Required

**URL Parameters:**
- `budget_id` (integer) - Budget ID

**Request Body:**
```json
{
  "transaction": {
    "amount": 75.00,
    "description": "Weekly groceries at Trader Joe's",
    "date": "2024-08-21",
    "category_id": 2
  }
}
```

**Success Response (201 Created):**
```json
{
  "data": {
    "id": "2",
    "type": "transaction",
    "attributes": {
      "amount": 75.00,
      "description": "Weekly groceries at Trader Joe's",
      "date": "2024-08-21",
      "budget_id": 1,
      "category_id": 2,
      "category_name": "Groceries",
      "created_at": "2024-08-21T10:00:00.000Z",
      "updated_at": "2024-08-21T10:00:00.000Z"
    }
  }
}
```

**Error Response (422 Unprocessable Entity):**
```json
{
  "errors": {
    "amount": ["must be greater than 0"],
    "date": ["can't be blank"],
    "category_id": ["must exist"]
  }
}
```

**cURL Example:**
```bash
curl -X POST http://localhost:3000/api/v1/budgets/1/transactions \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9..." \
  -H "Content-Type: application/json" \
  -d '{
    "transaction": {
      "amount": 75.00,
      "description": "Weekly groceries",
      "date": "2024-08-21",
      "category_id": 2
    }
  }'
```

---

#### 3. Get Specific Transaction

**Endpoint:** `GET /api/v1/budgets/:budget_id/transactions/:id`  
**Authentication:** Required

**URL Parameters:**
- `budget_id` (integer) - Budget ID
- `id` (integer) - Transaction ID

**Success Response (200 OK):**
```json
{
  "data": {
    "id": "1",
    "type": "transaction",
    "attributes": {
      "amount": 45.50,
      "description": "Whole Foods grocery shopping",
      "date": "2024-08-20",
      "budget_id": 1,
      "category_id": 2,
      "category_name": "Groceries",
      "created_at": "2024-08-20T15:30:00.000Z",
      "updated_at": "2024-08-20T15:30:00.000Z"
    }
  }
}
```

**cURL Example:**
```bash
curl -X GET http://localhost:3000/api/v1/budgets/1/transactions/1 \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9..."
```

---

#### 4. Update Transaction

**Endpoint:** `PATCH /api/v1/budgets/:budget_id/transactions/:id`  
**Authentication:** Required

**Request Body:**
```json
{
  "transaction": {
    "amount": 50.00,
    "description": "Updated description"
  }
}
```

**Success Response (200 OK):**
```json
{
  "data": {
    "id": "1",
    "type": "transaction",
    "attributes": {
      "amount": 50.00,
      "description": "Updated description",
      "date": "2024-08-20",
      "budget_id": 1,
      "category_id": 2,
      "category_name": "Groceries",
      "created_at": "2024-08-20T15:30:00.000Z",
      "updated_at": "2024-08-21T11:00:00.000Z"
    }
  }
}
```

**cURL Example:**
```bash
curl -X PATCH http://localhost:3000/api/v1/budgets/1/transactions/1 \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9..." \
  -H "Content-Type: application/json" \
  -d '{
    "transaction": {
      "amount": 50.00
    }
  }'
```

---

#### 5. Delete Transaction

**Endpoint:** `DELETE /api/v1/budgets/:budget_id/transactions/:id`  
**Authentication:** Required

**Success Response (204 No Content):**
```
No response body
```

**cURL Example:**
```bash
curl -X DELETE http://localhost:3000/api/v1/budgets/1/transactions/1 \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9..."
```

---

### Category Endpoints

#### 1. List All Categories

**Endpoint:** `GET /api/v1/categories`  
**Authentication:** Required

**Success Response (200 OK):**
```json
{
  "data": [
    {
      "id": "1",
      "type": "category",
      "attributes": {
        "name": "Food & Dining",
        "created_at": "2024-08-20T10:00:00.000Z",
        "updated_at": "2024-08-20T10:00:00.000Z"
      }
    },
    {
      "id": "2",
      "type": "category",
      "attributes": {
        "name": "Transportation",
        "created_at": "2024-08-20T10:00:00.000Z",
        "updated_at": "2024-08-20T10:00:00.000Z"
      }
    }
  ]
}
```

**cURL Example:**
```bash
curl -X GET http://localhost:3000/api/v1/categories \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9..."
```

---

#### 2. Create Category

**Endpoint:** `POST /api/v1/categories`  
**Authentication:** Required

**Request Body:**
```json
{
  "category": {
    "name": "Healthcare"
  }
}
```

**Success Response (201 Created):**
```json
{
  "data": {
    "id": "3",
    "type": "category",
    "attributes": {
      "name": "Healthcare",
      "created_at": "2024-08-21T12:00:00.000Z",
      "updated_at": "2024-08-21T12:00:00.000Z"
    }
  }
}
```

**Error Response (422 Unprocessable Entity):**
```json
{
  "errors": {
    "name": ["has already been taken"]
  }
}
```

**cURL Example:**
```bash
curl -X POST http://localhost:3000/api/v1/categories \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9..." \
  -H "Content-Type: application/json" \
  -d '{
    "category": {
      "name": "Healthcare"
    }
  }'
```

---

#### 3. Get Specific Category

**Endpoint:** `GET /api/v1/categories/:id`  
**Authentication:** Required

**Success Response (200 OK):**
```json
{
  "data": {
    "id": "1",
    "type": "category",
    "attributes": {
      "name": "Food & Dining",
      "created_at": "2024-08-20T10:00:00.000Z",
      "updated_at": "2024-08-20T10:00:00.000Z"
    }
  }
}
```

**cURL Example:**
```bash
curl -X GET http://localhost:3000/api/v1/categories/1 \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9..."
```

---

#### 4. Update Category

**Endpoint:** `PATCH /api/v1/categories/:id`  
**Authentication:** Required

**Request Body:**
```json
{
  "category": {
    "name": "Food & Restaurants"
  }
}
```

**Success Response (200 OK):**
```json
{
  "data": {
    "id": "1",
    "type": "category",
    "attributes": {
      "name": "Food & Restaurants",
      "created_at": "2024-08-20T10:00:00.000Z",
      "updated_at": "2024-08-21T13:30:00.000Z"
    }
  }
}
```

**cURL Example:**
```bash
curl -X PATCH http://localhost:3000/api/v1/categories/1 \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9..." \
  -H "Content-Type: application/json" \
  -d '{
    "category": {
      "name": "Food & Restaurants"
    }
  }'
```

---

#### 5. Delete Category

**Endpoint:** `DELETE /api/v1/categories/:id`  
**Authentication:** Required

**Success Response (204 No Content):**
```
No response body
```

**Error Response (422 Unprocessable Entity):**
```json
{
  "error": "Cannot delete category with associated transactions"
}
```

**cURL Example:**
```bash
curl -X DELETE http://localhost:3000/api/v1/categories/1 \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9..."
```

---

## Error Handling

### Standard Error Response Format

All error responses follow a consistent format:

```json
{
  "error": "Error message description",
  "status": 400
}
```

Or for validation errors:

```json
{
  "errors": {
    "field_name": ["error message 1", "error message 2"]
  }
}
```

### HTTP Status Codes

| Status Code | Description |
|-------------|-------------|
| 200 | OK - Request succeeded |
| 201 | Created - Resource successfully created |
| 204 | No Content - Request succeeded, no content to return |
| 400 | Bad Request - Invalid request format |
| 401 | Unauthorized - Missing or invalid authentication token |
| 403 | Forbidden - Authenticated but not authorized for this resource |
| 404 | Not Found - Resource doesn't exist |
| 422 | Unprocessable Entity - Validation errors |
| 500 | Internal Server Error - Server error occurred |

### Common Error Scenarios

#### 1. Missing Authentication Token
```json
{
  "error": "You need to sign in or sign up before continuing.",
  "status": 401
}
```

#### 2. Invalid/Expired Token
```json
{
  "error": "Signature has expired",
  "status": 401
}
```

#### 3. Resource Not Found
```json
{
  "error": "Budget not found",
  "status": 404
}
```

#### 4. Validation Errors
```json
{
  "errors": {
    "email": ["can't be blank", "is invalid"],
    "password": ["is too short (minimum is 6 characters)"]
  },
  "status": 422
}
```

#### 5. Unauthorized Access
```json
{
  "error": "You are not authorized to access this resource",
  "status": 403
}
```

---

## Rate Limiting

Currently, the API does not implement rate limiting. This may be added in future versions.

---

## Additional Notes

### Data Formats

- **Dates:** ISO 8601 format (YYYY-MM-DD)
- **Timestamps:** ISO 8601 format with timezone (YYYY-MM-DDTHH:MM:SS.sssZ)
- **Currency:** Decimal format with 2 decimal places (e.g., 123.45)

### CORS

The API supports Cross-Origin Resource Sharing (CORS) for the frontend application. Allowed origins are configured in `config/initializers/cors.rb`.

### Pagination

Currently, the API returns all results without pagination. Pagination support may be added in future versions.

---

## Testing the API

### Using cURL

All examples in this documentation use cURL. Make sure to:
1. Replace `<your-jwt-token>` with your actual JWT token
2. Update the base URL if not using localhost
3. Adjust IDs and data as needed

### Using Postman

A Postman collection is available in the `docs/api/` directory:
- Import `SpendWise_API_Collection.json` into Postman
- Set up an environment variable for your JWT token
- Start testing!

### Using HTTPie

HTTPie is a user-friendly command-line HTTP client:

```bash
# Login example
http POST localhost:3000/login user:='{"email":"user@example.com","password":"password123"}'

# Authenticated request
http GET localhost:3000/api/v1/budgets "Authorization: Bearer YOUR_TOKEN"
```

---

## Support

For issues, questions, or contributions, please visit the [GitHub repository](https://github.com/Limeload/SpendWise) or open an issue.

## Version History

- **v1.0** (Current) - Initial API release with core functionality