---
title: "Implement Role-Based Access Control"
description: "Simple guide to implementing role-based access control (RBAC) with Protekt, including user roles, permissions, and authorization patterns."
author: "Dickson Mwendia"
reviewer: "John Doe, Senior Product Manager"
date: "2025-09-23"
keywords: 
  - role-based access control
  - RBAC
  - authorization
  - permissions
  - user roles
tags:
  - guides
  - security
  - authorization
  - rbac
customer_intent: "As a developer, I want to implement secure role-based access control in my React application using Protekt's authorization features."
---

# Implement Role-Based Access Control

Role-Based Access Control (RBAC) restricts access to resources based on user roles. This guide shows you how to implement RBAC using Protekt.

## Prerequisites

- A Protekt application set up. If you haven't already, see our [Quickstart Guide](../getting-started/quickstart)
- Basic authentication working in your app
- An understanding of how roles in Protekt work

## Step 1: Create Roles in Protekt

### 1.1: Set up roles in your dashboard

Follow these steps to create a custom role in the Protekt Dashboard to manage your app and user registrations.

1. Sign in to the **Protekt Dashboard**
1. Browse to **Applications** → **Your App**
1. Navigate to **"Roles & Permissions"**
1. Create these roles:

    - **Admin**: Full system access
    - **Editor**: Can create and edit content  
    - **Viewer**: Read-only access

### 1.2: Define permissions

For each role, assign appropriate permissions as follows:

```json
{
  "admin": ["read:all", "write:all", "delete:all"],
  "editor": ["read:posts", "write:posts"], 
  "viewer": ["read:posts"]
}
```

## Step 2: Assign Roles to Users

The Management API allows you to programmatically assign roles to users. This is useful for automated user onboarding or admin interfaces.

```javascript
const { ManagementApiClient } = require('@protekt/management');

const management = new ManagementApiClient({
  domain: 'your-domain.protekt.dev',
  clientId: 'your-management-client-id',
  clientSecret: 'your-management-client-secret'
});

// Assign role to user
async function assignRole(userId, roleId) {
  await management.users.assignRoles(userId, {
    roles: [roleId]
  });
}

// Usage
await assignRole('user_123', 'rol_editor');
```

**How it works:**

1. **Management Client**: Creates a client authenticated with your management credentials
2. **assignRole Function**: Takes a user ID and role ID, then assigns the role to that user
3. **API Call**: Uses Protekt's Management API to update the user's roles
4. **Result**: The user now has the "editor" role and all its associated permissions

## Step 3: Check Permissions in Your App

### Frontend Permissions

This section shows how to check user permissions in your React components to control what users can see and do.

```javascript
// Get user permissions from token
function getUserPermissions(user) {
  return user['https://your-app.com/permissions'] || [];
}

// Check if user has permission
function hasPermission(user, permission) {
  const permissions = getUserPermissions(user);
  return permissions.includes(permission);
}

// Conditional rendering
function PostEditor({ user, post }) {
  if (!hasPermission(user, 'write:posts')) {
    return <div>You don't have permission to edit posts.</div>;
  }
  
  return <EditPostForm post={post} />;
}
```

**How it works:**

1. **getUserPermissions**: Extracts the permissions array from the user's JWT token (stored in custom claims)
2. **hasPermission**: Checks if a specific permission exists in the user's permission list
3. **PostEditor Component**: Uses conditional rendering to show either an error message or the edit form based on permissions
4. **Result**: Users without "write:posts" permission see a message instead of the edit interface

### Backend API Protection

Protect your API endpoints by verifying user permissions before allowing access to resources.

```javascript
// Middleware to check permissions
function requirePermission(permission) {
  return (req, res, next) => {
    const permissions = req.user['https://your-app.com/permissions'] || [];
    
    if (!permissions.includes(permission)) {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }
    
    next();
  };
}

// Protect routes
app.get('/api/posts', requirePermission('read:posts'), (req, res) => {
  // Return posts
});

app.post('/api/posts', requirePermission('write:posts'), (req, res) => {
  // Create post
});

app.delete('/api/posts/:id', requirePermission('delete:all'), (req, res) => {
  // Delete post
});
```

**How it works:**

1. **requirePermission Middleware**: Creates a reusable function that checks for specific permissions before route execution
2. **Permission Check**: Extracts permissions from the authenticated user object and validates the required permission
3. **403 Response**: Returns a Forbidden status if the user lacks the required permission
4. **Route Protection**: Each API endpoint is protected with the appropriate permission requirement
5. **Result**: Only authorized users can access each protected endpoint

## Step 4: Handle Role-Based Navigation

```javascript
// Navigation component with role checking
function Navigation({ user }) {
  const permissions = getUserPermissions(user);
  
  return (
    <nav>
      <Link to="/dashboard">Dashboard</Link>
      
      {permissions.includes('read:posts') && (
        <Link to="/posts">Posts</Link>
      )}
      
      {permissions.includes('write:posts') && (
        <Link to="/create-post">Create Post</Link>
      )}
      
      {permissions.includes('delete:all') && (
        <Link to="/admin">Admin Panel</Link>
      )}
    </nav>
  );
}
```

## Best Practices

### Security
- Always verify permissions on the backend
- Use specific permission names (e.g., `read:posts` not `read`)
- Implement the principle of least privilege

### Performance
- Cache user permissions in your app state
- Avoid checking permissions on every render
- Use React Context or state management for permissions

### User experience
- Show appropriate error messages for insufficient permissions
- Hide UI elements users can't access
- Provide clear feedback about user capabilities

## Example: Complete Implementation

The following React component demonstrates a complete role-based access control implementation that dynamically shows different routes based on user permissions.

```javascript
// React component with RBAC
import { useAuth } from '@protekt/react';

function App() {
  const { user, isAuthenticated } = useAuth();
  
  if (!isAuthenticated) {
    return <Login />;
  }
  
  const permissions = user['https://your-app.com/permissions'] || [];
  
  return (
    <div>
      <Navigation user={user} />
      
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        
        {permissions.includes('read:posts') && (
          <Route path="/posts" element={<PostList />} />
        )}
        
        {permissions.includes('write:posts') && (
          <Route path="/create-post" element={<CreatePost />} />
        )}
        
        {permissions.includes('delete:all') && (
          <Route path="/admin" element={<AdminPanel />} />
        )}
      </Routes>
    </div>
  );
}
```
**How it works:**

**Authentication and permission setup**: The `useAuth` hook retrieves the current user and authentication status from Protekt's React SDK. If the user isn't authenticated, they're immediately redirected to the login component. Once authenticated, permissions are extracted from the user's JWT token using a custom claim namespace (`https://your-app.com/permissions`).

**Dynamic routing for enhance security**: Each route is conditionally rendered based on whether the user has the required permission - the `/posts` route only appears if user has `read:posts` permission, `/create-post` route only appears if user has `write:posts` permission, and `/admin` route only appears if user has `delete:all` permission. The navigation component receives the user object to show appropriate menu items, creating a security layer where routes that don't match user permissions are completely hidden from the routing system.


## Next Steps

- Explore [User Management API](../api/user-management) for advanced user operations
- Check out [Roles & Permissions API](../api/roles-permissions) for programmatic role management
- Review [Authentication API](../api/authentication) for token customization

---

Need help implementing RBAC? Contact our [developer support team](mailto:developers@protekt.dev).
