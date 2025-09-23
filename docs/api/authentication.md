---
title: "Authentication API Reference"
description: "Essential API reference for Protekt's core authentication endpoints - login, logout, token validation, and user info."
author: "Protekt API Documentation Team"
reviewer: "Marcus Rodriguez, Lead Developer Advocate"
date: "2025-09-23"
keywords: 
  - authentication API
  - OAuth 2.0
  - JWT tokens
  - login API
  - logout API
  - user info
tags:
  - api-reference
  - authentication
  - oauth
  - jwt
customer_intent: "As a developer, I want to understand and integrate Protekt's core authentication API endpoints using Protekt."
---

# Authentication API Reference

The Protekt Authentication API provides secure OAuth 2.0 endpoints for user authentication, token management, and user information retrieval. This guide covers the essential endpoints you'll use in most applications.

## Base URL

```
https://your-domain.protekt.dev
```

## Core Authentication Flow

Most applications use the **Authorization Code Flow with PKCE**, which is secure for both web and mobile applications.

### 1. Authorize Endpoint

Redirects users to Protekt's secure login page.

#### `GET /oauth/authorize`

**Required Parameters:**

| Parameter | Description | Example |
|-----------|-------------|---------|
| `response_type` | Must be `code` | `code` |
| `client_id` | Your application's client ID | `your_client_id` |
| `redirect_uri` | Where to return after login | `https://yourapp.com/callback` |
| `code_challenge` | PKCE security parameter | `dBjftJeZ4CVP...` |
| `code_challenge_method` | Must be `S256` | `S256` |

**Example Request:**

```bash
GET /oauth/authorize?
  response_type=code&
  client_id=your_client_id&
  redirect_uri=https://yourapp.com/callback&
  code_challenge=dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk&
  code_challenge_method=S256
```

**What happens:** User logs in and returns to your app with an authorization code:

```
https://yourapp.com/callback?code=AUTH_CODE_HERE
```

### 2. Token Endpoint

Exchanges the authorization code for access tokens.

#### `POST /oauth/token`

**Headers:**
```
Content-Type: application/x-www-form-urlencoded
```

**Required Parameters:**

| Parameter | Description |
|-----------|-------------|
| `grant_type` | Must be `authorization_code` |
| `client_id` | Your application's client ID |
| `code` | Authorization code from step 1 |
| `redirect_uri` | Same redirect_uri used in authorize |
| `code_verifier` | Original random string for PKCE |

**Example Request:**

```bash
curl -X POST https://your-domain.protekt.dev/oauth/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=authorization_code" \
  -d "client_id=your_client_id" \
  -d "code=AUTH_CODE_HERE" \
  -d "redirect_uri=https://yourapp.com/callback" \
  -d "code_verifier=your_code_verifier"
```

**Response:**

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "id_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

### 3. Get User Info

Retrieves the authenticated user's profile information.

#### `GET /userinfo`

**Headers:**
```
Authorization: Bearer ACCESS_TOKEN
```

**Example Request:**

```bash
curl -X GET https://your-domain.protekt.dev/userinfo \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9..."
```

**Response:**

```json
{
  "sub": "protekt|507f1f77bcf86cd799439011",
  "name": "John Doe",
  "email": "john.doe@example.com",
  "email_verified": true,
  "picture": "https://example.com/avatar.jpg",
  "updated_at": "2025-09-23T10:30:00.000Z"
}
```

### 4. Validate Token

Check if an access token is valid and get token information.

#### `POST /oauth/introspect`

**Headers:**
```
Content-Type: application/x-www-form-urlencoded
Authorization: Basic base64(client_id:client_secret)
```

**Parameters:**

| Parameter | Description |
|-----------|-------------|
| `token` | The access token to validate |

**Example Request:**

```bash
curl -X POST https://your-domain.protekt.dev/oauth/introspect \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -H "Authorization: Basic $(echo -n 'client_id:client_secret' | base64)" \
  -d "token=eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9..."
```

**Response (Valid Token):**

```json
{
  "active": true,
  "client_id": "your_client_id",
  "username": "john.doe@example.com",
  "sub": "protekt|507f1f77bcf86cd799439011",
  "exp": 1695456000,
  "iat": 1695452400
}
```

