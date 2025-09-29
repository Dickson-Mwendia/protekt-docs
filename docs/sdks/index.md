---
sidebar_position: 1
title: SDKs Overview
description: Official Protekt SDKs for integrating authentication into your applications
---

# SDKs Overview

Protekt provides official SDKs for popular programming languages and frameworks to make integration fast and secure. Choose the SDK that matches your technology stack.

## Available SDKs

### Frontend SDKs

- **[JavaScript SDK](./javascript)** - For web applications using vanilla JavaScript, React, Vue.js, or other frontend frameworks
- **React SDK** *(Coming Soon)* - Specialized hooks and components for React applications
- **Vue SDK** *(Coming Soon)* - Vue.js specific composables and components

### Backend SDKs

- **[Node.js SDK](./node)** - For server-side JavaScript applications using Express, Fastify, or Next.js
- **[Python SDK](./python)** - For Python applications using Flask, Django, or FastAPI
- **Go SDK** *(Coming Soon)* - For Go applications and microservices
- **Java SDK** *(Coming Soon)* - For Java and Spring Boot applications

### Mobile SDKs

- **iOS SDK** *(Coming Soon)* - Native iOS applications in Swift
- **Android SDK** *(Coming Soon)* - Native Android applications in Kotlin/Java
- **React Native SDK** *(Coming Soon)* - Cross-platform mobile applications
- **Flutter SDK** *(Coming Soon)* - Cross-platform mobile applications

## Features Comparison

| Feature | JavaScript | Node.js | Python | Coming Soon |
|---------|------------|---------|--------|-------------|
| Authentication | ✅ | ✅ | ✅ | ✅ |
| JWT Verification | ✅ | ✅ | ✅ | ✅ |
| User Management | ❌ | ✅ | ✅ | ✅ |
| Role Management | ❌ | ✅ | ✅ | ✅ |
| Social Login | ✅ | ✅ | ✅ | ✅ |
| MFA Support | ✅ | ✅ | ✅ | ✅ |
| Custom Claims | ✅ | ✅ | ✅ | ✅ |

## Quick Start Guide

### 1. Choose Your SDK

Select the SDK that matches your application:

- **Building a web app?** → JavaScript SDK
- **Building an API?** → Node.js or Python SDK  
- **Building a full-stack app?** → Use both frontend and backend SDKs

### 2. Install the SDK

Each SDK can be installed via the standard package manager for that language:

```bash
# JavaScript/Node.js
npm install @protekt/js
npm install @protekt/node

# Python  
pip install protekt-python
```

### 3. Configure Authentication

Get your configuration from the Protekt Dashboard:

1. Go to **Applications** → **Your App** → **Settings**
2. Copy your **Domain**, **Client ID**, and **Client Secret**
3. Configure your chosen SDK with these values

### 4. Implement Authentication

Follow the specific guide for your chosen SDK to implement login, logout, and user management.

## Common Integration Patterns

### Single Page Application (SPA)

For React, Vue, or Angular applications:

- Use the **JavaScript SDK** for frontend authentication
- Use **Node.js SDK** or **Python SDK** for API protection
- Implement token-based authentication flow

### Server-Side Rendered Applications

For Next.js, Nuxt, or traditional server apps:

- Use server-side SDKs for both authentication and rendering
- Implement session-based or hybrid authentication
- Handle both server and client-side authentication states

### Microservices Architecture

For distributed systems:

- Use backend SDKs in each service for JWT verification
- Implement centralized user management
- Share user context across services via JWT claims

### Mobile Applications

For iOS, Android, or cross-platform apps:

- Use mobile-specific SDKs (coming soon)
- Implement native authentication flows
- Handle token storage and refresh securely

## Getting Help

### Documentation
- Each SDK has comprehensive documentation with examples
- API references are available for all methods and configurations
- Integration guides for popular frameworks

### Community Support
- [GitHub Discussions](https://github.com/protekt/sdks/discussions)
- [Community Forum](https://community.protekt.dev)
- [Discord Server](https://discord.gg/protekt)

### Professional Support
- Priority support available for enterprise customers
- Custom SDK development for specific use cases
- Integration consulting and code reviews

## Contributing

We welcome contributions to our SDKs:

- [JavaScript SDK GitHub](https://github.com/protekt/js-sdk)
- [Node.js SDK GitHub](https://github.com/protekt/node-sdk)  
- [Python SDK GitHub](https://github.com/protekt/python-sdk)

### Feature Requests

Missing a feature or SDK for your language? Let us know:

- [Create a feature request](https://github.com/protekt/sdks/issues/new?template=feature_request.md)
- [Join our Discord](https://discord.gg/protekt) to discuss with the team
- [Community Forum](https://community.protekt.dev) for broader discussions
