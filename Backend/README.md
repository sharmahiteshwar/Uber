# User Authentication API Documentation

## Overview
This API provides endpoints for user registration and authentication using JWT tokens.

---

## Endpoint: POST /users/register

### Description
This endpoint allows new users to register an account in the system. It validates the provided user data, creates a new user record with hashed password, and returns an authentication token along with the created user details.

### URL
```
POST /users/register
```

### Request Headers
```
Content-Type: application/json
```

### Request Body
The request body must be a JSON object with the following structure:

```json
{
  "fullname": {
    "firstname": "string",  // Required: minimum 3 characters
    "lastname": "string"    // Optional: minimum 3 characters if provided
  },
  "email": "string",        // Required: valid email format
  "password": "string"      // Required: minimum 6 characters
}
```

### Required Data Fields
| Field | Type | Required | Description | Validation Rules |
|-------|------|----------|-------------|------------------|
| fullname.firstname | string | Yes | User's first name | Minimum 3 characters |
| fullname.lastname | string | No | User's last name | Minimum 3 characters if provided |
| email | string | Yes | User's email address | Must be valid email format |
| password | string | Yes | User's password | Minimum 6 characters |

### Example Request
```bash
curl -X POST http://localhost:3000/users/register \
  -H "Content-Type: application/json" \
  -d '{
    "fullname": {
      "firstname": "John",
      "lastname": "Doe"
    },
    "email": "john.doe@example.com",
    "password": "securepassword123"
  }'
```

### Success Response
**Status Code:** `201 Created`

**Response Body:**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "_id": "507f1f77bcf86cd799439011",
    "fullname": {
      "firstname": "John",
      "lastname": "Doe"
    },
    "email": "john.doe@example.com",
    "__v": 0
  }
}
```

### Error Responses

#### 400 Bad Request
**Status Code:** `400 Bad Request`

**When:** Validation errors occur due to invalid input data.

**Response Body:**
```json
{
  "errors": [
    {
      "type": "field",
      "value": "invalid-email",
      "msg": "Please enter a valid email",
      "path": "email",
      "location": "body"
    }
  ]
}
```

---

## Endpoint: POST /users/login

### Description
This endpoint allows existing users to authenticate by providing their email and password. It validates the credentials, checks the password against the stored hash, and returns an authentication token along with the user details upon successful login.

### URL
```
POST /users/login
```

### Request Headers
```
Content-Type: application/json
```

### Request Body
The request body must be a JSON object with the following structure:

```json
{
  "email": "string",        // Required: valid email format
  "password": "string"      // Required: minimum 6 characters
}
```

### Required Data Fields
| Field | Type | Required | Description | Validation Rules |
|-------|------|----------|-------------|------------------|
| email | string | Yes | User's email address | Must be valid email format |
| password | string | Yes | User's password | Minimum 6 characters |

### Example Request
```bash
curl -X POST http://localhost:3000/users/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "john.doe@example.com",
    "password": "securepassword123"
  }'
```

### Success Response
**Status Code:** `200 OK`

**Response Body:**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "_id": "507f1f77bcf86cd799439011",
    "fullname": {
      "firstname": "John",
      "lastname": "Doe"
    },
    "email": "john.doe@example.com",
    "__v": 0
  }
}
```

### Error Responses

#### 400 Bad Request
**Status Code:** `400 Bad Request`

**When:** Validation errors occur due to invalid input data.

**Response Body:**
```json
{
  "errors": [
    {
      "type": "field",
      "value": "invalid-email",
      "msg": "Please enter a valid email",
      "path": "email",
      "location": "body"
    }
  ]
}
```

#### 401 Unauthorized
**Status Code:** `401 Unauthorized`

**When:** Invalid email or password is provided.

**Response Body:**
```json
{
  "message": "Invalid email or password"
}
```

### Authentication Process
1. **Validation**: The endpoint validates all input fields using express-validator middleware
2. **User Lookup**: The system searches for a user with the provided email address
3. **Password Verification**: The provided password is compared against the stored hashed password using bcrypt
4. **Token Generation**: A JWT authentication token is generated for the authenticated user
5. **Response**: The token and user details (excluding password) are returned to the client

### Security Features
- Passwords are never stored in plain text - always hashed using bcrypt
- JWT tokens are used for authentication
- Generic error messages prevent user enumeration attacks
- Password field is excluded from database queries by default

### Notes
- The JWT token should be included in the Authorization header for subsequent authenticated requests
- Tokens typically expire after a certain period and need to be refreshed

---

## Endpoint: GET /users/profile

### Description
This endpoint retrieves the authenticated user's profile information. It requires a valid JWT token to access the user's details.

### URL
```
GET /users/profile
```

### Request Headers
```
Authorization: Bearer <token>
```

### Request Body
No request body is required for this endpoint.

### Example Request
```bash
curl -X GET http://localhost:3000/users/profile \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

### Success Response
**Status Code:** `200 OK`

**Response Body:**
```json
{
  "_id": "507f1f77bcf86cd799439011",
  "fullname": {
    "firstname": "John",
    "lastname": "Doe"
  },
  "email": "john.doe@example.com",
  "__v": 0
}
```

### Error Responses

#### 401 Unauthorized
**Status Code:** `401 Unauthorized`

**When:** No valid token is provided or the token has expired.

**Response Body:**
```json
{
  "message": "Unauthorized"
}
```

### Security Features
- Requires valid JWT token for authentication
- Returns only the authenticated user's information
- No sensitive data like passwords are included in the response

---

## Endpoint: GET /users/logout

### Description
This endpoint logs out the authenticated user by invalidating their current JWT token. The token is added to a blacklist to prevent future use.

### URL
```
GET /users/logout
```

### Request Headers
```
Authorization: Bearer <token>
```

### Request Body
No request body is required for this endpoint.

### Example Request
```bash
curl -X GET http://localhost:3000/users/logout \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

### Success Response
**Status Code:** `200 OK`

**Response Body:**
```json
{
  "message": "Logged out successfully"
}
```

### Error Responses

#### 401 Unauthorized
**Status Code:** `401 Unauthorized`

**When:** No valid token is provided or the token has expired.

**Response Body:**
```json
{
  "message": "Unauthorized"
}
```

### Security Features
- Requires valid JWT token for authentication
- Invalidates the token by adding it to a blacklist
- Clears the authentication cookie
- Prevents token reuse after logout

### Notes
- After successful logout, the token cannot be used for subsequent requests
- Users need to log in again to obtain a new valid token