**Response (Invalid Token):**

```json
{
  "active": false
}
```

### 5. Logout

Terminates the user's session and optionally redirects to a specified URL.

#### `GET /logout`

**Parameters:**

| Parameter | Description |
|-----------|-------------|
| `returnTo` | URL to redirect after logout (optional) |

**Example Request:**

```bash
GET /logout?returnTo=https://yourapp.com/goodbye
```

## SDK Integration Examples

These examples show how the API endpoints are used in practice with Protekt SDKs:

### JavaScript (Browser)

```javascript
// Initialize Protekt client
const protekt = new Protekt({
  domain: 'your-domain.protekt.dev',
  clientId: 'your_client_id',
  redirectUri: window.location.origin + '/callback'
});

// Login - redirects to /oauth/authorize
async function login() {
  await protekt.loginWithRedirect();
}

// Handle callback - exchanges code at /oauth/token
if (window.location.search.includes('code=')) {
  await protekt.handleRedirectCallback();
}

// Get user info - calls /userinfo
async function getUserInfo() {
  const user = await protekt.getUser();
  return user;
}

// Check if authenticated - validates token
async function checkAuth() {
  const isAuthenticated = await protekt.isAuthenticated();
  return isAuthenticated;
}

// Logout - calls /logout
async function logout() {
  await protekt.logout({
    returnTo: window.location.origin
  });
}
```

### Node.js (Backend)

```javascript
const express = require('express');
const { ProtektExpress } = require('@protekt/express');

const app = express();

// Configure Protekt middleware
const protekt = new ProtektExpress({
  domain: 'your-domain.protekt.dev',
  clientId: 'your_client_id',
  clientSecret: 'your_client_secret'
});

// Protected route - validates token using /oauth/introspect
app.get('/api/protected', protekt.requireAuth(), (req, res) => {
  // req.user contains user info from /userinfo
  res.json({ 
    message: 'Protected data',
    user: req.user 
  });
});
```

## JWT Token Structure

Access tokens are JWTs with this basic structure:

```json
{
  "iss": "https://your-domain.protekt.dev/",
  "sub": "protekt|507f1f77bcf86cd799439011",
  "aud": "your_client_id",
  "exp": 1695456000,
  "iat": 1695452400,
  "scope": "openid profile email"
}
```

## Error Handling

### Common Error Responses

| Error Code | Description | Solution |
|------------|-------------|----------|
| `invalid_client` | Client ID/secret incorrect | Check your credentials |
| `invalid_grant` | Authorization code expired | Start auth flow again |
| `invalid_request` | Missing required parameters | Check request format |
| `access_denied` | User denied login | Handle gracefully |

### Example Error Response

```json
{
  "error": "invalid_grant",
  "error_description": "The authorization code is invalid or expired"
}
```

## Security Best Practices

1. **Always use HTTPS** in production
2. **Use PKCE** for all OAuth flows
3. **Validate tokens** on your backend
4. **Store tokens securely** (HttpOnly cookies recommended)
5. **Implement proper logout** to clear sessions

## Rate Limits

| Endpoint | Limit | Window |
|----------|-------|--------|
| `/oauth/authorize` | 100 requests | 5 minutes |
| `/oauth/token` | 50 requests | 1 minute |
| `/userinfo` | 1000 requests | 1 minute |

## Quick Setup Checklist

1. ✅ Get your `client_id` from Protekt dashboard
2. ✅ Configure your `redirect_uri` in dashboard
3. ✅ Implement authorization flow (redirect to `/oauth/authorize`)
4. ✅ Handle callback (exchange code at `/oauth/token`)
5. ✅ Get user info from `/userinfo`
6. ✅ Implement logout (redirect to `/logout`)

## Related Resources

- [Quickstart Guide](../getting-started/quickstart) - Complete implementation example
- [RBAC Guide](../guides/role-based-access-control) - Adding roles and permissions
- [User Management API](./user-management) - Managing users programmatically

---

For questions about the Authentication API, contact [developers@protekt.dev](mailto:developers@protekt.dev) or visit our [community forum](https://community.protekt.dev).
