# Password Management Guide

## Overview

The system uses two distinct password management flows:

1. **Forgot Password** → Email-based OTP (no authentication required)
2. **Change Password** → Token-based (requires authentication)

---

## 1. Forgot Password (OTP-Based, No Auth Required)

### Flow

User forgets password → Receives OTP via email → Verifies OTP + enters new password

### Step 1: Send OTP to Email

```bash
POST /v1/otp/password-reset/send
Content-Type: application/json

{
  "email": "user@example.com"
}
```

**Response:**
```json
{
  "message": "OTP sent successfully",
  "email": "user@example.com",
  "expiresAt": "2025-10-27T12:01:00.000Z"
}
```

**Note:** OTP expires in **1 minute** ⏱️

### Step 2: Verify OTP and Reset Password

```bash
POST /v1/otp/password-reset/verify
Content-Type: application/json

{
  "email": "user@example.com",
  "otpCode": "1234",
  "password": "NewPass@123"
}
```

**Response:**
```json
{
  "message": "Password reset successful"
}
```

### Step 3: Resend OTP (if needed)

```bash
POST /v1/otp/resend
Content-Type: application/json

{
  "email": "user@example.com",
  "type": "password_reset"
}
```

**Rate Limited:** 1 request per minute

---

## 2. Change Password (Token-Based, Auth Required)

### Flow

Authenticated user → Provides old password → Sets new password

### Endpoint

```bash
POST /v1/auth/change-password
Authorization: Bearer <access-token>
Content-Type: application/json

{
  "oldPassword": "OldPass@123",
  "newPassword": "NewPass@123"
}
```

**Response:**
```json
{
  "message": "Password changed successfully"
}
```

### Error Responses

**Incorrect Old Password:**
```json
{
  "code": 401,
  "message": "Incorrect old password"
}
```

**Unauthorized (No Token):**
```json
{
  "code": 401,
  "message": "Please authenticate"
}
```

---

## Comparison

| Feature | Forgot Password | Change Password |
|---------|-----------------|-----------------|
| **Authentication** | Not required | Required (Bearer token) |
| **Verification Method** | OTP to email | Old password |
| **Use Case** | User forgot their password | User wants to update password |
| **Expiration** | OTP expires in 1 minute | Token-based (standard expiration) |
| **Endpoint** | `/v1/otp/password-reset/*` | `/v1/auth/change-password` |
| **Steps** | 1. Send OTP<br>2. Verify OTP + Reset | 1. Provide old password<br>2. Set new password |

---

## OTP Features

### Email Verification OTP
- **4-digit code**
- **1-minute expiration** ⏱️
- **3 attempts maximum**
- **Rate limited** (1 per minute)

### Password Reset OTP
- **4-digit code**
- **1-minute expiration** ⏱️
- **3 attempts maximum**
- **Rate limited** (1 per minute)

---

## Frontend Implementation

### Forgot Password Flow

```javascript
// Step 1: User enters email
async function requestPasswordReset(email) {
  const response = await fetch('/v1/otp/password-reset/send', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email })
  });
  
  const result = await response.json();
  // Show: "OTP sent to your email (expires in 1 minute)"
  return result;
}

// Step 2: User enters OTP + new password
async function resetPassword(email, otpCode, newPassword) {
  const response = await fetch('/v1/otp/password-reset/verify', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email, otpCode, password: newPassword })
  });
  
  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.message);
  }
  
  return await response.json();
  // Show: "Password reset successful. Please login."
}

// Step 3: Resend OTP if expired
async function resendPasswordResetOTP(email) {
  const response = await fetch('/v1/otp/resend', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email, type: 'password_reset' })
  });
  
  return await response.json();
}
```

### Change Password Flow

```javascript
// Authenticated user changes password
async function changePassword(oldPassword, newPassword) {
  const token = localStorage.getItem('accessToken');
  
  const response = await fetch('/v1/auth/change-password', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${token}`
    },
    body: JSON.stringify({ oldPassword, newPassword })
  });
  
  if (!response.ok) {
    const error = await response.json();
    if (error.code === 401) {
      throw new Error('Incorrect old password');
    }
    throw new Error(error.message);
  }
  
  return await response.json();
  // Show: "Password changed successfully"
}
```

---

## Security Considerations

### Forgot Password (OTP)
1. **No authentication required** - Anyone can request OTP for any email
2. **Rate limiting** - Prevents spam (1 per minute)
3. **Short expiration** - 1-minute window reduces attack surface
4. **3 attempts only** - Prevents brute force
5. **Email-based** - Requires access to user's email account

### Change Password (Token)
1. **Authentication required** - Must be logged in
2. **Old password verification** - Proves user identity
3. **Token-based** - Uses JWT for authentication
4. **No rate limiting needed** - Already authenticated

---

## Error Handling

### OTP Errors

**Invalid OTP:**
```json
{
  "code": 400,
  "message": "Invalid OTP code. 2 attempt(s) remaining"
}
```

**Expired OTP:**
```json
{
  "code": 400,
  "message": "OTP has expired. Please request a new OTP"
}
```

**Max Attempts:**
```json
{
  "code": 400,
  "message": "Maximum OTP attempts exceeded. Please request a new OTP"
}
```

**Rate Limit:**
```json
{
  "code": 429,
  "message": "Please wait 45 seconds before requesting a new OTP"
}
```

### Change Password Errors

**Incorrect Old Password:**
```json
{
  "code": 401,
  "message": "Incorrect old password"
}
```

**Unauthorized:**
```json
{
  "code": 401,
  "message": "Please authenticate"
}
```

**User Not Found:**
```json
{
  "code": 404,
  "message": "User not found"
}
```

---

## Testing

### Test Forgot Password

```bash
# 1. Request OTP
curl -X POST http://localhost:2323/v1/otp/password-reset/send \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com"}'

# Check email for 4-digit OTP

# 2. Reset password with OTP
curl -X POST http://localhost:2323/v1/otp/password-reset/verify \
  -H "Content-Type: application/json" \
  -d '{
    "email":"user@example.com",
    "otpCode":"1234",
    "password":"NewPassword@123"
  }'

# 3. Login with new password
curl -X POST http://localhost:2323/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"NewPassword@123"}'
```

### Test Change Password

```bash
# 1. Login first
curl -X POST http://localhost:2323/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"OldPassword@123"}'

# Save accessToken from response

# 2. Change password
curl -X POST http://localhost:2323/v1/auth/change-password \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <ACCESS_TOKEN>" \
  -d '{
    "oldPassword":"OldPassword@123",
    "newPassword":"NewPassword@123"
  }'

# 3. Verify with new password
curl -X POST http://localhost:2323/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"NewPassword@123"}'
```

---

## Summary

### ✅ Forgot Password (OTP-Based)
- **No authentication required**
- **Email-based** verification
- **1-minute expiration**
- **3 attempts maximum**
- **Rate limited**
- Use when user has no access to their account

### ✅ Change Password (Token-Based)
- **Authentication required**
- **Old password** verification
- **Token-based** (JWT)
- **Immediate effect**
- Use when user is logged in and knows their current password

Both flows ensure security while providing appropriate access methods for different scenarios! 🔐

