---
sidebar_position: 3
title: Python SDK
description: Integrate Protekt authentication into your Python applications with our official SDK
customer_intent: As a Python developer, I want to integrate Protekt authentication into my Python application using the Python SDK
---

# Python SDK

The Protekt Python SDK provides authentication capabilities for Python applications, including Flask, Django, FastAPI, and more. It handles JWT verification, user management, and API protection.

## Installation

Install the SDK using pip:

```bash
pip install protekt-python
```

## Quick Start

### Initialize the SDK

```python
from protekt import ProtektAuth

protekt = ProtektAuth(
    domain='your-domain.protekt.dev',
    client_id='your-client-id',
    client_secret='your-client-secret',
    audience='your-api-audience'
)
```

### JWT Verification

```python
from flask import Flask, request, jsonify
from protekt import ProtektAuth
from protekt.decorators import requires_auth

app = Flask(__name__)

protekt = ProtektAuth(
    domain='your-domain.protekt.dev',
    client_id='your-client-id',
    client_secret='your-client-secret'
)

@app.route('/api/protected')
@requires_auth
def protected():
    user = request.current_user
    return jsonify({
        'message': 'Protected endpoint accessed',
        'user': user
    })

if __name__ == '__main__':
    app.run()
```

### User Management

```python
# Get user by ID
user = protekt.users.get('user_123')

# Update user
updated_user = protekt.users.update('user_123', {
    'user_metadata': {
        'preferences': {'theme': 'dark'}
    }
})

# Create new user
new_user = protekt.users.create({
    'email': 'user@example.com',
    'password': 'SecurePassword123!',
    'user_metadata': {
        'first_name': 'John',
        'last_name': 'Doe'
    }
})
```

## API Reference

### ProtektAuth Class

#### Constructor Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `domain` | str | Yes | Your Protekt domain |
| `client_id` | str | Yes | Your application's client ID |
| `client_secret` | str | Yes | Your application's client secret |
| `audience` | str | No | API audience identifier |
| `algorithm` | str | No | JWT algorithm (default: 'RS256') |

#### Methods

##### `verify_token(token)`

Verify a JWT token and return user information.

**Parameters:**
- `token`: JWT token string

**Returns:** Dictionary containing user information

##### `get_access_token()`

Get a machine-to-machine access token.

**Returns:** Access token string

### Users API

#### `users.get(user_id)`

Get user information by ID.

**Parameters:**
- `user_id`: User identifier

**Returns:** User dictionary

#### `users.create(user_data)`

Create a new user.

**Parameters:**
- `user_data`: Dictionary containing user data

**Returns:** Created user dictionary

#### `users.update(user_id, user_data)`

Update user information.

**Parameters:**
- `user_id`: User identifier
- `user_data`: Dictionary containing updated data

**Returns:** Updated user dictionary

#### `users.delete(user_id)`

Delete a user.

**Parameters:**
- `user_id`: User identifier

**Returns:** Boolean indicating success

### Decorators

#### `@requires_auth`

Decorator to protect Flask routes with authentication.

```python
from protekt.decorators import requires_auth

@app.route('/protected')
@requires_auth
def protected_route():
    # Access current user via request.current_user
    return jsonify({'user': request.current_user})
```

#### `@requires_role(role)`

Decorator to require specific roles.

```python
from protekt.decorators import requires_role

@app.route('/admin')
@requires_role('admin')
def admin_route():
    return jsonify({'message': 'Admin access granted'})
```

## Framework Integration

### Flask Integration

```python
from flask import Flask, request, jsonify
from protekt import ProtektAuth
from protekt.decorators import requires_auth, requires_role

app = Flask(__name__)

protekt = ProtektAuth(
    domain='your-domain.protekt.dev',
    client_id='your-client-id',
    client_secret='your-client-secret'
)

# Configure the app with Protekt
app.config['PROTEKT'] = protekt

@app.route('/api/public')
def public():
    return jsonify({'message': 'This is a public endpoint'})

@app.route('/api/profile')
@requires_auth
def profile():
    user = request.current_user
    return jsonify({'user': user})

@app.route('/api/admin')
@requires_auth
@requires_role('admin')
def admin():
    return jsonify({'message': 'Admin endpoint'})

if __name__ == '__main__':
    app.run(debug=True)
```

### Django Integration

```python
# settings.py
PROTEKT_CONFIG = {
    'DOMAIN': 'your-domain.protekt.dev',
    'CLIENT_ID': 'your-client-id',
    'CLIENT_SECRET': 'your-client-secret',
    'AUDIENCE': 'your-api-audience'
}

# middleware.py
from django.utils.deprecation import MiddlewareMixin
from protekt import ProtektAuth
from django.conf import settings

class ProtektMiddleware(MiddlewareMixin):
    def __init__(self, get_response):
        self.get_response = get_response
        self.protekt = ProtektAuth(**settings.PROTEKT_CONFIG)

    def process_request(self, request):
        auth_header = request.META.get('HTTP_AUTHORIZATION', '')
        if auth_header.startswith('Bearer '):
            token = auth_header.split(' ')[1]
            try:
                user = self.protekt.verify_token(token)
                request.user = user
            except Exception:
                request.user = None

# views.py
from django.http import JsonResponse
from django.views.decorators.csrf import csrf_exempt
from protekt.django import requires_auth

@csrf_exempt
@requires_auth
def protected_view(request):
    return JsonResponse({
        'message': 'Protected endpoint',
        'user': request.user
    })
```

