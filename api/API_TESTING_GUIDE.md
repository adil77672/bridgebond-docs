# Bridge Bond - API Testing Guide

This guide provides sample API requests with the correct structure and required fields.

## 🔧 Base URL

```
http://localhost:2323/v1
```

## 📖 Interactive Documentation

Visit `http://localhost:2323/v1/docs` for interactive Swagger documentation.

---

## 🔐 Authentication Endpoints

### 1. Register User

**POST** `/auth/register`

```json
{
  "firstName": "John",
  "lastName": "Doe",
  "email": "john.doe@techcorp.com",
  "password": "Pass@123",
  "do": "1990-01-15",
  "dateOfHire": "2020-03-01",
  "department": "5ebac534954b54139806c114",
  "jobTitle": "Software Engineer",
  "role": "user"
}
```

**Required Fields:**
- `firstName` (string)
- `lastName` (string)
- `email` (string, valid email format)
- `password` (string, min 8 chars, at least one letter and one number)
- `dob` (date, ISO format: YYYY-MM-DD, date of birth)
- `dateOfHire` (date, ISO format: YYYY-MM-DD)
- `department` (string, valid MongoDB ObjectId)
- `jobTitle` (string)

**Optional Fields:**
- `role` (string: "user" or "org_admin", defaults to "user")

**Response:**
```json
{
  "user": {
    "id": "5ebac534954b54139806c112",
    "firstName": "John",
    "lastName": "Doe",
    "email": "john.doe@techcorp.com",
    "role": "user",
    "isEmailVerified": false,
    "isActive": true
  },
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

---

### 2. Login

**POST** `/auth/login`

```json
{
  "email": "john.doe@techcorp.com",
  "password": "Pass@123"
}
```

**Response:**
```json
{
  "user": {
    "id": "5ebac534954b54139806c112",
    "firstName": "Super",
    "lastName": "Admin",
    "email": "superadmin@bridge-bond.com",
    "role": "superadmin",
    "isEmailVerified": true,
    "isActive": true
  },
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

---

### 3. Refresh Tokens

**POST** `/auth/refresh-tokens`

```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

---

### 4. Forgot Password

**POST** `/auth/forgot-password`

```json
{
  "email": "john.doe@techcorp.com"
}
```

---

### 5. Reset Password

**POST** `/auth/reset-password`

```json
{
  "token": "password-reset-token-here",
  "password": "NewPass@123"
}
```

---

## 👥 User Management Endpoints

### 1. Create User

**POST** `/users`  
**Auth Required:** Yes  
**Permission:** manageUsers

```json
{
  "firstName": "Jane",
  "lastName": "Smith",
  "email": "jane.smith@techcorp.com",
  "password": "Pass@123",
  "dob": "1992-05-20",
  "dateOfHire": "2021-06-15",
  "department": "5ebac534954b54139806c114",
  "jobTitle": "Senior Developer",
  "role": "user",
  "organizationMemberships": [
    {
      "organizationId": "5ebac534954b54139806c113",
      "role": "user",
      "isActive": true
    }
  ],
  "departmentMemberships": [
    {
      "departmentId": "5ebac534954b54139806c114",
      "organizationId": "5ebac534954b54139806c113",
      "isActive": true
    }
  ]
}
```

---

### 2. Get All Users

**GET** `/users?page=1&limit=10&role=user&sortBy=createdAt:desc`  
**Auth Required:** Yes  
**Permission:** getUsers

**Query Parameters:**
- `firstName` (string, optional)
- `lastName` (string, optional)
- `email` (string, optional)
- `role` (string: "user", "org_admin", "superadmin", optional)
- `department` (string, ObjectId, optional)
- `organizationId` (string, ObjectId, optional)
- `isActive` (boolean, optional)
- `isEmailVerified` (boolean, optional)
- `page` (number, default: 1)
- `limit` (number, default: 10, max: 100)
- `sortBy` (string, format: `field:asc` or `field:desc`)

---

### 3. Get User by ID

**GET** `/users/:userId`  
**Auth Required:** Yes  
**Permission:** getUsers

---

### 4. Update User

**PATCH** `/users/:userId`  
**Auth Required:** Yes  
**Permission:** manageUsers

```json
{
  "firstName": "John",
  "jobTitle": "Lead Software Engineer",
  "isActive": true
}
```

---

### 5. Delete User

**DELETE** `/users/:userId`  
**Auth Required:** Yes  
**Permission:** manageUsers

---

## 🏢 Organization Management

### 1. Create Organization

**POST** `/organizations`  
**Auth Required:** Yes (Superadmin only)  
**Permission:** manageOrganizations

```json
{
  "name": "TechCorp Solutions",
  "description": "Leading technology company",
  "domains": ["techcorp.com", "techcorp.io"],
  "adminId": "5ebac534954b54139806c112",
  "isActive": true,
  "settings": {
    "allowPublicSignup": true,
    "requireEmailVerification": true,
    "maxUsersPerDepartment": 100
  }
}
```

**Required Fields:**
- `name` (string, unique)
- `adminId` (string, ObjectId of organization admin user)

---

### 2. Get All Organizations

**GET** `/organizations?page=1&limit=10`  
**Auth Required:** Yes  
**Permission:** getOrganizations

---

### 3. Get Organization Details

**GET** `/organizations/:organizationId`  
**Auth Required:** Yes  
**Permission:** getOrganization

---

### 4. Update Organization

**PATCH** `/organizations/:organizationId`  
**Auth Required:** Yes  
**Permission:** updateOrganization

```json
{
  "description": "Updated description",
  "isActive": true,
  "domains": ["techcorp.com", "techcorp.net"]
}
```

---

### 5. Get Organization Users

**GET** `/organizations/:organizationId/users`  
**Auth Required:** Yes  
**Permission:** getOrgUsers

---

### 6. Get Organization Departments

**GET** `/organizations/:organizationId/departments`  
**Auth Required:** Yes  
**Permission:** getDepartments

---

## 🏛️ Department Management

### 1. Create Department

**POST** `/departments`  
**Auth Required:** Yes  
**Permission:** manageDepartments

```json
{
  "name": "Engineering",
  "description": "Engineering department",
  "organizationId": "5ebac534954b54139806c113",
  "isActive": true,
  "settings": {
    "teamSize": 25,
    "budget": 500000
  }
}
```

**Required Fields:**
- `name` (string)
- `organizationId` (string, ObjectId)

---

### 2. Get All Departments

**GET** `/departments?organizationId=5ebac534954b54139806c113&page=1&limit=10`  
**Auth Required:** Yes  
**Permission:** getDepartments

---

### 3. Get Department Details

**GET** `/departments/:departmentId`  
**Auth Required:** Yes  
**Permission:** getDepartments

---

### 4. Update Department

**PATCH** `/departments/:departmentId`  
**Auth Required:** Yes  
**Permission:** manageDepartments

```json
{
  "description": "Updated engineering department",
  "isActive": true
}
```

---

### 5. Get Department Users

**GET** `/departments/:departmentId/users`  
**Auth Required:** Yes  
**Permission:** getDepartments

---

### 6. Add User to Department

**POST** `/departments/:departmentId/users`  
**Auth Required:** Yes  
**Permission:** manageDepartments

```json
{
  "userId": "5ebac534954b54139806c112"
}
```

---

### 7. Remove User from Department

**DELETE** `/departments/:departmentId/users/:userId`  
**Auth Required:** Yes  
**Permission:** manageDepartments

---

## 🔢 OTP Endpoints

### 1. Send Email Verification OTP

**POST** `/otp/email-verification/send`  
**Auth Required:** Yes

```json
{
  "email": "john.doe@techcorp.com"
}
```

**Response:**
```json
{
  "message": "OTP sent successfully",
  "expiresAt": "2024-10-28T12:15:00.000Z"
}
```

---

### 2. Verify Email with OTP

**POST** `/otp/email-verification/verify`  
**Auth Required:** Yes

```json
{
  "email": "john.doe@techcorp.com",
  "code": "1234"
}
```

---

### 3. Send Password Reset OTP

**POST** `/otp/password-reset/send`  
**Auth Required:** No

```json
{
  "email": "john.doe@techcorp.com"
}
```

---

### 4. Reset Password with OTP

**POST** `/otp/password-reset/verify`  
**Auth Required:** No

```json
{
  "email": "john.doe@techcorp.com",
  "code": "1234",
  "password": "NewPass@123"
}
```

---

### 5. Resend OTP

**POST** `/otp/resend`  
**Auth Required:** Varies by type

```json
{
  "email": "john.doe@techcorp.com",
  "type": "email_verification"
}
```

---

## 🔑 Authentication Headers

For authenticated requests, include the JWT token in the Authorization header:

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

---

## ❌ Common Error Responses

### Validation Error (400)

```json
{
  "code": 400,
  "message": "Validation error: firstName is required, dob must be a valid date"
}
```

### Unauthorized (401)

```json
{
  "code": 401,
  "message": "Please authenticate"
}
```

### Forbidden (403)

```json
{
  "code": 403,
  "message": "Forbidden"
}
```

### Not Found (404)

```json
{
  "code": 404,
  "message": "User not found"
}
```

### Duplicate Email (400)

```json
{
  "code": 400,
  "message": "Email already taken"
}
```

---

## 🧪 Testing with cURL

### Example: Create a User

```bash
curl -X POST http://localhost:2323/v1/users \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -d '{
    "firstName": "John",
    "lastName": "Doe",
    "email": "john.doe@techcorp.com",
    "password": "Pass@123",
    "dob": "1990-01-15",
    "dateOfHire": "2020-03-01",
    "department": "5ebac534954b54139806c114",
    "jobTitle": "Software Engineer"
  }'
