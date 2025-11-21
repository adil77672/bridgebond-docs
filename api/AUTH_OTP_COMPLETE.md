# Auth & OTP System - Complete Documentation ✅

## 🎯 Overview

The authentication and OTP system is **fully implemented and documented** with Swagger documentation that exactly matches the code implementation.

---

## 📍 Current Routes (Exact Match)

### **Authentication Routes** (`/v1/auth/*`)

| # | Method | Endpoint | Auth | Swagger | Description |
|---|--------|----------|------|---------|-------------|
| 1 | POST | `/auth/register` | No | ✅ | Register user (auto-sends OTP) |
| 2 | POST | `/auth/login` | No | ✅ | Login (returns orgSelectionToken) |
| 3 | POST | `/auth/select-organization` | No | ✅ | Select organization (returns tokens) |
| 4 | POST | `/auth/logout` | No | ✅ | Logout |
| 5 | POST | `/auth/refresh-tokens` | No | ✅ | Refresh access token |
| 6 | POST | `/auth/change-password` | Yes | ✅ | Change password (authenticated) |

**File:** `/Users/macbookpro/projects/bridge-bond/src/routes/v1/auth.route.js`

---

### **OTP Routes** (`/v1/otp/*`)

| # | Method | Endpoint | Auth | Swagger | Description |
|---|--------|----------|------|---------|-------------|
| 1 | POST | `/otp/send` | No | ✅ | Send OTP (type-based) |
| 2 | POST | `/otp/verify` | No | ✅ | Verify OTP (type-based) |
| 3 | POST | `/otp/resend` | No | ✅ | Resend OTP (semantic endpoint) |

**File:** `/Users/macbookpro/projects/bridge-bond/src/routes/v1/otp.route.js`

---

## 🔄 Complete Authentication Flow

### **1. Registration Flow**

```bash
# Step 1: Register
POST /v1/auth/register
{
  "email": "john@example.com",
  "password": "Pass@123",
  "firstName": "John",
  "lastName": "Doe",
  "dob": "1990-01-15",
  "dateOfHire": "2020-03-01",
  "department": "5ebac534954b54139806c114",
  "jobTitle": "Software Engineer",
  "organizationId": "5ebac534954b54139806c113"
}

# Response:
{
  "user": { "id": "...", "email": "john@example.com" },
  "message": "Registration successful. Please check your email for verification OTP.",
  "otp": {
    "expiresAt": "2025-10-29T12:01:00.000Z"
  }
}

# Step 2: Verify Email (OTP sent automatically)
POST /v1/otp/verify
{
  "email": "john@example.com",
  "code": "1234",
  "type": "email_verification"
}

# Response:
{
  "message": "Email verified successfully"
}
```

---

### **2. Login Flow (Two-Step)**

```bash
# Step 1: Login
POST /v1/auth/login
{
  "email": "john@example.com",
  "password": "Pass@123"
}

# Response:
{
  "orgSelectionToken": "eyJhbGciOiJIUzI1NiI...",
  "organizations": [
    { "id": "5ebac534954b54139806c113", "name": "TechCorp" },
    { "id": "5ebac534954b54139806c114", "name": "StartupInc" }
  ]
}

# Step 2: Select Organization
POST /v1/auth/select-organization
{
  "orgSelectionToken": "eyJhbGciOiJIUzI1NiI...",
  "organizationId": "5ebac534954b54139806c113"
}

# Response:
{
  "tokens": {
    "access": {
      "token": "eyJhbGciOiJIUzI1NiI...",
      "expires": "2025-10-29T13:00:00.000Z"
    },
    "refresh": {
      "token": "eyJhbGciOiJIUzI1NiI...",
      "expires": "2025-11-28T12:00:00.000Z"
    }
  },
  "user": {
    "id": "...",
    "email": "john@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "user",
    "currentOrganization": {
      "id": "5ebac534954b54139806c113",
      "name": "TechCorp"
    }
  }
}
```

---

### **3. Forgot Password Flow**

```bash
# Step 1: Request OTP
POST /v1/otp/send
{
  "email": "john@example.com",
  "type": "password_reset"
}

# Response:
{
  "message": "OTP sent successfully",
  "email": "john@example.com",
  "type": "password_reset",
  "expiresAt": "2025-10-29T12:01:00.000Z"
}

# Step 2: Reset Password with OTP
POST /v1/otp/verify
{
  "email": "john@example.com",
  "code": "1234",
  "type": "password_reset",
  "password": "NewPass@456"
}

# Response:
{
  "message": "Password reset successful"
}
```

---

### **4. Change Password Flow (Authenticated)**

```bash
POST /v1/auth/change-password
Headers: {
  "Authorization": "Bearer eyJhbGciOiJIUzI1NiI..."
}
Body: {
  "oldPassword": "Pass@123",
  "newPassword": "NewPass@456"
}

# Response:
{
  "message": "Password changed successfully"
}
```

---

### **5. Resend OTP**

