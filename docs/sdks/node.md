---
sidebar_position: 2
title: Node.js SDK
description: Build secure backend services with Protekt authentication using our Node.js SDK
customer_intent: As a backend developer, I want to integrate Protekt authentication into my Node.js application using the Node.js SDK
---

# Node.js SDK

The Protekt Node.js SDK provides server-side authentication capabilities for your Node.js applications, including JWT verification, user management, and API protection.

## Installation

Install the SDK using npm or yarn:

```bash
npm install @protekt/node
# or
yarn add @protekt/node
```

## Quick Start

### Initialize the SDK

```javascript
const { ProtektNode } = require('@protekt/node');

const protekt = new ProtektNode({
  domain: 'your-domain.protekt.dev',
  clientId: 'your-client-id',
  clientSecret: 'your-client-secret',
  audience: 'your-api-audience'
});
```

### JWT Verification Middleware

```javascript
const express = require('express');
const app = express();

// Verify JWT tokens
app.use('/api/protected', protekt.verifyToken(), (req, res) => {
  // Access user information from req.user
  res.json({
    message: 'Protected endpoint accessed',
    user: req.user
  });
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

### User Management

```javascript
// Get user by ID
const user = await protekt.users.getById('user_123');

// Update user metadata
await protekt.users.update('user_123', {
  user_metadata: {
    preferences: { theme: 'dark' }
  }
});

// Create a new user
const newUser = await protekt.users.create({
  email: 'user@example.com',
  password: 'SecurePassword123!',
  user_metadata: {
    firstName: 'John',
    lastName: 'Doe'
  }
});
```

## API Reference

### ProtektNode Class

#### Constructor Options

| Option | Type | Required | Description |
|--------|------|----------|-------------|
| `domain` | string | Yes | Your Protekt domain |
| `clientId` | string | Yes | Your application's client ID |
| `clientSecret` | string | Yes | Your application's client secret |
| `audience` | string | No | API audience identifier |
| `scope` | string | No | Default scopes for API calls |

#### Methods

##### `verifyToken(options?)`

Express middleware to verify JWT tokens.

**Parameters:**
- `options` (optional): Verification options
  - `audience`: Override default audience
  - `algorithms`: Allowed signing algorithms (default: ['RS256'])

**Returns:** Express middleware function

##### `getAccessToken()`

Gets a machine-to-machine access token.

**Returns:** Promise that resolves to access token

### Users API

#### `users.getById(userId)`

Get user information by ID.

**Parameters:**
- `userId`: User identifier

**Returns:** Promise that resolves to user object

#### `users.create(userData)`

Create a new user.

**Parameters:**
- `userData`: User creation data
  - `email`: User's email address
  - `password`: User's password
  - `user_metadata`: Custom user metadata

**Returns:** Promise that resolves to created user

#### `users.update(userId, userData)`

Update user information.

**Parameters:**
- `userId`: User identifier
- `userData`: User update data

**Returns:** Promise that resolves to updated user

#### `users.delete(userId)`

Delete a user.

**Parameters:**
- `userId`: User identifier

**Returns:** Promise that resolves when deletion is complete

### Roles API

#### `roles.assignToUser(userId, roleIds)`

Assign roles to a user.

**Parameters:**
- `userId`: User identifier
- `roleIds`: Array of role IDs

#### `roles.removeFromUser(userId, roleIds)`

Remove roles from a user.

**Parameters:**
- `userId`: User identifier
- `roleIds`: Array of role IDs

## Framework Integration

### Express.js with Authentication

```javascript
const express = require('express');
const { ProtektNode } = require('@protekt/node');

const app = express();
const protekt = new ProtektNode({
  domain: process.env.PROTEKT_DOMAIN,
  clientId: process.env.PROTEKT_CLIENT_ID,
  clientSecret: process.env.PROTEKT_CLIENT_SECRET,
  audience: process.env.PROTEKT_AUDIENCE
});

app.use(express.json());

// Public endpoint
app.get('/api/public', (req, res) => {
  res.json({ message: 'This is a public endpoint' });
});

// Protected endpoint
app.get('/api/profile', protekt.verifyToken(), async (req, res) => {
  try {
    const user = await protekt.users.getById(req.user.sub);
    res.json({ user });
  } catch (error) {
    res.status(500).json({ error: 'Failed to fetch user' });
  }
});

// Admin only endpoint
app.get('/api/admin', 
  protekt.verifyToken(),
  protekt.requireRole('admin'),
  (req, res) => {
    res.json({ message: 'Admin access granted' });
  }
);

