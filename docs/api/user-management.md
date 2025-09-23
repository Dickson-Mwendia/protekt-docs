---
title: "User Management API Reference"
description: "Essential API reference for Protekt's user management endpoints - creating users, managing roles, and basic user operations."
author: "Protekt API Documentation Team"
reviewer: "Sarah Chen, Senior Product Manager"
date: "2025-09-23"
keywords: 
  - user management API
  - user creation
  - role assignment
  - user profiles
tags:
  - api-reference
  - user-management
  - rbac
customer_intent: "As a developer, I want to integrate Protekt's essential user management capabilities to programmatically manage users and roles using Protekt."
---

# User Management API Reference

The Protekt User Management API allows you to programmatically create users, manage roles, and perform basic user operations. This guide covers the core endpoints you'll need for most applications.

## Authentication

All User Management API calls require a Management API access token.

### Getting a Management API Token

```bash
curl -X POST https://your-domain.protekt.dev/oauth/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=client_credentials" \
  -d "client_id=your_management_client_id" \
  -d "client_secret=your_management_client_secret" \
  -d "audience=https://your-domain.protekt.dev/api/v2/"
```

**Response:**
```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

## Base URL

```
https://your-domain.protekt.dev/api/v2
```

## Core User Operations

### 1. Get Users

Retrieves a list of users with basic filtering.

#### `GET /users`

**Headers:**
```
Authorization: Bearer MANAGEMENT_API_TOKEN
```

**Common Query Parameters:**

| Parameter | Description | Example |
|-----------|-------------|---------|
| `page` | Page number (0-based) | `0` |
| `per_page` | Results per page (1-50) | `25` |
| `q` | Search query | `email:"john@example.com"` |

**Example Request:**

```bash
curl -X GET "https://your-domain.protekt.dev/api/v2/users?per_page=10" \
  -H "Authorization: Bearer MANAGEMENT_API_TOKEN"
```

**Example Response:**

```json
{
  "users": [
    {
      "user_id": "protekt|507f1f77bcf86cd799439011",
      "email": "john.doe@example.com",
      "name": "John Doe",
      "created_at": "2025-09-23T10:30:00.000Z",
      "last_login": "2025-09-23T14:15:00.000Z",
      "app_metadata": {
        "roles": ["editor", "user"]
      }
    }
  ]
}
```

### 2. Get User by ID

Retrieves a specific user's information.

#### `GET /users/{id}`

**Example Request:**

```bash
curl -X GET "https://your-domain.protekt.dev/api/v2/users/protekt%7C507f1f77bcf86cd799439011" \
  -H "Authorization: Bearer MANAGEMENT_API_TOKEN"
```

**Example Response:**

```json
{
  "user_id": "protekt|507f1f77bcf86cd799439011",
  "email": "john.doe@example.com",
  "name": "John Doe",
  "given_name": "John",
  "family_name": "Doe",
  "picture": "https://example.com/avatar.jpg",
  "created_at": "2025-09-23T10:30:00.000Z",
  "last_login": "2025-09-23T14:15:00.000Z",
  "app_metadata": {
    "roles": ["editor", "user"]
  }
}
```

### 3. Create User

Creates a new user in your application.

#### `POST /users`

**Headers:**
```
Authorization: Bearer MANAGEMENT_API_TOKEN
Content-Type: application/json
```

**Required Fields:**

| Field | Description |
|-------|-------------|
| `connection` | Connection name (usually "Username-Password-Authentication") |
| `email` | User's email address |
| `password` | User's password |

**Example Request:**

```bash
curl -X POST https://your-domain.protekt.dev/api/v2/users \
  -H "Authorization: Bearer MANAGEMENT_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "connection": "Username-Password-Authentication",
    "email": "jane.doe@example.com",
    "password": "SecurePassword123!",
    "name": "Jane Doe",
    "verify_email": true
  }'
