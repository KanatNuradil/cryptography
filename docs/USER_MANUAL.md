# Secure Messenger - User Manual

## Table of Contents

1. [Introduction](#introduction)
2. [Getting Started](#getting-started)
3. [Account Management](#account-management)
4. [Sending Messages](#sending-messages)
5. [Receiving Messages](#receiving-messages)
6. [Two-Factor Authentication (2FA)](#two-factor-authentication-2fa)
7. [Password Reset](#password-reset)
8. [Troubleshooting](#troubleshooting)
9. [Security Tips](#security-tips)

## Introduction

Welcome to Secure Messenger! This application provides end-to-end encrypted messaging, ensuring that your conversations remain private and secure. Messages are encrypted on your device before being sent and can only be decrypted by the intended recipient.

### Key Features

- **End-to-End Encryption**: Messages are encrypted using industry-standard cryptography
- **Multi-Factor Authentication**: Optional two-factor authentication for enhanced security
- **Secure Password Reset**: Token-based password recovery via email
- **Group Messaging**: Send encrypted messages to multiple recipients
- **Digital Signatures**: Verify message authenticity and integrity

## Getting Started

### Accessing the Application

1. Open your web browser
2. Navigate to the Secure Messenger URL (provided by your administrator)
3. You will see the home page with options to register or login

### System Requirements

- Modern web browser (Chrome, Firefox, Safari, Edge)
- JavaScript enabled
- Internet connection
- Email address (for password reset)

## Account Management

### Creating an Account

1. Click **"Get Started"** or **"Sign Up"** on the home page
2. Fill in the registration form:
   - **Username**: Choose a unique username (letters, numbers, underscores only)
   - **Password**: Create a strong password (see Password Requirements below)
   - **Email** (optional): Provide an email address for password recovery
3. Click **"Register"**

#### Password Requirements

Your password must meet the following requirements:
- **Minimum 8 characters**
- **At least one letter** (a-z or A-Z)
- **At least one number** (0-9)
- **At least one special character** (!@#$%^&*()_+-=[]{}|;:,.<>?)

**Examples of valid passwords:**
- `SecurePass123!`
- `MyP@ssw0rd`
- `Str0ng#Key`

**Examples of invalid passwords:**
- `password` (no number or special character)
- `12345678` (no letters or special characters)
- `Secure` (too short, no number or special character)

### Logging In

1. Click **"Login"** on the home page
2. Enter your username and password
3. If you have Two-Factor Authentication enabled:
   - Enter the 6-digit code from your authenticator app
   - Click **"Verify"**
4. Click **"Login"**

You will be redirected to the main messaging interface upon successful login.

### Logging Out

1. Click the **"Logout"** button in the top-right corner
2. You will be logged out and redirected to the home page

**Security Note**: Always log out when using a shared or public computer.

## Sending Messages

### Sending a Message to One Recipient

1. Ensure you are logged in
2. In the **"Send Message"** section:
   - Select a recipient from the dropdown menu (or type username)
   - Type your message in the message box
   - Click **"Send"**
3. You will see a confirmation message when the message is sent successfully

### Sending a Group Message

1. In the **"Send Group Message"** section:
   - Enter recipient usernames separated by commas (e.g., `alice, bob, charlie`)
   - Type your message
   - Click **"Send to Group"**
2. The message will be encrypted separately for each recipient

**Note**: Each recipient receives an individually encrypted copy of your message. The encryption ensures that even if one recipient's account is compromised, other recipients' messages remain secure.

### Message Limitations

- Maximum message length: 10,000 characters (recommended)
- Messages are encrypted before sending
- Messages cannot be edited or deleted after sending

## Receiving Messages

### Viewing Your Inbox

1. After logging in, your inbox is displayed automatically
2. Messages are shown in reverse chronological order (newest first)
3. Each message displays:
   - **From**: Sender's username
   - **Timestamp**: When the message was sent
   - **Message**: Decrypted message content
   - **Signature Status**: Whether the message signature is valid

### Understanding Signature Status

- **✓ Valid Signature**: Message is authentic and hasn't been tampered with
- **✗ Invalid Signature**: Message may have been modified or is from an untrusted source

**Note**: If you see "decryption failed", the message may be corrupted or you may not have the correct decryption key.

### Message Security

- Messages are automatically decrypted when you view your inbox
- Decryption happens on your device - the server never sees plaintext messages
- Each message uses unique encryption keys for forward secrecy

## Two-Factor Authentication (2FA)

Two-Factor Authentication adds an extra layer of security to your account by requiring both your password and a time-based code from an authenticator app.

### Setting Up 2FA

1. Log in to your account
2. In the **"Two-Factor Authentication"** section, click **"Set up Two-Factor Authentication"**
3. A QR code will appear on your screen
4. Open your authenticator app on your mobile device:
   - **Google Authenticator** (iOS/Android)
   - **Authy** (iOS/Android)
   - **Microsoft Authenticator** (iOS/Android)
   - Any TOTP-compatible app
5. Scan the QR code with your authenticator app
6. The app will generate 6-digit codes that change every 30 seconds
7. Click **"Done"** to complete setup

**Important**: Save your backup codes or keep access to your authenticator app. If you lose access, you may need to contact support.

### Using 2FA

1. When logging in, enter your username and password
2. You will be prompted for a 6-digit code
3. Open your authenticator app
4. Enter the current 6-digit code
5. Click **"Verify"** to complete login

### Disabling 2FA

1. Log in to your account
2. In the **"Two-Factor Authentication"** section, click **"Disable Two-Factor Authentication"**
3. Confirm that you want to disable 2FA

**Security Warning**: Disabling 2FA reduces your account security. Only disable if necessary.

## Password Reset

If you forget your password, you can reset it using the secure password reset flow.

### Requesting a Password Reset

1. On the login page, click **"Forgot Password?"**
2. Enter the email address associated with your account
3. Click **"Send Reset Token"**
4. Check your email for a password reset message

**Note**: You will receive a generic success message regardless of whether your email is registered. This prevents email enumeration attacks.

### Completing Password Reset

1. Open the password reset email
2. Click the reset link (or copy the reset token)
3. You will be redirected to the password reset page
4. Enter the reset token (if not in the URL)
5. Enter your new password (must meet password requirements)
6. Confirm your new password
7. Click **"Reset Password"**

**Important Security Notes**:
- Reset tokens expire after 1 hour
- Each token can only be used once
- After password reset, your old messages cannot be decrypted (by design)
- Your encryption keys are regenerated for security

### If You Don't Receive the Email

1. Check your spam/junk folder
2. Verify you entered the correct email address
3. Wait a few minutes and try again
4. Contact your administrator if the problem persists

**Note**: Email delivery requires SMTP configuration. In development environments, email may not be configured.

## Troubleshooting

### Cannot Log In

**Problem**: "Invalid credentials" error

**Solutions**:
- Verify your username and password are correct
- Check for typos (passwords are case-sensitive)
- If 2FA is enabled, ensure you're entering the current 6-digit code
- Try resetting your password

### TOTP Code Not Working

**Problem**: "Invalid TOTP token" error

**Solutions**:
- Ensure your device clock is synchronized (TOTP is time-based)
- Make sure you're using the correct authenticator app
- Wait for a new code to generate (codes change every 30 seconds)
- Verify you scanned the correct QR code during setup

### Messages Not Appearing

**Problem**: Inbox is empty or messages don't decrypt

**Solutions**:
- Refresh the page
- Check your internet connection
- Verify you're logged in as the correct user
- If you see "decryption failed", the message may be corrupted or from an old account

### Password Reset Not Working

**Problem**: Reset token invalid or expired

**Solutions**:
- Request a new reset token (tokens expire after 1 hour)
- Ensure you're using the most recent reset token
- Check that you copied the entire token correctly
- Contact support if the problem persists

### Browser Issues

**Problem**: Application doesn't load or features don't work

**Solutions**:
- Clear your browser cache and cookies
- Ensure JavaScript is enabled
- Try a different browser
- Check that you're using a modern browser version
- Disable browser extensions that might interfere

## Security Tips

### Password Security

1. **Use a Strong Password**: Follow the password requirements and create a unique password
2. **Don't Reuse Passwords**: Use a different password for Secure Messenger than other services
3. **Use a Password Manager**: Consider using a password manager to generate and store strong passwords
4. **Never Share Your Password**: Your password is your responsibility - never share it with anyone

### Account Security

1. **Enable Two-Factor Authentication**: This significantly improves your account security
2. **Log Out on Shared Devices**: Always log out when using public or shared computers
3. **Monitor Your Account**: Regularly check your inbox for suspicious activity
4. **Keep Your Email Secure**: Your email is used for password reset - keep it secure

### Message Security

1. **Verify Recipients**: Double-check recipient usernames before sending sensitive messages
2. **Check Signature Status**: Pay attention to signature validity indicators
3. **Don't Share Screenshots**: Avoid sharing screenshots of encrypted messages
4. **Be Cautious with Group Messages**: Remember that group messages are sent to multiple recipients

### General Security

1. **Keep Your Browser Updated**: Use the latest version of your web browser
2. **Use HTTPS**: Ensure you're accessing the application over HTTPS (look for the padlock icon)
3. **Be Wary of Phishing**: Only access Secure Messenger through the official URL
4. **Report Suspicious Activity**: If you notice anything suspicious, report it to your administrator

## Command-Line Interface (CLI)

For advanced users, Secure Messenger also provides a command-line interface.

### Starting the CLI

```bash
python -m secure_messaging
```

### Available Commands

- `register` - Create a new account
- `login` - Log in to your account
- `send` - Send a message to a user
- `inbox` - View your received messages
- `users` - List all registered users
- `logout` - Log out of your account
- `quit` or `exit` - Exit the application
- `help` - Show available commands

### CLI Example Session

```
Secure Messaging CLI
Type 'help' for options.
[guest] > register
Choose username: alice
Choose password: ********
Confirm password: ********
User registered.
[guest] > login
Username: alice
Password: ********
Welcome alice!
[alice] > users
Registered users:
 - alice
 - bob
[alice] > send
Recipient username: bob
Message: Hello, this is a test message!
Message sent.
[alice] > inbox
[1] From bob @ 2025-01-15T10:30:00 (valid signature)
    Hi Alice, how are you?
[alice] > logout
[guest] > quit
Goodbye!
```

## Frequently Asked Questions (FAQ)

### Q: Can I recover my messages if I forget my password?

**A**: No. For security reasons, password reset regenerates your encryption keys. Old messages cannot be decrypted after a password reset. This is by design to ensure forward secrecy.

### Q: Can the server read my messages?

**A**: No. Messages are encrypted on your device before being sent. The server only stores encrypted ciphertext and cannot decrypt it without your private key.

### Q: What happens if I lose access to my authenticator app?

**A**: Contact your administrator for assistance. You may need to disable 2FA through an alternative method or create a new account.

### Q: Can I delete messages?

**A**: Currently, messages cannot be deleted. This feature may be added in future versions.

### Q: Is there a mobile app?

**A**: The application is web-based and works on mobile browsers. A dedicated mobile app may be available in the future.

### Q: How do I report a security issue?

**A**: Contact your administrator or the security team through the appropriate channel. Do not report security issues through public channels.

## Getting Help

If you need assistance:

1. Check this user manual
2. Review the troubleshooting section
3. Contact your system administrator
4. Check for application updates

## Version Information

This manual applies to Secure Messenger version 1.0.

For the latest updates and documentation, please refer to the project repository or contact your administrator.
