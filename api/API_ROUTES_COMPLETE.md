# Complete API Routes Reference

## ✅ Current Routes (Exact Match with Code)

---

## 🔐 Authentication Routes (`/v1/auth/*`)

| Method | Endpoint | Auth Required | Description |
|--------|----------|---------------|-------------|
| POST | `/auth/register` | No | Register new user (auto-sends OTP) |
| POST | `/auth/login` | No | Login (returns orgSelectionToken) |
| POST | `/auth/select-organization` | No | Select organization (returns access tokens) |
| POST | `/auth/logout` | No | Logout (client-side token removal) |
| POST | `/auth/refresh-tokens` | No | Refresh access token |
| POST | `/auth/change-password` | Yes | Change password (authenticated users) |

---

## 📧 OTP Routes (`/v1/otp/*`)

| Method | Endpoint | Auth Required | Description |
|--------|----------|---------------|-------------|
| POST | `/otp/send` | No | Send OTP (type: email_verification or password_reset) |
| POST | `/otp/verify` | No | Verify OTP (type-based) |
| POST | `/otp/resend` | No | Resend OTP (same as /send) |

---

## 👥 User Routes (`/v1/users/*`)

| Method | Endpoint | Auth Required | Permission | Description |
|--------|----------|---------------|------------|-------------|
| POST | `/users` | Yes | manageUsers | Create user |
| GET | `/users` | Yes | getUsers | Get all users (org-filtered) |
| GET | `/users/:userId` | Yes | getUsers | Get single user |
| PATCH | `/users/:userId` | Yes | manageUsers | Update user |
| DELETE | `/users/:userId` | Yes | manageUsers | Delete user |

---

## ❓ Question Routes (`/v1/questions/*`)

| Method | Endpoint | Auth Required | Permission | Description |
|--------|----------|---------------|------------|-------------|
| POST | `/questions` | Yes | manageUsers | Create question |
| GET | `/questions` | Yes | - | Get all questions (org-filtered) |
| GET | `/questions/:questionId` | Yes | - | Get single question |
| PATCH | `/questions/:questionId` | Yes | manageUsers | Update question |
| DELETE | `/questions/:questionId` | Yes | manageUsers | Delete question |
| GET | `/questions/:questionId/details` | Yes | - | Get question with responses & reactions |
| GET | `/questions/:questionId/responses` | Yes | - | Get responses for question |
| POST | `/questions/responses` | Yes | - | Submit response |
| DELETE | `/questions/responses/:responseId` | Yes | - | Delete response |
| POST | `/questions/reactions` | Yes | - | Add reaction |
| GET | `/questions/reactions` | Yes | - | Get reactions |
| DELETE | `/questions/reactions/:reactionId` | Yes | - | Remove reaction |
| GET | `/questions/alerts/:organizationId` | Yes | - | Get alert settings |
| PATCH | `/questions/alerts/:organizationId` | Yes | - | Update alert settings |
| GET | `/questions/master` | Yes | manageUsers | Get master questions (superadmin) |
| POST | `/questions/:questionId/assign` | Yes | manageUsers | Assign master question (superadmin) |
| POST | `/questions/:questionId/unassign` | Yes | manageUsers | Unassign question (superadmin) |
| GET | `/questions/:questionId/versions` | Yes | manageUsers | Get customized versions (superadmin) |

---

## 🏢 Organization Routes (`/v1/organizations/*`)

_Routes to be verified..._

---

## 🎉 Celebration Routes (`/v1/celebrations/*`)

_Routes to be verified..._

---

## 🏷️ Department Routes (`/v1/departments/*`)

_Routes to be verified..._

---

## 🎂 DOB Alert Routes (`/v1/dob-alerts/*`)

_Routes to be verified..._

---

## 📊 Audit Log Routes (`/v1/audit-logs/*`)

_Routes to be verified..._

---

## 📄 Documentation Routes (`/v1/docs`)

| Method | Endpoint | Auth Required | Description |
|--------|----------|---------------|-------------|
| GET | `/docs` | No | Swagger UI documentation |

---

## 🔄 Complete Authentication Flow

### 1. Register
```bash
POST /v1/auth/register
{
  "email": "user@example.com",
  "password": "Pass@123",
  "firstName": "John",
  "lastName": "Doe",
  "dob": "1990-01-15",
  "dateOfHire": "2020-03-01",
  "department": "5ebac534954b54139806c114",
  "jobTitle": "Software Engineer",
  "organizationId": "5ebac534954b54139806c113"
}

Response:
{
  "user": { ... },
  "message": "Registration successful. Please check your email for verification OTP."
}
```

### 2. Verify Email
```bash
POST /v1/otp/verify
{
  "email": "user@example.com",
  "code": "1234",
  "type": "email_verification"
}
```

### 3. Login
```bash
POST /v1/auth/login
{
  "email": "user@example.com",
  "password": "Pass@123"
}

Response:
{
  "orgSelectionToken": "temp_token_here",
  "organizations": [
    { "id": "...", "name": "TechCorp" },
    { "id": "...", "name": "StartupInc" }
  ]
}
```

### 4. Select Organization
```bash
POST /v1/auth/select-organization
{
  "orgSelectionToken": "temp_token_here",
  "organizationId": "5ebac534954b54139806c113"
}

Response:
{
  "tokens": {
    "access": { "token": "...", "expires": "..." },
    "refresh": { "token": "...", "expires": "..." }
  },
  "user": { ... }
}
```

### 5. Forgot Password
```bash
# Request OTP
POST /v1/otp/send
{
  "email": "user@example.com",
  "type": "password_reset"
}

# Reset Password
POST /v1/otp/verify
{
  "email": "user@example.com",
  "code": "1234",
  "type": "password_reset",
  "password": "NewPass@123"
}
```

### 6. Change Password (Authenticated)
```bash
POST /v1/auth/change-password
Headers: { "Authorization": "Bearer <access_token>" }
{
  "oldPassword": "Pass@123",
  "newPassword": "NewPass@456"
}
```

---

## ✅ Swagger Documentation Status

| Route File | Swagger Docs | Status |
|------------|--------------|--------|
| `auth.route.js` | ✅ Complete | Up to date |
| `otp.route.js` | ⚠️ Needs verification | Checking... |
| `user.route.js` | ✅ Complete | Up to date |
| `question.route.js` | ✅ Complete | Up to date |
| `organization.route.js` | ⏳ To verify | - |
| `celebration.route.js` | ⏳ To verify | - |
| `department.route.js` | ⏳ To verify | - |
| `dobAlert.route.js` | ⏳ To verify | - |
| `auditLog.route.js` | ⏳ To verify | - |

---

**Last Updated:** October 29, 2024

