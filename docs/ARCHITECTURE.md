# Secure Messenger - Architecture Document

## Overview

The Secure Messenger is an end-to-end encrypted messaging application designed with security as a primary concern. It provides secure communication between users through cryptographic primitives, multi-factor authentication, and secure session management.

## System Architecture

### High-Level Architecture

The application follows a **client-server architecture** with three main components:

1. **Backend Server** (FastAPI)
2. **Frontend Client** (Single Page Application)
3. **Database** (SQLite)

```
┌─────────────────┐
│  Web Browser    │
│  (Frontend SPA) │
└────────┬────────┘
         │ HTTPS
         │
┌────────▼────────┐
│  FastAPI Server │
│  (Python)       │
└────────┬────────┘
         │
    ┌────┴────┐
    │         │
┌───▼───┐ ┌──▼────┐
│ SQLite│ │ SMTP  │
│  DB   │ │Server │
└───────┘ └───────┘
```

### Component Architecture

#### 1. Backend Components

**`server.py`** - FastAPI Application
- HTTP API endpoints for authentication, messaging, and TOTP management
- Session management using HttpOnly cookies
- CORS middleware for cross-origin requests
- Static file serving for frontend

**`app.py`** - Business Logic Layer
- `AuthService`: User registration, login, password reset, TOTP management
- `MessagingService`: Message encryption, decryption, and delivery
- `SecureMessagingCLI`: Command-line interface (alternative to web UI)

**`crypto.py`** - Cryptographic Primitives
- Key generation (X25519, Ed25519)
- Password hashing (bcrypt)
- Key derivation (PBKDF2, HKDF)
- Message encryption/decryption (AES-256-GCM)
- Digital signatures (Ed25519)
- HMAC-SHA256 for message integrity

**`db.py`** - Data Persistence Layer
- SQLite database operations
- Schema migrations
- User and message storage

**`validation.py`** - Input Validation
- Password complexity validation
- Requirements enforcement

**`totp.py`** - Multi-Factor Authentication
- TOTP secret generation
- QR code generation for authenticator apps
- TOTP verification

**`emailer.py`** - Email Services
- SMTP email sending for password reset
- Secure token delivery

#### 2. Frontend Components

**`index.html`** - Main Application UI
- Single Page Application structure
- User registration and login forms
- Message composition and inbox
- TOTP setup interface

**`app.js`** - Frontend Logic
- API communication
- Cookie-based authentication
- Password validation
- UI state management

**`styles.css`** - User Interface Styling
- Modern, responsive design
- User-friendly error messages

**`reset_password.html`** - Password Reset Page
- Token-based password reset flow

#### 3. Database Schema

**Users Table**
- `id`: Primary key
- `username`: Unique identifier
- `password_hash`: Bcrypt hash
- `wrapped_keys`: Encrypted private keys (AES-GCM)
- `public_payload`: Public keys (X25519, Ed25519)
- `email`: Email address (optional)
- `totp_secret`: TOTP secret for MFA
- `reset_token`: Password reset token
- `reset_token_expires_at`: Token expiration timestamp
- `created_at`: Account creation timestamp

**Messages Table**
- `id`: Primary key
- `sender`: Username of sender
- `recipient`: Username of recipient
- `timestamp`: Message timestamp
- `nonce`: AES-GCM nonce (base64)
- `ciphertext`: Encrypted message (base64)
- `hmac`: HMAC-SHA256 tag (base64)
- `ephemeral_pub`: Ephemeral public key (base64)
- `signature`: Ed25519 signature (base64)

## Data Flow

### User Registration Flow

```
1. User submits registration form
   ↓
2. Frontend validates password complexity
   ↓
3. POST /api/register
   ↓
4. AuthService.register()
   - Validates password complexity
   - Generates X25519 and Ed25519 key pairs
   - Hashes password with bcrypt
   - Wraps private keys with PBKDF2-derived key
   - Stores user in database
   ↓
5. Returns success response
```

### Message Sending Flow

```
1. User composes message
   ↓
2. POST /api/messages
   ↓
3. MessagingService.send_message()
   - Retrieves recipient's public key
   - Generates ephemeral X25519 key pair
   - Derives shared secret via ECDH
   - Expands to AES key + HMAC key via HKDF
   - Encrypts message with AES-256-GCM
   - Computes HMAC over nonce || ciphertext
   - Signs envelope with Ed25519
   - Stores encrypted message in database
   ↓
4. Returns encrypted envelope
```

### Message Receiving Flow

```
1. User requests inbox
   ↓
2. GET /api/messages
   ↓
3. MessagingService.inbox()
   - Retrieves encrypted messages from database
   - For each message:
     a. Verifies Ed25519 signature
     b. Derives decryption key via ECDH
     c. Verifies HMAC
     d. Decrypts with AES-256-GCM
   ↓
4. Returns decrypted messages
```

### Authentication Flow

```
1. User submits login credentials
   ↓
2. POST /api/login
   ↓
3. AuthService.login()
   - Verifies password hash
   - Checks if TOTP is enabled
   - If TOTP enabled and not provided: return requires_totp=true
   - If TOTP provided: verify TOTP token
   - Unwraps private keys with password
   - Creates ActiveSession
   ↓
4. Server creates session token
   - Sets HttpOnly cookie
   - Returns session info
```

