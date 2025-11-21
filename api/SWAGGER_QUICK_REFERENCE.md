# Swagger Documentation - Quick Reference

## 🚀 Quick Start

### 1. View Documentation
```bash
npm start
# Visit: http://localhost:3000/v1/docs
```

### 2. Test with Fresh Data
```bash
npm run seed
```

---

## 🔑 Test Credentials

### Super Admin (Fixed)
```
Email: superadmin@bridge-bond.com
Password: Super@123
```

### Organization Users (Random emails - check output)
```bash
# Get actual emails:
node src/scripts/checkUsers.js

# Default passwords:
Org Admins: Admin@123
Users: User@123
```

---

## 🔐 Authentication Flow

### Step 1: Login
```bash
POST /v1/auth/login
{
  "email": "superadmin@bridge-bond.com",
  "password": "Super@123"
}

# Returns:
{
  "user": {...},
  "organizations": [{id, name, logo, ...}],
  "orgSelectionToken": "temporary-token"
}
```

### Step 2: Select Organization
```bash
POST /v1/auth/select-organization
{
  "orgSelectionToken": "<from-step-1>",
  "organizationId": "<choose-from-list>"
}

# Returns:
{
  "accessToken": "jwt-with-org-context",
  "refreshToken": "jwt-refresh-token"
}
```

### Use Access Token
```bash
GET /v1/users
Authorization: Bearer <accessToken>
```

---

## ✉️ Email Verification (OTP)

### Send OTP
```bash
POST /v1/otp/email-verification/send
{
  "email": "user@example.com"
}
```

### Verify OTP (within 1 minute)
```bash
POST /v1/otp/email-verification/verify
{
  "email": "user@example.com",
  "otpCode": "1234"
}
```

---

## 🔒 Password Management

### Forgot Password (No Auth Required)
```bash
# Step 1: Send OTP
POST /v1/otp/password-reset/send
{
  "email": "user@example.com"
}

# Step 2: Verify OTP & Reset (within 1 minute)
POST /v1/otp/password-reset/verify
{
  "email": "user@example.com",
  "otpCode": "1234",
  "password": "NewPass@123"
}
```

### Change Password (Auth Required)
```bash
POST /v1/auth/change-password
Authorization: Bearer <accessToken>
{
  "oldPassword": "OldPass@123",
  "newPassword": "NewPass@123"
}
```

---

## 🏢 Organization-Based Filtering

### How It Works

1. **JWT contains organizationId:**
```json
{
  "sub": "user-id",
  "organizationId": "org-id",  // ⭐ Current organization
  "type": "access"
}
```

2. **All requests automatically filtered:**
```bash
GET /v1/users
# Returns only users in YOUR organization (from token)

GET /v1/questions
# Returns only questions for YOUR organization (from token)

GET /v1/departments
# Returns only departments in YOUR organization (from token)
```

3. **Superadmin bypass:**
```bash
# Superadmin can see ALL organizations
GET /v1/users
# Returns users from ALL organizations
```

---

## 🔗 Dynamic Population

### Basic Examples

```bash
# Populate single field
GET /v1/users?populate=[{"path":"department"}]

# Populate multiple fields
GET /v1/users?populate=[{"path":"department"},{"path":"organizationId"}]

# Nested population
GET /v1/users?populate=[{"path":"department","populate":["organizationId"]}]

# With field selection
GET /v1/users?populate=[{"path":"department","select":"name description"}]

# With filtering
GET /v1/users?populate=[{"path":"celebrations","match":{"isActive":true}}]
```

### URL-Encoded (for browsers)
```
/v1/users?populate=%5B%7B%22path%22%3A%22department%22%7D%5D
```

---

## ❓ Questions System

### Master Questions (Superadmin Only)

```bash
# Create master question
POST /v1/questions
Authorization: Bearer <superadmin-token>
{
  "title": "What motivates you?",
  "questionType": "multiline",
  "isMasterQuestion": true,
  "assignedOrganizations": ["org1", "org2"]
}
```

### Customized Questions (Org Admin)

```bash
# Modify master question (creates org-specific copy)
PATCH /v1/questions/<master-question-id>
Authorization: Bearer <org-admin-token>
{
  "title": "What motivates you at our company?"
}
# Original master question unchanged!
```