```

### Example: Login

```bash
curl -X POST http://localhost:2323/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "superadmin@bridge-bond.com",
    "password": "Super@123"
  }'
```

---

## 📝 Tips

1. **Date Format**: Always use ISO 8601 format for dates: `YYYY-MM-DD` or `YYYY-MM-DDTHH:mm:ss.sssZ`

2. **ObjectId Format**: MongoDB ObjectIds are 24-character hexadecimal strings (e.g., `5ebac534954b54139806c112`)

3. **Password Requirements**: 
   - Minimum 8 characters
   - At least one letter
   - At least one number

4. **Role Hierarchy**:
   - `superadmin`: Full system access
   - `org_admin`: Organization-level access
   - `user`: Department-level access

5. **Testing with Seeded Data**: Run `npm run seed` to populate the database with test data. See [SEED_QUICK_START.md](SEED_QUICK_START.md) for credentials.

6. **Pagination**: Use `page` and `limit` query parameters. Maximum limit is 100.

7. **Sorting**: Use `sortBy` parameter with format `field:asc` or `field:desc` (e.g., `sortBy=createdAt:desc`)

---

## 🔗 Related Documentation

- [README.md](README.md) - Project overview and setup
- [SEED_QUICK_START.md](SEED_QUICK_START.md) - Database seeding guide
- [SAMPLE_CREDENTIALS.md](SAMPLE_CREDENTIALS.md) - Sample login credentials
- [CREDENTIALS_LIST.txt](CREDENTIALS_LIST.txt) - Quick credential reference

---

**Last Updated:** October 28, 2025