```

**Example Response:**

```json
{
  "user_id": "protekt|507f1f77bcf86cd799439012",
  "email": "jane.doe@example.com",
  "name": "Jane Doe",
  "created_at": "2025-09-23T15:00:00.000Z"
}
```

### 4. Update User

Updates a user's basic information.

#### `PATCH /users/{id}`

**Example Request:**

```bash
curl -X PATCH "https://your-domain.protekt.dev/api/v2/users/protekt%7C507f1f77bcf86cd799439011" \
  -H "Authorization: Bearer MANAGEMENT_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "John Smith",
    "user_metadata": {
      "department": "Engineering"
    }
  }'
```

## Role Management

### 5. Get User Roles

Retrieves roles assigned to a specific user.

#### `GET /users/{id}/roles`

**Example Request:**

```bash
curl -X GET "https://your-domain.protekt.dev/api/v2/users/protekt%7C507f1f77bcf86cd799439011/roles" \
  -H "Authorization: Bearer MANAGEMENT_API_TOKEN"
```

**Example Response:**

```json
[
  {
    "id": "rol_12345",
    "name": "editor",
    "description": "Content Editor Role"
  },
  {
    "id": "rol_12346", 
    "name": "user",
    "description": "Standard User Role"
  }
]
```

### 6. Assign Roles to User

Assigns one or more roles to a user.

#### `POST /users/{id}/roles`

**Request Body:**

```json
{
  "roles": ["rol_12345", "rol_12347"]
}
```

**Example Request:**

```bash
curl -X POST "https://your-domain.protekt.dev/api/v2/users/protekt%7C507f1f77bcf86cd799439011/roles" \
  -H "Authorization: Bearer MANAGEMENT_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "roles": ["rol_12345"]
  }'
```

### 7. Remove Roles from User

Removes one or more roles from a user.

#### `DELETE /users/{id}/roles`

**Request Body:**

```json
{
  "roles": ["rol_12346"]
}
```

**Example Request:**

```bash
curl -X DELETE "https://your-domain.protekt.dev/api/v2/users/protekt%7C507f1f77bcf86cd799439011/roles" \
  -H "Authorization: Bearer MANAGEMENT_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "roles": ["rol_12346"]
  }'
```

## SDK Integration Examples

### Node.js

```javascript
const { ManagementApiClient } = require('@protekt/management');

const management = new ManagementApiClient({
  domain: 'your-domain.protekt.dev',
  clientId: 'your_management_client_id',
  clientSecret: 'your_management_client_secret'
});

// Create a new user
async function createUser(email, password, name) {
  try {
    const user = await management.users.create({
      connection: 'Username-Password-Authentication',
      email,
      password,
      name,
      verify_email: true
    });
    
    console.log('User created:', user.user_id);
    return user;
  } catch (error) {
    console.error('Error creating user:', error);
  }
}

// Assign role to user
async function assignRole(userId, roleId) {
  try {
    await management.users.assignRoles(userId, {
      roles: [roleId]
    });
    
    console.log(`Role ${roleId} assigned to ${userId}`);
  } catch (error) {
    console.error('Error assigning role:', error);
  }
}

// Get user by email
async function findUserByEmail(email) {
  try {
    const users = await management.users.getAll({
      q: `email:"${email}"`,
      per_page: 1
    });
    
    return users.length > 0 ? users[0] : null;
  } catch (error) {
    console.error('Error finding user:', error);
    return null;
  }
}

// Get user roles
async function getUserRoles(userId) {
  try {
    const roles = await management.users.getRoles(userId);
    return roles;
  } catch (error) {
    console.error('Error getting user roles:', error);
    return [];
  }
}
```

### Python

```python
from protekt.management import ManagementApiClient

management = ManagementApiClient(
    domain='your-domain.protekt.dev',
    client_id='your_management_client_id',
    client_secret='your_management_client_secret'
)

# Create a new user
def create_user(email, password, name):
    try:
        user = management.users.create({
            'connection': 'Username-Password-Authentication',
            'email': email,
            'password': password,
            'name': name,
            'verify_email': True
        })
        
        print(f'User created: {user["user_id"]}')
        return user
    except Exception as e:
        print(f'Error creating user: {e}')