### Get Questions with Responses & Reactions

```bash
GET /v1/questions
Authorization: Bearer <token>

# Response includes:
{
  "results": [
    {
      "id": "...",
      "title": "...",
      "myResponses": [
        // Current user's responses only
      ],
      "reactions": {
        "like": [
          {userId, userName, userEmail, userImage, createdAt}
        ],
        "love": [...]
      },
      "stats": {
        "myResponsesCount": 1,
        "totalReactions": 5,
        "totalResponses": 15
      }
    }
  ]
}
```

---

## 👥 Multi-Organization Users

### Organization-Specific Profile

Each user can have **different profiles** in different organizations:

```json
{
  "email": "john@example.com",  // Global
  "firstName": "John",           // Global
  "organizationMemberships": [
    {
      "organizationId": "org1",
      "role": "user",
      "jobTitle": "Software Engineer",  // Different per org
      "department": "dept1",
      "dateOfHire": "2020-01-01",
      "employeeId": "EMP12345",
      "workEmail": "john@org1.com"
    },
    {
      "organizationId": "org2",
      "role": "org_admin",
      "jobTitle": "Senior Developer",   // Different per org
      "department": "dept2",
      "dateOfHire": "2022-06-15",
      "employeeId": "DEV042",
      "workEmail": "john.doe@org2.com"
    }
  ]
}
```

---

## 📊 Role-Based Access

| Role | Access Level |
|------|-------------|
| **superadmin** | All organizations, bypasses filtering |
| **org_admin** | Only current organization (from token) |
| **user** | Only current organization (from token) |

---

## 🎯 Common API Patterns

### Pagination
```bash
GET /v1/users?page=1&limit=10&sortBy=createdAt:desc
```

### Filtering + Population
```bash
GET /v1/users?role=user&populate=[{"path":"department"}]
```

### Create User (Simple Mode)
```bash
POST /v1/users
Authorization: Bearer <admin-token>
{
  "firstName": "John",
  "lastName": "Doe",
  "email": "john@example.com",
  "password": "Pass@123",
  "dob": "1990-01-15",
  "dateOfHire": "2020-03-01",
  "department": "dept-id",
  "jobTitle": "Software Engineer",
  "organizationId": "org-id"  // Auto-builds memberships
}
```

---

## 🛠️ Troubleshooting

### Email Not Verified
```
Error: 403 - Please verify your email before logging in
Solution: Use OTP verification flow
```

### Organization Access Denied
```
Error: 403 - You no longer have access to this organization
Solution: User removed from organization, use different org
```

### OTP Expired
```
Error: 400 - OTP has expired
Solution: Request new OTP (max 1 per minute)
```

### Invalid Populate Syntax
```
Error: 400 - Invalid populate format
Solution: Use JSON array: [{"path":"fieldName"}]
```

---

## 📁 Documentation Files

1. **[SWAGGER_COMPLETE_UPDATE_2024.md](SWAGGER_COMPLETE_UPDATE_2024.md)** - Complete changelog
2. **[AUTHENTICATION_AND_ORGANIZATION_GUIDE.md](AUTHENTICATION_AND_ORGANIZATION_GUIDE.md)** - Auth flows
3. **[PASSWORD_MANAGEMENT_GUIDE.md](PASSWORD_MANAGEMENT_GUIDE.md)** - Password operations
4. **[ORGANIZATION_BASED_ROUTES_STATUS.md](ORGANIZATION_BASED_ROUTES_STATUS.md)** - Route status
5. **[API_TESTING_GUIDE.md](API_TESTING_GUIDE.md)** - cURL examples

---

## ✨ Key Features

✅ **Two-step authentication** with organization selection  
✅ **OTP-based verification** (1-minute expiration)  
✅ **Automatic organization filtering** from JWT  
✅ **Multi-organization users** with separate profiles  
✅ **Master/customized questions** system  
✅ **Dynamic population** for all GET endpoints  
✅ **Stateless JWT** authentication  
✅ **Role-based access control**  
✅ **Complete Swagger documentation**  

---

**Last Updated:** October 29, 2024  
**Swagger UI:** http://localhost:3000/v1/docs

