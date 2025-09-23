---
title: "Roles & Permissions API Reference"
description: "Essential API reference for managing roles and permissions in Protekt to implement RBAC."
customer_intent: "Programmatically manage roles and permissions to implement role-based access control."
author: "Protekt API Documentation Team"
reviewer: "Sarah Chen, Senior Product Manager"
date: "2025-09-23"
keywords: 
  - roles API
  - permissions API
  - RBAC
  - access control
  - authorization
tags:
  - api-reference
  - roles
  - permissions
  - rbac
  - authorization
---

# Roles & Permissions API Reference

The Protekt Roles & Permissions API enables you to programmatically manage roles and permissions for implementing RBAC in your applications.

## Authentication

All API calls require a Management API access token with appropriate scopes.

### Required Scopes

| Scope | Description |
|-------|-------------|
| `read:roles` | Read role information |
| `create:roles` | Create new roles |
| `update:roles` | Update existing roles |
| `delete:roles` | Delete roles |

## Base URL

```text
https://your-domain.protekt.dev/api/v2
```

## Core Endpoints

## Core Endpoints

### 1. Get Roles

Retrieves a list of roles in your application.

#### `GET /roles`

**Headers:**

```text
Authorization: Bearer MANAGEMENT_API_TOKEN
```

**Query Parameters:**

| Parameter | Type | Description | Default |
|-----------|------|-------------|---------|
| `per_page` | integer | Results per page (1-100) | 25 |
| `page` | integer | Page number (0-based) | 0 |

**Example Request:**

```bash
curl -X GET "https://your-domain.protekt.dev/api/v2/roles" \
  -H "Authorization: Bearer MANAGEMENT_API_TOKEN"
```

**Example Response:**

```json
{
  "roles": [
    {
      "id": "rol_admin",
      "name": "admin",
      "description": "Administrator role with full access"
    },
    {
      "id": "rol_editor",
      "name": "editor",
      "description": "Content editor role"
    },
    {
      "id": "rol_viewer",
      "name": "viewer",
      "description": "Read-only access role"
    }
  ]
}
```

### 2. Create Role

Creates a new role for your application.

#### `POST /roles`

**Request Body:**

```json
{
  "name": "moderator",
  "description": "Content moderator role"
}
```

**Example Request:**

```bash
curl -X POST https://your-domain.protekt.dev/api/v2/roles \
  -H "Authorization: Bearer MANAGEMENT_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "moderator",
    "description": "Content moderator role"
  }'
```

**Example Response:**

```json
{
  "id": "rol_moderator",
  "name": "moderator",
  "description": "Content moderator role"
}
```

### 3. Get Role Permissions

Retrieves permissions assigned to a specific role.

#### `GET /roles/{id}/permissions`

**Example Request:**

```bash
curl -X GET "https://your-domain.protekt.dev/api/v2/roles/rol_editor/permissions" \
  -H "Authorization: Bearer MANAGEMENT_API_TOKEN"
```

**Example Response:**

```json
[
  {
    "permission_name": "read:posts",
    "description": "Read blog posts"
  },
  {
    "permission_name": "write:posts",
    "description": "Create and edit blog posts"
  }
]
```

### 4. Assign Permissions to Role

Assigns one or more permissions to a role.

#### `POST /roles/{id}/permissions`

**Request Body:**

```json
{
  "permissions": [
    {
      "permission_name": "read:posts",
      "resource_server_identifier": "your-api"
    },
    {
      "permission_name": "write:posts", 
      "resource_server_identifier": "your-api"
    }
  ]
}
```

**Example Request:**

```bash
curl -X POST "https://your-domain.protekt.dev/api/v2/roles/rol_editor/permissions" \
  -H "Authorization: Bearer MANAGEMENT_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "permissions": [
      {
        "permission_name": "write:posts",
        "resource_server_identifier": "your-api"
      }
    ]
  }'
```

### 5. Remove Permissions from Role

Removes permissions from a role.

#### `DELETE /roles/{id}/permissions`

**Request Body:**

```json
{
  "permissions": [
    {
      "permission_name": "delete:posts",
      "resource_server_identifier": "your-api"
    }
  ]
}
```

