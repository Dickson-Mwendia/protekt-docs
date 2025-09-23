---
title: "Quickstart Guide"
description: "Get authentication working in your app in under 30 minutes. Step-by-step guide to integrate Protekt authentication with working code examples."
author: "Dickson-Mwendia"
reviewer: "John Doe, Senior Product Manager"
date: "2025-09-24"
keywords: 
  - quickstart
  - authentication setup
  - getting started
  - OAuth implementation
tags:
  - getting-started
  - quickstart
  - authentication
customer_intent: "As a developer, I want to quickly implement secure authentication in my application using Protekt with minimal setup time and achieve my first successful login flow."
---

# Quickstart Guide

Get up and running with Protekt authentication in under 30 minutes. This guide will walk you through setting up your first application, implementing login functionality, and securing your users.

## What You'll Build

By the end of this guide, you'll have:

- ✅ A Protekt application configured
- ✅ Login and logout functionality 
- ✅ User profile information display
- ✅ Protected routes in your application
- ✅ Working authentication flow

## Prerequisites

Before starting, make sure you have:

- A modern web browser
- Node.js 16+ installed (for JavaScript examples)
- Basic knowledge of HTML, CSS, and JavaScript
- A code editor (VS Code recommended)

## Step 1: Create Your Protekt Account

