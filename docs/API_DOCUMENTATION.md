# Secure Messenger - API Documentation

## Base URL

All API endpoints are prefixed with `/api`.

**Example**: `https://yourdomain.com/api/login`

## Authentication

The API uses **cookie-based authentication** with HttpOnly cookies for security. Session tokens are stored in cookies and automatically sent with requests.

### Alternative: Bearer Token (Backward Compatibility)

For API clients, Bearer token authentication is also supported:

```
Authorization: Bearer <session_token>
```

**Note**: Cookie-based authentication is recommended for web clients as it provides better XSS protection.

## Response Format

### Success Response

Most endpoints return JSON with a `status` field:

```json
{
  "status": "ok",
  ...
}
```

### Error Response

Errors return appropriate HTTP status codes with error details:

```json
{
  "detail": "Error message describing what went wrong"
}
```

### HTTP Status Codes

- `200 OK`: Request successful
- `400 Bad Request`: Invalid input or request format
- `401 Unauthorized`: Authentication required or failed
- `404 Not Found`: Resource not found
- `500 Internal Server Error`: Server-side error

## Endpoints

### Authentication Endpoints

#### Register User

Create a new user account.

**Endpoint**: `POST /api/register`

**Request Body**:
```json
{
  "username": "alice",
  "password": "SecurePass123!",
  "email": "alice@example.com"  // Optional
}
```

**Response** (200 OK):
```json
{
  "status": "ok"
}
```

**Error Responses**:
- `400 Bad Request`: Password doesn't meet complexity requirements or username already exists
  ```json
  {
    "detail": "Password must be at least 8 characters long"
  }
  ```