### FastAPI Integration

```python
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from protekt import ProtektAuth

app = FastAPI()
security = HTTPBearer()

protekt = ProtektAuth(
    domain='your-domain.protekt.dev',
    client_id='your-client-id',
    client_secret='your-client-secret'
)

async def get_current_user(credentials: HTTPAuthorizationCredentials = Depends(security)):
    try:
        user = protekt.verify_token(credentials.credentials)
        return user
    except Exception:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid authentication credentials",
            headers={"WWW-Authenticate": "Bearer"},
        )

@app.get("/api/public")
async def public_endpoint():
    return {"message": "This is a public endpoint"}

@app.get("/api/protected")
async def protected_endpoint(current_user: dict = Depends(get_current_user)):
    return {
        "message": "Protected endpoint accessed",
        "user": current_user
    }

@app.get("/api/admin")
async def admin_endpoint(current_user: dict = Depends(get_current_user)):
    roles = current_user.get('https://your-app.com/roles', [])
    if 'admin' not in roles:
        raise HTTPException(status_code=403, detail="Insufficient permissions")
    
    return {"message": "Admin access granted"}
```

## Error Handling

```python
from protekt.exceptions import ProtektError, InvalidTokenError, UserNotFoundError

try:
    user = protekt.users.get('invalid-user-id')
except UserNotFoundError:
    print("User does not exist")
except InvalidTokenError:
    print("Invalid or expired token")
except ProtektError as e:
    print(f"Protekt API error: {e}")
except Exception as e:
    print(f"Unexpected error: {e}")
```

## Configuration

### Environment Variables

```python
import os
from protekt import ProtektAuth

protekt = ProtektAuth(
    domain=os.getenv('PROTEKT_DOMAIN'),
    client_id=os.getenv('PROTEKT_CLIENT_ID'),
    client_secret=os.getenv('PROTEKT_CLIENT_SECRET'),
    audience=os.getenv('PROTEKT_AUDIENCE')
)
```

### Advanced Configuration

```python
from protekt import ProtektAuth

protekt = ProtektAuth(
    domain='your-domain.protekt.dev',
    client_id='your-client-id',
    client_secret='your-client-secret',
    
    # Advanced options
    cache_enabled=True,
    cache_ttl=3600,  # 1 hour
    timeout=5,  # 5 seconds
    retry_attempts=3,
    retry_delay=1.0
)
```

## Common Use Cases

### Role-Based Access Control

```python
from flask import Flask, jsonify
from protekt.decorators import requires_auth, requires_permission

app = Flask(__name__)

@app.route('/api/posts', methods=['GET'])
@requires_auth
@requires_permission('read:posts')
def get_posts():
    return jsonify({'posts': []})

@app.route('/api/posts', methods=['POST'])
@requires_auth
@requires_permission('write:posts')
def create_post():
    return jsonify({'message': 'Post created'})

@app.route('/api/admin/users')
@requires_auth
@requires_role('admin')
def admin_users():
    users = protekt.users.list()
    return jsonify({'users': users})
```

### Custom User Registration

```python
@app.route('/api/register', methods=['POST'])
def register():
    data = request.get_json()
    
    try:
        user = protekt.users.create({
            'email': data['email'],
            'password': data['password'],
            'user_metadata': {
                'first_name': data['first_name'],
                'last_name': data['last_name'],
                'company': data.get('company'),
                'registration_date': datetime.utcnow().isoformat()
            }
        })
        
        return jsonify({'user': user}), 201
    except Exception as e:
        return jsonify({'error': str(e)}), 400
```

### Token Validation Middleware

```python
from functools import wraps
from flask import request, jsonify

def token_required(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        token = request.headers.get('Authorization')
        
        if not token:
            return jsonify({'error': 'Token is missing'}), 401
        
        try:
            token = token.split(' ')[1]  # Remove 'Bearer ' prefix
            user = protekt.verify_token(token)
            request.current_user = user
        except Exception:
            return jsonify({'error': 'Token is invalid'}), 401
        
        return f(*args, **kwargs)
    
    return decorated

@app.route('/api/protected')
@token_required
def protected():
    return jsonify({'user': request.current_user})
```

## Testing

```python
import unittest
from unittest.mock import Mock, patch
from your_app import app

class TestProtektIntegration(unittest.TestCase):
    def setUp(self):
        self.app = app.test_client()
        self.app.testing = True

    @patch('protekt.ProtektAuth.verify_token')
    def test_protected_endpoint(self, mock_verify):
        mock_verify.return_value = {
            'sub': 'user_123',
            'email': 'test@example.com'
        }
        
        response = self.app.get('/api/protected', 
                               headers={'Authorization': 'Bearer fake_token'})
        
        self.assertEqual(response.status_code, 200)
        mock_verify.assert_called_once()

    def test_public_endpoint(self):
        response = self.app.get('/api/public')
        self.assertEqual(response.status_code, 200)

if __name__ == '__main__':
    unittest.main()
```

## Support

- [GitHub Repository](https://github.com/protekt/python-sdk)
- [Issue Tracker](https://github.com/protekt/python-sdk/issues)  
- [Community Forum](https://community.protekt.dev)