# Assign role to user
def assign_role(user_id, role_id):
    try:
        management.users.assign_roles(user_id, {'roles': [role_id]})
        print(f'Role {role_id} assigned to {user_id}')
    except Exception as e:
        print(f'Error assigning role: {e}')

# Find user by email
def find_user_by_email(email):
    try:
        users = management.users.list(q=f'email:"{email}"', per_page=1)
        return users[0] if users else None
    except Exception as e:
        print(f'Error finding user: {e}')
        return None
```

## Common Use Cases

### User Registration Flow

```javascript
// Complete user registration with role assignment
async function registerUser(userData) {
  try {
    // 1. Create user
    const user = await management.users.create({
      connection: 'Username-Password-Authentication',
      email: userData.email,
      password: userData.password,
      name: userData.name,
      verify_email: true
    });
    
    // 2. Assign default role
    await management.users.assignRoles(user.user_id, {
      roles: ['rol_default_user']
    });
    
    // 3. Update user metadata
    await management.users.update(user.user_id, {
      user_metadata: {
        registration_source: 'web',
        onboarding_completed: false
      }
    });
    
    return user;
  } catch (error) {
    console.error('Registration failed:', error);
    throw error;
  }
}
```

### Role-Based Access Control

```javascript
// Check if user has specific role
async function userHasRole(userId, roleName) {
  try {
    const roles = await management.users.getRoles(userId);
    return roles.some(role => role.name === roleName);
  } catch (error) {
    console.error('Error checking user role:', error);
    return false;
  }
}

// Promote user to admin
async function promoteToAdmin(userId) {
  try {
    await management.users.assignRoles(userId, {
      roles: ['rol_admin']
    });
    
    console.log(`User ${userId} promoted to admin`);
  } catch (error) {
    console.error('Error promoting user:', error);
  }
}
```

## Error Handling

### Common Error Responses

| Status Code | Description | Solution |
|-------------|-------------|----------|
| 400 | Invalid request data | Check request format |
| 401 | Invalid access token | Get new management token |
| 403 | Insufficient permissions | Check token scopes |
| 404 | User not found | Verify user ID |
| 409 | User already exists | Use different email |

### Example Error Response

```json
{
  "statusCode": 409,
  "error": "Conflict",
  "message": "The user already exists",
  "errorCode": "user_exists"
}
```

### Error Handling Best Practices

```javascript
async function createUserSafely(userData) {
  try {
    return await management.users.create(userData);
  } catch (error) {
    switch (error.statusCode) {
      case 409:
        // User already exists
        console.log('User already exists, fetching existing user');
        return await findUserByEmail(userData.email);
      case 401:
        // Token expired
        console.log('Management token expired, refreshing');
        await refreshManagementToken();
        return await management.users.create(userData);
      default:
        console.error('Unexpected error:', error);
        throw error;
    }
  }
}
```

## Rate Limits

| Endpoint | Limit | Window |
|----------|-------|--------|
| `GET /users` | 500 requests | 1 minute |
| `POST /users` | 50 requests | 1 minute |
| `PATCH /users/{id}` | 200 requests | 1 minute |
| Role operations | 100 requests | 1 minute |

## Security Best Practices

1. **Use least privilege** - Grant minimal required scopes to management tokens
2. **Validate input** - Always validate user data before API calls
3. **Handle errors gracefully** - Don't expose sensitive error details to end users
4. **Log operations** - Keep audit logs of user management operations
5. **Rotate tokens** - Regularly refresh management API tokens

## Quick Setup Checklist

1. ✅ Get Management API credentials from Protekt dashboard
2. ✅ Generate management access token with required scopes
3. ✅ Test user creation with basic fields
4. ✅ Set up role assignment for new users
5. ✅ Implement error handling for common scenarios

## Related Resources

- [Authentication API](./authentication) - Core authentication endpoints
- [RBAC Implementation Guide](../guides/role-based-access-control) - Complete RBAC setup
- [Quickstart Guide](../getting-started/quickstart) - Basic integration example

---

For questions about the User Management API, contact [developers@protekt.dev](mailto:developers@protekt.dev) or visit our [community forum](https://community.protekt.dev).