## Cryptographic Design

### Key Exchange Protocol

The system uses **X25519 ECDH** (Elliptic Curve Diffie-Hellman) for key exchange:

1. Each user has a long-term X25519 key pair
2. For each message, sender generates a fresh ephemeral X25519 key pair
3. Sender performs ECDH with recipient's public key
4. Shared secret is expanded via **HKDF-SHA256** into:
   - 32-byte AES encryption key
   - 32-byte HMAC key

**Benefits:**
- **Forward Secrecy**: Each message uses a unique ephemeral key
- **Perfect Forward Secrecy**: Compromising long-term keys doesn't reveal past messages

### Encryption Scheme

**AES-256-GCM** (Galois/Counter Mode) is used for message encryption:

- **Confidentiality**: AES-256 encryption
- **Integrity**: GCM authentication tag
- **Nonce**: 12-byte random nonce per message

**Additional HMAC Layer:**
- HMAC-SHA256 computed over `nonce || ciphertext`
- Provides defense-in-depth integrity verification
- Uses separate HMAC key derived from ECDH

### Digital Signatures

**Ed25519** signatures provide:
- **Authentication**: Verifies message sender
- **Non-repudiation**: Sender cannot deny sending
- **Integrity**: Detects message tampering

Signature covers entire message envelope (metadata + ciphertext).

### Password Security

**Password Storage:**
- Passwords hashed with **bcrypt** (configurable cost factor)
- Never stored in plaintext

**Private Key Protection:**
- Private keys encrypted at rest using **AES-256-GCM**
- Encryption key derived via **PBKDF2-SHA256** (200,000 iterations)
- Salt: 16-byte random per user
- Nonce: 12-byte random per encryption

## Security Features

### Authentication & Authorization

1. **Password Complexity Validation**
   - Minimum 8 characters
   - Requires letter, number, and special character

2. **Multi-Factor Authentication (MFA)**
   - TOTP-based 2FA using authenticator apps
   - Optional but recommended

3. **Session Management**
   - HttpOnly cookies prevent XSS attacks
   - Secure flag (in production) ensures HTTPS-only
   - SameSite=Lax prevents CSRF attacks
   - 24-hour session expiration

### Password Reset

1. **Secure Token Generation**
   - 32-byte URL-safe random token
   - 1-hour expiration
   - Stored in database with expiration timestamp

2. **Email Delivery**
   - SMTP with STARTTLS encryption
   - Token never exposed in API responses
   - Generic success message prevents email enumeration

3. **Key Regeneration**
   - Password reset regenerates user key pairs
   - Old messages become undecryptable (by design)

## Deployment Architecture

### Development Mode

- SQLite database (file-based)
- HTTP (no TLS)
- In-memory session storage
- CORS allows all origins

### Production Recommendations

1. **Database**: Migrate to PostgreSQL or MySQL
2. **HTTPS**: Enable TLS/SSL certificates
3. **Session Storage**: Use Redis or database-backed sessions
4. **CORS**: Restrict allowed origins
5. **Environment Variables**: Store secrets securely
6. **Reverse Proxy**: Use Nginx or similar
7. **Monitoring**: Add logging and error tracking

## Scalability Considerations

### Current Limitations

- In-memory session storage (doesn't scale horizontally)
- SQLite database (single-writer limitation)
- No message queuing system
- Synchronous email sending

### Scaling Strategies

1. **Horizontal Scaling**
   - Move sessions to Redis
   - Use PostgreSQL with connection pooling
   - Add message queue (RabbitMQ, Redis Queue)

2. **Performance Optimization**
   - Database indexing on frequently queried fields
   - Caching of public keys
   - Asynchronous email sending

3. **Load Balancing**
   - Multiple FastAPI instances behind load balancer
   - Session affinity or shared session store

## Technology Stack

- **Backend**: Python 3.9+, FastAPI, Uvicorn
- **Frontend**: Vanilla JavaScript (no framework dependencies)
- **Database**: SQLite (development), PostgreSQL recommended (production)
- **Cryptography**: `cryptography` library (X25519, Ed25519, AES-GCM)
- **Password Hashing**: `bcrypt`
- **TOTP**: `pyotp`
- **QR Codes**: `qrcode[pil]`
- **Email**: `smtplib` (Python standard library)

## Design Principles

1. **Security First**: All cryptographic operations use industry-standard algorithms
2. **Defense in Depth**: Multiple layers of security (encryption + signatures + HMAC)
3. **Forward Secrecy**: Ephemeral keys for each message
4. **Zero-Knowledge**: Server never sees plaintext messages
5. **Fail-Safe Defaults**: Secure by default, requires explicit insecure configuration
6. **Principle of Least Privilege**: Users can only access their own messages

## Future Enhancements

- Group messaging with forward secrecy
- Message deletion/expiration
- File attachments with encryption
- Offline message queue
- Push notifications
- End-to-end encrypted voice/video calls
- Key rotation mechanisms
- Message search (encrypted search indices)