## SDK Examples

### Node.js

```javascript
const { ManagementApiClient } = require('@protekt/management');

const management = new ManagementApiClient({
  domain: 'your-domain.protekt.dev',
  clientId: 'your_management_client_id',
  clientSecret: 'your_management_client_secret'
});

// Create a new role
async function createRole() {
  const role = await management.roles.create({
    name: 'content-manager',
    description: 'Content management role'
  });
  
  console.log('Role created:', role.id);
  return role;
}

// Assign permissions to role
async function assignPermissions(roleId) {
  await management.roles.addPermissions(roleId, {
    permissions: [
      { permission_name: 'read:posts', resource_server_identifier: 'your-api' },
      { permission_name: 'write:posts', resource_server_identifier: 'your-api' }
    ]
  });
  
  console.log(`Permissions assigned to role ${roleId}`);
}

// Get all roles
async function getRoles() {
  const roles = await management.roles.getAll();
  return roles;
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

# Create role with permissions
def create_role_with_permissions():
    # Create role
    role = management.roles.create({
        'name': 'data-analyst',
        'description': 'Data analysis role'
    })
    
    # Assign permissions
    management.roles.add_permissions(role['id'], {
        'permissions': [
            {
                'permission_name': 'read:analytics',
                'resource_server_identifier': 'your-api'
            }
        ]
    })
    
    return role

# Get role permissions
def get_role_permissions(role_id):
    permissions = management.roles.get_permissions(role_id)
    return permissions
```

## Error Responses

### Common Error Codes

| Status Code | Error Code | Description |
|-------------|------------|-------------|
| 400 | `invalid_body` | Request body is invalid |
| 401 | `unauthorized` | Invalid or missing access token |
| 403 | `insufficient_scope` | Token lacks required scope |
| 404 | `not_found` | Role or permission not found |
| 409 | `conflict` | Role already exists |

### Error Response Format

```json
{
  "statusCode": 400,
  "error": "Bad Request",
  "message": "The specified role does not exist",
  "errorCode": "role_not_found"
}
```

## Best Practices

### Role Design
- Use descriptive names (`content-editor` instead of `role1`)
- Follow principle of least privilege
- Group related permissions together
- Document role purposes clearly

### Permission Management
- Use namespaced permissions (`read:posts`, `write:posts`)
- Be specific with permission names
- Plan permissions for scalability
- Regular audits of role assignments

### Example: Setting Up Basic RBAC

```javascript
// Complete example matching the how-to guide
async function setupBasicRBAC() {
  // 1. Create roles
  const adminRole = await management.roles.create({
    name: 'admin',
    description: 'Full system access'
  });
  
  const editorRole = await management.roles.create({
    name: 'editor',
    description: 'Can create and edit content'
  });
  
  const viewerRole = await management.roles.create({
    name: 'viewer',
    description: 'Read-only access'
  });
  
  // 2. Assign permissions
  await management.roles.addPermissions(adminRole.id, {
    permissions: [
      { permission_name: 'read:all', resource_server_identifier: 'your-api' },
      { permission_name: 'write:all', resource_server_identifier: 'your-api' },
      { permission_name: 'delete:all', resource_server_identifier: 'your-api' }
    ]
  });
  
  await management.roles.addPermissions(editorRole.id, {
    permissions: [
      { permission_name: 'read:posts', resource_server_identifier: 'your-api' },
      { permission_name: 'write:posts', resource_server_identifier: 'your-api' }
    ]
  });
  
  await management.roles.addPermissions(viewerRole.id, {
    permissions: [
      { permission_name: 'read:posts', resource_server_identifier: 'your-api' }
    ]
  });
  
  console.log('Basic RBAC setup complete');
  return { adminRole, editorRole, viewerRole };
}
```

## Related Resources

- [Implement Role-Based Access Control](../guides/role-based-access-control) - Step-by-step implementation guide
- [User Management API](./user-management) - Assign roles to users
- [Authentication API](./authentication) - Include permissions in tokens

---

For additional support with the Roles & Permissions API, please contact our [developer support team](mailto:developers@protekt.dev).
