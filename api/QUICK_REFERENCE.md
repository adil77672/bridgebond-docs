# API Quick Reference Card

## 🔐 Authentication & OTP Endpoints

### **Registration & Email Verification**

```bash
# 1. Register (OTP auto-sent)
POST /v1/auth/register
Body: { email, password, firstName, lastName, dob, dateOfHire, department, jobTitle, organizationId }

# 2. Verify Email
POST /v1/otp/verify
Body: { email, code, type: "email_verification" }

# Resend OTP
POST /v1/otp/resend
Body: { email, type: "email_verification" }
```

---

### **Login (Two-Step)**

```bash
# Step 1: Login
POST /v1/auth/login
Body: { email, password }
→ Returns: { orgSelectionToken, organizations[] }

# Step 2: Select Organization
POST /v1/auth/select-organization
Body: { orgSelectionToken, organizationId }
→ Returns: { tokens: { access, refresh }, user }
```

---

### **Password Management**

```bash
# Forgot Password (No Auth Required)
POST /v1/otp/send
Body: { email, type: "password_reset" }

POST /v1/otp/verify
Body: { email, code, type: "password_reset", password: "NewPass@123" }

# Change Password (Auth Required)
POST /v1/auth/change-password
Headers: { Authorization: "Bearer <token>" }
Body: { oldPassword, newPassword }
```

---

### **Token Management**

```bash
# Refresh Tokens
POST /v1/auth/refresh-tokens
Body: { refreshToken }

# Logout
POST /v1/auth/logout
```

---

## 🎯 OTP Types

| Type | Purpose | Password Required |
|------|---------|-------------------|
| `email_verification` | Verify email after registration | ❌ No |
| `password_reset` | Reset forgotten password | ✅ Yes |

---

## ⏱️ Timeouts & Limits

| Feature | Value |
|---------|-------|
| OTP Expiration | 1 minute |
| OTP Attempts | 3 max |
| OTP Rate Limit | 1 per minute |
| Access Token Expiry | 1 hour |
| Refresh Token Expiry | 30 days |

---

## 📍 Complete Route List

### Auth Routes (`/v1/auth/*`)
- `POST /auth/register` - Register user
- `POST /auth/login` - Login (step 1)
- `POST /auth/select-organization` - Select org (step 2)
- `POST /auth/logout` - Logout
- `POST /auth/refresh-tokens` - Refresh access token
- `POST /auth/change-password` 🔒 - Change password

### OTP Routes (`/v1/otp/*`)
- `POST /otp/send` - Send OTP
- `POST /otp/verify` - Verify OTP
- `POST /otp/resend` - Resend OTP

---

## 🔑 Request Headers

```bash
# Authenticated Requests
Authorization: Bearer <access_token>
Content-Type: application/json
```

---

## 📊 Response Codes

| Code | Meaning |
|------|---------|
| 200 | Success |
| 400 | Bad Request (invalid data, expired OTP) |
| 401 | Unauthorized (invalid/expired token) |
| 403 | Forbidden (email not verified, insufficient permissions) |
| 404 | Not Found (user not found) |
| 429 | Too Many Requests (rate limit exceeded) |

---

## 🧪 Swagger UI

**Local:** `http://localhost:3000/v1/docs`

---

## ✅ Complete Flow Example

```bash
# 1. Register
POST /v1/auth/register → OTP auto-sent

# 2. Verify Email
POST /v1/otp/verify (type: email_verification)

# 3. Login
POST /v1/auth/login → orgSelectionToken

# 4. Select Org
POST /v1/auth/select-organization → access & refresh tokens

# 5. Make Authenticated Requests
Use access token in Authorization header

# 6. Refresh When Expired
POST /v1/auth/refresh-tokens
```

---

**Documentation:** `/docs/api/AUTH_OTP_COMPLETE.md`  
**Swagger:** `http://localhost:3000/v1/docs`