**Password Requirements**:
- Minimum 8 characters
- At least one letter (a-z, A-Z)
- At least one number (0-9)
- At least one special character (!@#$%^&*()_+-=[]{}|;:,.<>?)

**Example**:
```bash
curl -X POST https://yourdomain.com/api/register \
  -H "Content-Type: application/json" \
  -d '{
    "username": "alice",
    "password": "SecurePass123!",
    "email": "alice@example.com"
  }'
```

---

#### Login

Authenticate and create a session.

**Endpoint**: `POST /api/login`

**Request Body**:
```json
{
  "username": "alice",
  "password": "SecurePass123!",
  "totp_token": "123456"  // Optional, required if 2FA is enabled
}
```

**Response** (200 OK):
```json
{
  "status": "ok",
  "username": "alice",
  "token": "session_token_here"  // Also set as HttpOnly cookie
}
```

**Response** (200 OK - TOTP Required):
```json
{
  "requires_totp": true,
  "message": "TOTP token required"
}
```

**Error Responses**:
- `401 Unauthorized`: Invalid credentials or TOTP token
  ```json
  {
    "detail": "Invalid credentials"
  }
  ```

**Example**:
```bash
curl -X POST https://yourdomain.com/api/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "alice",
    "password": "SecurePass123!"
  }' \
  -c cookies.txt
```

---

#### Logout

Terminate the current session.

**Endpoint**: `POST /api/logout`

**Authentication**: Required

**Response** (200 OK):
```json
{
  "status": "ok"
}
```

**Example**:
```bash
curl -X POST https://yourdomain.com/api/logout \
  -b cookies.txt
```

---

### TOTP Management Endpoints

#### Setup TOTP

Generate a TOTP secret and QR code for two-factor authentication.

**Endpoint**: `POST /api/totp/setup`

**Authentication**: Required

**Response** (200 OK):
```json
{
  "status": "ok",
  "secret": "JBSWY3DPEHPK3PXP",
  "qr_code": "data:image/png;base64,iVBORw0KGgoAAAANS...",
  "message": "Scan QR code with your authenticator app"
}
```

**Error Responses**:
- `401 Unauthorized`: Not authenticated
- `400 Bad Request`: User not found or other error

**Example**:
```bash
curl -X POST https://yourdomain.com/api/totp/setup \
  -b cookies.txt
```

**Usage**:
1. Receive the QR code in the response
2. Display it to the user
3. User scans QR code with authenticator app (Google Authenticator, Authy, etc.)
4. On next login, user will be prompted for TOTP code

---

#### Disable TOTP

Disable two-factor authentication for the current user.

**Endpoint**: `POST /api/totp/disable`

**Authentication**: Required

**Response** (200 OK):
```json
{
  "status": "ok",
  "message": "TOTP disabled"
}
```

**Example**:
```bash
curl -X POST https://yourdomain.com/api/totp/disable \
  -b cookies.txt
```

---

### Password Reset Endpoints

#### Forgot Password

Initiate password reset by sending a secure token via email.

**Endpoint**: `POST /api/forgot-password`

**Request Body**:
```json
{
  "email": "alice@example.com"
}
```

**Response** (200 OK):
```json
{
  "status": "ok",
  "message": "If this email is registered, a password reset token has been sent. Please check your email."
}
```

**Note**: The response message is generic regardless of whether the email exists. This prevents email enumeration attacks.

**Error Responses**:
- `500 Internal Server Error`: SMTP configuration missing or email sending failed
  ```json
  {
    "detail": "Failed to send reset email. Please check SMTP configuration or try again later."
  }
  ```

**Example**:
```bash
curl -X POST https://yourdomain.com/api/forgot-password \
  -H "Content-Type: application/json" \
  -d '{
    "email": "alice@example.com"
  }'
```

**Security Notes**:
- Reset tokens expire after 1 hour
- Tokens are only sent via email, never exposed in API responses
- Generic success message prevents email enumeration

---

#### Reset Password

Complete password reset using the token received via email.

**Endpoint**: `POST /api/reset-password`

**Request Body**:
```json
{
  "token": "reset_token_from_email",
  "new_password": "NewSecurePass123!"
}
```

**Response** (200 OK):
```json
{
  "status": "ok",
  "message": "Password reset successful"
}
```

**Error Responses**:
- `400 Bad Request`: Invalid token, expired token, or password doesn't meet requirements
  ```json
  {
    "detail": "Invalid or expired reset token"
  }
  ```

**Example**:
```bash
curl -X POST https://yourdomain.com/api/reset-password \
  -H "Content-Type: application/json" \
  -d '{
    "token": "reset_token_here",
    "new_password": "NewSecurePass123!"
  }'
```

**Important**: After password reset, the user's encryption keys are regenerated. Old messages cannot be decrypted.

---

### Messaging Endpoints

#### List Users

Get a list of all registered usernames.

**Endpoint**: `GET /api/users`

**Authentication**: Required

**Response** (200 OK):
```json
[
  "alice",
  "bob",
  "charlie"
]
```

**Example**:
```bash
curl -X GET https://yourdomain.com/api/users \
  -b cookies.txt
```

---

#### Get Inbox

Retrieve all messages for the authenticated user.

**Endpoint**: `GET /api/messages`

**Authentication**: Required

**Response** (200 OK):
```json
[
  {
    "from": "bob",
    "timestamp": "2025-01-15T10:30:00",
    "message": "Hello Alice!",
    "signature_valid": true
  },
  {
    "from": "charlie",
    "timestamp": "2025-01-15T09:15:00",
    "message": "Meeting at 3pm",
    "signature_valid": true
  }
]
```

**Response Fields**:
- `from`: Sender's username
- `timestamp`: ISO 8601 timestamp
- `message`: Decrypted message content
- `signature_valid`: Boolean indicating if Ed25519 signature is valid

**Example**:
```bash
curl -X GET https://yourdomain.com/api/messages \
  -b cookies.txt
```

**Note**: Messages are automatically decrypted using the user's private key. The server never sees plaintext messages.

---

#### Send Message

Send an encrypted message to a single recipient.

**Endpoint**: `POST /api/messages`

**Authentication**: Required

**Request Body**:
```json
{
  "recipient": "bob",
  "message": "Hello Bob, this is a secret message!"
}
```

**Response** (200 OK):
```json
{
  "sender": "alice",
  "recipient": "bob",
  "timestamp": "2025-01-15T10:30:00",
  "nonce": "base64_encoded_nonce",
  "ciphertext": "base64_encoded_ciphertext",
  "hmac": "base64_encoded_hmac",
  "ephemeral_pub": "base64_encoded_ephemeral_public_key",
  "signature": "base64_encoded_signature"
}
```

**Error Responses**:
- `400 Bad Request`: Recipient not found
  ```json
  {
    "detail": "Recipient not found"
  }
  ```

**Example**:
```bash
curl -X POST https://yourdomain.com/api/messages \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{
    "recipient": "bob",
    "message": "Hello Bob!"
  }'
```

**Cryptographic Details**:
- Message encrypted with AES-256-GCM
- Encryption key derived via X25519 ECDH
- Ephemeral key pair generated per message (forward secrecy)
- HMAC-SHA256 computed over nonce || ciphertext
- Ed25519 signature covers entire message envelope

---

#### Send Group Message

Send an encrypted message to multiple recipients.

**Endpoint**: `POST /api/group-messages`

**Authentication**: Required

**Request Body**:
```json
{
  "recipients": ["bob", "charlie", "david"],
  "message": "Hello everyone!"
}
```

**Response** (200 OK):
```json
{
  "status": "ok",
  "sent": 3
}
```

**Error Responses**:
- `400 Bad Request`: No valid recipients
  ```json
  {
    "detail": "No valid recipients"
  }
  ```

**Example**:
```bash
curl -X POST https://yourdomain.com/api/group-messages \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{
    "recipients": ["bob", "charlie"],
    "message": "Group message!"
  }'
```

**Note**: Each recipient receives an individually encrypted copy of the message. Invalid recipients are silently skipped.

---

### Health Check Endpoint

#### Health Check

Check if the API is running.

**Endpoint**: `GET /api/health`

**Authentication**: Not required

**Response** (200 OK):
```json
{
  "status": "ok"
}
```

**Example**:
```bash
curl -X GET https://yourdomain.com/api/health
```

---

## Authentication Flow Examples

### Complete Registration and Login Flow

```bash
# 1. Register
curl -X POST https://yourdomain.com/api/register \
  -H "Content-Type: application/json" \
  -d '{
    "username": "alice",
    "password": "SecurePass123!",
    "email": "alice@example.com"
  }'

# 2. Login (cookies saved to file)
curl -X POST https://yourdomain.com/api/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "alice",
    "password": "SecurePass123!"
  }' \
  -c cookies.txt

# 3. Use authenticated endpoints
curl -X GET https://yourdomain.com/api/messages \
  -b cookies.txt
```

### TOTP Setup Flow

```bash
# 1. Login
curl -X POST https://yourdomain.com/api/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "alice",
    "password": "SecurePass123!"
  }' \
  -c cookies.txt

# 2. Setup TOTP
curl -X POST https://yourdomain.com/api/totp/setup \
  -b cookies.txt

# Response contains QR code - user scans with authenticator app

# 3. Login with TOTP
curl -X POST https://yourdomain.com/api/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "alice",
    "password": "SecurePass123!",
    "totp_token": "123456"
  }' \
  -c cookies.txt
```

### Password Reset Flow

```bash
# 1. Request reset
curl -X POST https://yourdomain.com/api/forgot-password \
  -H "Content-Type: application/json" \
  -d '{
    "email": "alice@example.com"
  }'

# 2. User receives email with token

# 3. Reset password
curl -X POST https://yourdomain.com/api/reset-password \
  -H "Content-Type: application/json" \
  -d '{
    "token": "token_from_email",
    "new_password": "NewSecurePass123!"
  }'
```

## Error Handling

### Common Error Scenarios

#### Invalid Credentials
```json
{
  "detail": "Invalid credentials"
}
```
**HTTP Status**: 401 Unauthorized

#### Missing Authentication
```json
{
  "detail": "Missing authentication"
}
```
**HTTP Status**: 401 Unauthorized

#### Invalid Input
```json
{
  "detail": "Password must be at least 8 characters long"
}
```
**HTTP Status**: 400 Bad Request

#### Resource Not Found
```json
{
  "detail": "Recipient not found"
}
```
**HTTP Status**: 400 Bad Request

## Rate Limiting

**Note**: Rate limiting is not currently implemented but is recommended for production. Future versions may include:

- 5 login attempts per minute per IP
- 3 password reset requests per hour per email
- 100 API requests per minute per session

## CORS Configuration

The API includes CORS middleware. In development, all origins are allowed. In production, configure allowed origins:

```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://yourdomain.com"],
    allow_methods=["GET", "POST"],
    allow_headers=["*"],
)
```

## WebSocket Support

**Note**: WebSocket support is not currently implemented. Future versions may include real-time messaging via WebSockets.

## API Versioning

Current API version: **v1.0**

Future versions may include versioning via URL path (`/api/v1/...`) or headers.

## SDKs and Client Libraries

Official SDKs are not currently available. The API follows RESTful principles and can be consumed by any HTTP client.

### Python Example

```python
import requests

BASE_URL = "https://yourdomain.com/api"
session = requests.Session()

# Register
session.post(f"{BASE_URL}/register", json={
    "username": "alice",
    "password": "SecurePass123!",
    "email": "alice@example.com"
})

# Login
session.post(f"{BASE_URL}/login", json={
    "username": "alice",
    "password": "SecurePass123!"
})

# Send message
session.post(f"{BASE_URL}/messages", json={
    "recipient": "bob",
    "message": "Hello!"
})

# Get inbox
response = session.get(f"{BASE_URL}/messages")
messages = response.json()
```

### JavaScript Example

```javascript
const API_BASE = '/api';

async function login(username, password) {
  const response = await fetch(`${API_BASE}/login`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    credentials: 'include',  // Important for cookies
    body: JSON.stringify({ username, password })
  });
  return response.json();
}

async function sendMessage(recipient, message) {
  const response = await fetch(`${API_BASE}/messages`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    credentials: 'include',
    body: JSON.stringify({ recipient, message })
  });
  return response.json();
}

async function getInbox() {
  const response = await fetch(`${API_BASE}/messages`, {
    credentials: 'include'
  });
  return response.json();
}
```

## Changelog

### Version 1.0
- Initial API release
- Cookie-based authentication
- End-to-end encrypted messaging
- TOTP support
- Password reset flow

## Support

For API support, please contact your administrator or refer to the project documentation.
