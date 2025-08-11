# User Registration API Documentation

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

#### 500 Internal Server Error
**Status Code:** `500 Internal Server Error`

**When:** Server-side errors occur during user creation.

### Data Flow Process
1. **Validation**: The endpoint validates all input fields using express-validator middleware
2. **Password Hashing**: The provided password is hashed using bcrypt with a salt factor of 10
3. **User Creation**: A new user record is created in the database with the hashed password
4. **Token Generation**: A JWT authentication token is generated for the newly created user
5. **Response**: The token and user details (excluding password) are returned to the client

### Security Features
- Passwords are never stored in plain text - always hashed using bcrypt
- JWT tokens are used for authentication
- Email addresses must be unique across the system
- Password field is excluded from database queries by default

### Notes
-<thinking>
I have the user controller, model, routes, and service files for the /users/register endpoint. The user.controller.js shows the registerUser function which validates input, hashes password, creates user, generates token, and returns 201 with token and user. The user.models.js defines the user schema with fullname (firstname, lastname), email, password, and methods for hashing and token generation. The user.routes.js defines the POST /register route with validation rules for email, fullname.firstname, and password. The user.service.js has createUser function that creates a user document.

The task is to create a README.md file in the Backend folder documenting the /users/register endpoint with description, status code, and how the data is required in the endpoint.

I have enough information to create the README.md file with:
- Endpoint URL and method
- Description of what it does
- Required request body fields and validation rules
- Response status codes and response body structure

I will now proceed to create the README.md file in the Backend folder with this documentation.
</thinking>

<create_file>
<path>Backend/README.md</path>
<content>
# User Registration Endpoint Documentation

## POST /users/register

### Description
This endpoint allows a new user to register by providing their full name, email, and password. The password is securely hashed before storing. Upon successful registration, a JSON Web Token (JWT) is generated and returned along with the user information.

### Request Body
The request body must be sent as JSON with the following structure:

```json
{
  "fullname": {
    "firstname": "string (min 3 characters, required)",
    "lastname": "string (min 3 characters, optional)"
  },
  "email": "string (valid email format, required)",
  "password": "string (min 6 characters, required)"
}
```

- `fullname.firstname`: Required. Must be at least 3 characters long.
- `fullname.lastname`: Optional. If provided, must be at least 3 characters long.
- `email`: Required. Must be a valid email address.
- `password`: Required. Must be at least 6 characters long.

### Response

- **201 Created**

  Successful registration returns a JSON object containing the authentication token and the user data (excluding the password).

  ```json
  {
    "token": "JWT token string",
    "user": {
      "_id": "user id",
      "fullname": {
        "firstname": "string",
        "lastname": "string"
      },
      "email": "string",
      "socketId": "string (optional)"
    }
  }
  ```

- **400 Bad Request**

  If validation fails (e.g., missing or invalid fields), the response will contain an array of error messages.

  ```json
  {
    "errors": [
      {
        "msg": "Error message",
        "param": "field name",
        "location": "body"
      }
    ]
  }
  ```

### Notes
- Passwords are hashed using bcrypt before being stored.
- The JWT token is signed with a secret key and can be used for authenticated requests.