```bash
POST /v1/otp/resend
{
  "email": "john@example.com",
  "type": "email_verification"  # or "password_reset"
}

# Response:
{
  "message": "OTP sent successfully",
  "email": "john@example.com",
  "type": "email_verification",
  "expiresAt": "2025-10-29T12:01:00.000Z"
}
```

---

### **6. Refresh Tokens**

```bash
POST /v1/auth/refresh-tokens
{
  "refreshToken": "eyJhbGciOiJIUzI1NiI..."
}

# Response:
{
  "access": {
    "token": "eyJhbGciOiJIUzI1NiI...",
    "expires": "2025-10-29T13:00:00.000Z"
  },
  "refresh": {
    "token": "eyJhbGciOiJIUzI1NiI...",
    "expires": "2025-11-28T12:00:00.000Z"
  }
}
```

---

### **7. Logout**

```bash
POST /v1/auth/logout

# Response:
{
  "message": "Logout successful"
}

# Note: Tokens are removed client-side (JWT-based, not stored in DB)
```

---

## 🔑 Key Features

### **1. Unified OTP System**
- Single `/otp/send` endpoint handles both email verification and password reset
- Single `/otp/verify` endpoint verifies both types
- `type` parameter determines operation: `email_verification` or `password_reset`

### **2. Auto-Send on Registration**
- OTP automatically sent when user registers
- No need to call `/otp/send` separately for email verification

### **3. Two-Step Authentication**
- Step 1: Login → Get `orgSelectionToken` + list of user's organizations
- Step 2: Select org → Get `access` & `refresh` tokens with organization context

### **4. Stateless JWT Tokens**
- Tokens not stored in database
- All user context (including `organizationId`) embedded in token
- Verified cryptographically on each request

### **5. Organization-Based Filtering**
- Every authenticated request automatically filtered by `organizationId` from token
- Users can belong to multiple organizations with different profiles per org

### **6. Email Verification Check**
- Users must verify email before logging in
- Returns `403 Forbidden` if email not verified

### **7. Security Features**
- OTP: 4-digit code, 1-minute expiration, 3 attempts max
- Rate limiting on OTP send/resend
- Password requirements: min 8 chars, must contain letter & number
- Separate flows for forgot vs. change password

---

## 📊 Routes Summary

| Category | Routes | Swagger Docs | Status |
|----------|--------|--------------|--------|
| Auth | 6 | 6 | ✅ 100% |
| OTP | 3 | 3 | ✅ 100% |
| **Total** | **9** | **9** | ✅ **100%** |

---

## 📝 Implementation Files

| Component | File | Status |
|-----------|------|--------|
| **Routes** | `src/routes/v1/auth.route.js` | ✅ |
| **Routes** | `src/routes/v1/otp.route.js` | ✅ |
| **Controllers** | `src/controllers/auth.controller.js` | ✅ |
| **Controllers** | `src/controllers/otp.controller.js` | ✅ |
| **Services** | `src/services/auth.service.js` | ✅ |
| **Validations** | `src/validations/auth.validation.js` | ✅ |
| **Validations** | `src/validations/otp.validation.js` | ✅ |
| **Middleware** | `src/middlewares/auth.js` | ✅ |
| **Models** | `src/models/user.model.js` | ✅ |
| **Models** | `src/models/otp.model.js` | ✅ |

---

## 🎯 Testing

### **Local Swagger UI**
```
http://localhost:3000/v1/docs
```

### **Quick Test Flow**
```bash
# 1. Register
curl -X POST http://localhost:3000/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"Pass@123",...}'

# 2. Verify Email
curl -X POST http://localhost:3000/v1/otp/verify \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","code":"1234","type":"email_verification"}'

# 3. Login
curl -X POST http://localhost:3000/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"Pass@123"}'

# 4. Select Organization
curl -X POST http://localhost:3000/v1/auth/select-organization \
  -H "Content-Type: application/json" \
  -d '{"orgSelectionToken":"...","organizationId":"..."}'
```

---

## ✅ Documentation Verification

- [x] All routes have Swagger documentation
- [x] Swagger docs match code implementation exactly
- [x] Request/response schemas defined
- [x] All error codes documented (400, 401, 403, 404, 429)
- [x] Examples provided for all use cases
- [x] Security requirements specified
- [x] Type enums documented
- [x] Conditional fields explained (password in verify)
- [x] Rate limiting documented
- [x] Expiration times noted
- [x] Complete flow examples provided

---

## 🚀 Status: Production Ready ✅

- ✅ All 9 routes implemented
- ✅ 100% Swagger documentation coverage
- ✅ Type-safe validation with Joi
- ✅ Secure OTP implementation
- ✅ Stateless JWT authentication
- ✅ Organization-based multi-tenancy
- ✅ Email verification enforcement
- ✅ Rate limiting and security features
- ✅ Clean, maintainable code structure

---

**Last Updated:** October 29, 2024  
**Status:** ✅ **Complete and Production Ready**