1. **Sign up** at [dashboard.protekt.dev](https://dashboard.protekt.dev/signup)
2. **Verify your email** address
3. **Complete the onboarding** by providing your company information

:::tip Free Tier Included
Protekt's free tier includes up to 1,000 monthly active users with all core features. Perfect for development and small projects!
:::

## Step 2: Create Your First Application

1. **Navigate to Applications** in your Protekt dashboard
2. **Click "Create Application"**
3. **Configure your application**:
   ```
   Application Name: My First App
   Application Type: Single Page Application (SPA)
   Allowed Callback URLs: http://localhost:3000/callback
   Allowed Logout URLs: http://localhost:3000
   ```
4. **Save your Application ID and Secret** - you'll need these in the next steps

:::warning Keep Your Secret Safe
Your Application Secret should never be exposed in client-side code. For SPAs, we'll use the PKCE flow which doesn't require the secret.
:::

## Step 3: Set Up Your Project

We'll build our demo application incrementally, adding authentication pieces step by step.

### 3.1: Create the Basic HTML Structure

First, let's create a basic HTML file with styling:

```html title="index.html"
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My First Protekt App</title>
    <style>
        body { 
            font-family: Arial, sans-serif; 
            max-width: 800px; 
            margin: 50px auto; 
            padding: 20px;
        }
        .user-info { 
            background: #f5f5f5; 
            padding: 20px; 
            border-radius: 8px; 
            margin: 20px 0; 
        }
        .hidden { display: none; }
        button { 
            background: #007bff; 
            color: white; 
            border: none; 
            padding: 10px 20px; 
            border-radius: 4px; 
            cursor: pointer; 
            margin: 5px;
        }
        button:hover { background: #0056b3; }
    </style>
</head>
<body>
    <h1>🛡️ My First Protekt App</h1>
    
    <!-- We'll add content here in the next steps -->
    
</body>
</html>
```

### 3.2: Add the User Interface Elements

Now let's add the login and user profile sections:

```html title="index.html (add inside <body>)"
    <!-- Login Section - shown when user is not authenticated -->
    <div id="login-section">
        <h2>Welcome! Please log in to continue.</h2>
        <button onclick="login()">Login with Protekt</button>
    </div>
    
    <!-- User Profile Section - shown when user is authenticated -->
    <div id="profile-section" class="hidden">
        <h2>Welcome back!</h2>
        <div id="user-info" class="user-info"></div>
        <button onclick="logout()">Logout</button>
    </div>
```

This creates two main sections: one for login and one for displaying user info. We use CSS classes to show/hide sections based on authentication state, and include placeholder functions for login/logout that we'll implement in the following steps.

### 3.3: Include the Protekt SDK

Add the Protekt JavaScript SDK before your custom script:

```html title="index.html (add before closing </body>)"
    <script src="https://cdn.jsdelivr.net/npm/@protekt/browser@latest/dist/protekt-browser.min.js"></script>
```

This loads the Protekt browser SDK from CDN, which provides the `Protekt` class for authentication operations and handles the OAuth 2.0/PKCE flow automatically.

### 3.4: Initialize the Protekt Client

Add your configuration and initialize the Protekt client:

```html title="index.html (add after the SDK script)"
    <script>
        // Replace with your actual Application ID and Domain
        const PROTEKT_APP_ID = 'your_app_id_here';
        const PROTEKT_DOMAIN = 'your_domain.protekt.dev';
        
        // Initialize Protekt client with your configuration
        const protekt = new Protekt({
            domain: PROTEKT_DOMAIN,
            clientId: PROTEKT_APP_ID,
            redirectUri: window.location.origin + '/callback'
        });
    </script>
```

This sets up your application credentials (you'll replace these with real values from your dashboard), initializes the Protekt client with OAuth configuration, and configures the callback URL where users return after authentication.

### 3.5: Add Authentication State Detection

Add code to check if the user is already logged in:

```javascript title="index.html (add to your script section)"
        // Check authentication status when page loads
        window.addEventListener('load', async () => {
            try {
                const isAuthenticated = await protekt.isAuthenticated();
                if (isAuthenticated) {
                    showProfile();
                } else {
                    showLogin();
                }
            } catch (error) {
                console.error('Error checking authentication:', error);
                showLogin();
            }
        });
        
        // Helper functions to show/hide UI sections
        function showLogin() {
            document.getElementById('login-section').classList.remove('hidden');
            document.getElementById('profile-section').classList.add('hidden');
        }
        
        function showProfile() {
            document.getElementById('login-section').classList.add('hidden');
            document.getElementById('profile-section').classList.remove('hidden');
        }
```

This code checks authentication status when the page loads, shows the appropriate UI section based on user's login state, and provides helper functions to switch between login and profile views.

### 3.6: Implement Login Functionality

Add the login function that redirects to Protekt:

```javascript title="index.html (add to your script section)"
        // Login function - redirects to Protekt authentication
        async function login() {
            try {
                await protekt.loginWithRedirect();
                // User will be redirected to Protekt's login page
            } catch (error) {
                console.error('Login error:', error);
                alert('Login failed. Please try again.');
            }
        }
```

This function initiates the OAuth 2.0 authorization flow, redirects the user to Protekt's secure login page, and handles any errors during the login process.

### 3.7: Implement Logout Functionality

Add the logout function:

```javascript title="index.html (add to your script section)"
        // Logout function - clears authentication and redirects
        async function logout() {
            try {
                await protekt.logout({
                    returnTo: window.location.origin
                });
            } catch (error) {
                console.error('Logout error:', error);
            }
        }
```

This function clears the user's authentication tokens, redirects back to your application, and ensures a clean logout from Protekt's session.

### 3.8: Handle Authentication Callback

Add code to handle the return from Protekt's login page:

```javascript title="index.html (add to your script section)"
        // Handle callback after successful authentication
        if (window.location.search.includes('code=')) {
            protekt.handleRedirectCallback()
                .then(() => {
                    showProfile();
                    loadUserProfile();
                    // Clean up URL parameters
                    window.history.replaceState({}, document.title, window.location.pathname);
                })
                .catch(error => {
                    console.error('Callback error:', error);
                    alert('Authentication failed. Please try logging in again.');
                    showLogin();
                });
        }
```

This code detects when the user returns from Protekt with an authorization code, exchanges the code for authentication tokens, cleans up the URL to remove OAuth parameters, and shows the user profile on success.

### 3.9: Display User Information

Finally, add the function to load and display user data:

```javascript title="index.html (add to your script section)"
        // Load and display user profile information
        async function loadUserProfile() {
            try {
                const user = await protekt.getUser();
                
                document.getElementById('user-info').innerHTML = `
                    <h3>User Information</h3>
                    <p><strong>Name:</strong> ${user.name || 'Not provided'}</p>
                    <p><strong>Email:</strong> ${user.email}</p>
                    <p><strong>User ID:</strong> ${user.sub}</p>
                    <p><strong>Last Login:</strong> ${new Date(user.updated_at).toLocaleString()}</p>
                `;
            } catch (error) {
                console.error('Error fetching user:', error);
                document.getElementById('user-info').innerHTML = '<p>Error loading user information</p>';
            }
        }
```

This function fetches user profile data from Protekt, displays user information in a formatted way, and handles errors gracefully if user data can't be loaded.

### 3.10: Complete HTML File

Here's your complete `index.html` file with all pieces together:

```html title="index.html"
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My First Protekt App</title>
    <style>
        body { 
            font-family: Arial, sans-serif; 
            max-width: 800px; 
            margin: 50px auto; 
            padding: 20px;
        }
        .user-info { 
            background: #f5f5f5; 
            padding: 20px; 
            border-radius: 8px; 
            margin: 20px 0; 
        }
        .hidden { display: none; }
        button { 
            background: #007bff; 
            color: white; 
            border: none; 
            padding: 10px 20px; 
            border-radius: 4px; 
            cursor: pointer; 
            margin: 5px;
        }
        button:hover { background: #0056b3; }
    </style>
</head>
<body>
    <h1>🛡️ My First Protekt App</h1>
    
    <!-- Login Section -->
    <div id="login-section">
        <h2>Welcome! Please log in to continue.</h2>
        <button onclick="login()">Login with Protekt</button>
    </div>
    
    <!-- User Profile Section -->
    <div id="profile-section" class="hidden">
        <h2>Welcome back!</h2>
        <div id="user-info" class="user-info"></div>
        <button onclick="logout()">Logout</button>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/@protekt/browser@latest/dist/protekt-browser.min.js"></script>
    <script>
        // Replace with your actual Application ID
        const PROTEKT_APP_ID = 'your_app_id_here';
        const PROTEKT_DOMAIN = 'your_domain.protekt.dev';
        
        // Initialize Protekt client
        const protekt = new Protekt({
            domain: PROTEKT_DOMAIN,
            clientId: PROTEKT_APP_ID,
            redirectUri: window.location.origin + '/callback'
        });

        // Check if user is already authenticated
        window.addEventListener('load', async () => {
            try {
                const isAuthenticated = await protekt.isAuthenticated();
                if (isAuthenticated) {
                    showProfile();
                }
            } catch (error) {
                console.error('Error checking authentication:', error);
            }
        });

        // Login function
        async function login() {
            try {
                await protekt.loginWithRedirect();
            } catch (error) {
                console.error('Login error:', error);
                alert('Login failed. Please try again.');
            }
        }

        // Logout function
        async function logout() {
            try {
                await protekt.logout({
                    returnTo: window.location.origin
                });
            } catch (error) {
                console.error('Logout error:', error);
            }
        }

        // Show user profile
        async function showProfile() {
            try {
                const user = await protekt.getUser();
                
                document.getElementById('login-section').classList.add('hidden');
                document.getElementById('profile-section').classList.remove('hidden');
                
                document.getElementById('user-info').innerHTML = `
                    <h3>User Information</h3>
                    <p><strong>Name:</strong> ${user.name || 'Not provided'}</p>
                    <p><strong>Email:</strong> ${user.email}</p>
                    <p><strong>User ID:</strong> ${user.sub}</p>
                    <p><strong>Last Login:</strong> ${new Date(user.updated_at).toLocaleString()}</p>
                `;
            } catch (error) {
                console.error('Error fetching user:', error);
            }
        }

        // Handle callback after login
        if (window.location.search.includes('code=')) {
            protekt.handleRedirectCallback().then(() => {
                showProfile();
                // Clean up URL
                window.history.replaceState({}, document.title, window.location.pathname);
            }).catch(error => {
                console.error('Callback error:', error);
                alert('Authentication failed. Please try logging in again.');
            });
        }
    </script>
</body>
</html>
```

## Step 4: Configure Your Application

1. **Update the configuration** in your HTML file:
   ```javascript
   // Replace these with your actual values from the Protekt dashboard
   const PROTEKT_APP_ID = 'app_1234567890abcdef';
   const PROTEKT_DOMAIN = 'mycompany.protekt.dev';
   ```

2. **Set up a local server** to test your application:
   ```bash
   # Using Python (if installed)
   python -m http.server 3000
   
   # Or using Node.js
   npx serve -p 3000
   
   # Or using VS Code Live Server extension
   # Right-click on index.html → "Open with Live Server"
   ```

## Step 5: Test Your Application

1. **Open your browser** and navigate to `http://localhost:3000`
2. **Click "Login with Protekt"** - you'll be redirected to Protekt's secure login page
3. **Create an account** or log in with existing credentials
4. **Complete any required verification** (email, MFA if enabled)
5. **You'll be redirected back** to your app with user information displayed

## Common Issues & Solutions

### Issue: "Invalid Callback URL"
**Solution**: Ensure your callback URL in the Protekt dashboard matches exactly:
- Development: `http://localhost:3000/callback`
- Production: `https://yourdomain.com/callback`

### Issue: "Application not found"
**Solution**: Double-check your Application ID and domain configuration.

### Issue: CORS Errors
**Solution**: Make sure you're serving your HTML file from a web server, not opening it directly in the browser.

## Code Examples by Framework

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

<Tabs>
<TabItem value="react" label="React" default>

```jsx title="LoginButton.jsx"
import { useAuth } from '@protekt/react';

export function LoginButton() {
    const { login, logout, user, isAuthenticated } = useAuth();
    
    if (isAuthenticated) {
        return (
            <div>
                <p>Welcome, {user.name}!</p>
                <button onClick={logout}>Logout</button>
            </div>
        );
    }
    
    return <button onClick={login}>Login</button>;
}
```

</TabItem>
<TabItem value="vue" label="Vue.js">

```vue title="LoginComponent.vue"
<template>
    <div>
        <div v-if="isAuthenticated">
            <p>Welcome, {{ user.name }}!</p>
            <button @click="logout">Logout</button>
        </div>
        <button v-else @click="login">Login</button>
    </div>
</template>

<script>
import { useProtekt } from '@protekt/vue';

export default {
    setup() {
        const { login, logout, user, isAuthenticated } = useProtekt();
        return { login, logout, user, isAuthenticated };
    }
}
</script>
```

</TabItem>
<TabItem value="express" label="Express.js">

```javascript title="server.js"
const express = require('express');
const { ProtektExpress } = require('@protekt/express');

const app = express();

// Configure Protekt middleware
const protekt = new ProtektExpress({
    domain: 'your_domain.protekt.dev',
    clientId: 'your_client_id',
    clientSecret: 'your_client_secret'
});

// Protect routes
app.get('/api/protected', protekt.requireAuth(), (req, res) => {
    res.json({ 
        message: 'This is a protected route',
        user: req.user 
    });
});

app.listen(3001, () => {
    console.log('API server running on port 3001');
});
```

</TabItem>
</Tabs>


---