app.listen(3000);
```

### Fastify Integration

```javascript
const fastify = require('fastify')({ logger: true });
const { ProtektNode } = require('@protekt/node');

const protekt = new ProtektNode({
  domain: process.env.PROTEKT_DOMAIN,
  clientId: process.env.PROTEKT_CLIENT_ID,
  clientSecret: process.env.PROTEKT_CLIENT_SECRET
});

// Register authentication plugin
fastify.register(async function (fastify) {
  fastify.decorateRequest('user', null);
  
  fastify.addHook('preHandler', async (request, reply) => {
    if (request.routerPath.startsWith('/api/protected')) {
      try {
        const token = request.headers.authorization?.replace('Bearer ', '');
        const user = await protekt.verifyJWT(token);
        request.user = user;
      } catch (error) {
        reply.code(401).send({ error: 'Unauthorized' });
      }
    }
  });
});

fastify.get('/api/protected/profile', async (request, reply) => {
  return { user: request.user };
});

fastify.listen({ port: 3000 });
```

### Next.js API Routes

```javascript
// pages/api/protected.js
import { ProtektNode } from '@protekt/node';

const protekt = new ProtektNode({
  domain: process.env.PROTEKT_DOMAIN,
  clientId: process.env.PROTEKT_CLIENT_ID,
  clientSecret: process.env.PROTEKT_CLIENT_SECRET
});

export default async function handler(req, res) {
  try {
    const token = req.headers.authorization?.replace('Bearer ', '');
    const user = await protekt.verifyJWT(token);
    
    res.status(200).json({
      message: 'Access granted',
      user: user
    });
  } catch (error) {
    res.status(401).json({ error: 'Unauthorized' });
  }
}
```

## Error Handling

```javascript
const { ProtektError } = require('@protekt/node');

try {
  const user = await protekt.users.getById('invalid-id');
} catch (error) {
  if (error instanceof ProtektError) {
    switch (error.code) {
      case 'USER_NOT_FOUND':
        console.log('User does not exist');
        break;
      case 'INSUFFICIENT_PERMISSIONS':
        console.log('Insufficient permissions');
        break;
      default:
        console.log('Protekt API error:', error.message);
    }
  } else {
    console.log('Unexpected error:', error);
  }
}
```

## Configuration

### Environment Variables

Create a `.env` file with your Protekt configuration:

```bash
PROTEKT_DOMAIN=your-domain.protekt.dev
PROTEKT_CLIENT_ID=your-client-id
PROTEKT_CLIENT_SECRET=your-client-secret
PROTEKT_AUDIENCE=your-api-audience
```

### Advanced Configuration

```javascript
const protekt = new ProtektNode({
  domain: process.env.PROTEKT_DOMAIN,
  clientId: process.env.PROTEKT_CLIENT_ID,
  clientSecret: process.env.PROTEKT_CLIENT_SECRET,
  audience: process.env.PROTEKT_AUDIENCE,
  
  // Advanced options
  cache: {
    enabled: true,
    ttl: 3600 // 1 hour
  },
  
  retryOptions: {
    retries: 3,
    retryDelay: 1000
  },
  
  timeout: 5000 // 5 seconds
});
```

## Common Use Cases

### Role-Based Access Control

```javascript
function requireRole(role) {
  return async (req, res, next) => {
    try {
      const userRoles = req.user['https://your-app.com/roles'] || [];
      
      if (!userRoles.includes(role)) {
        return res.status(403).json({ error: 'Insufficient permissions' });
      }
      
      next();
    } catch (error) {
      res.status(500).json({ error: 'Authorization error' });
    }
  };
}

// Usage
app.get('/api/admin-only', 
  protekt.verifyToken(),
  requireRole('admin'),
  (req, res) => {
    res.json({ message: 'Admin endpoint' });
  }
);
```

### User Registration with Custom Data

```javascript
app.post('/api/register', async (req, res) => {
  try {
    const { email, password, firstName, lastName, company } = req.body;
    
    const user = await protekt.users.create({
      email,
      password,
      user_metadata: {
        firstName,
        lastName,
        company,
        registrationDate: new Date().toISOString()
      }
    });
    
    res.status(201).json({ user });
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});
```

## Support

- [GitHub Repository](https://github.com/protekt/node-sdk)
- [Issue Tracker](https://github.com/protekt/node-sdk/issues)
- [Community Forum](https://community.protekt.dev)
