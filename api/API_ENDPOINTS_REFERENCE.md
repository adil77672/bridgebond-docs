# API Endpoints Reference - Minimal & Detailed

**Base URL:** `http://localhost:3000/v1`  
**Swagger UI:** `http://localhost:3000/v1/docs`

---

## 📑 Table of Contents

1. [Authentication](#authentication) - 6 endpoints
2. [OTP](#otp) - 5 endpoints
3. [Users](#users) - 5 endpoints
4. [Organizations](#organizations) - 8 endpoints
5. [Departments](#departments) - 7 endpoints
6. [Questions](#questions) - 11 endpoints
7. [Celebrations](#celebrations) - 5 endpoints
8. [DOB Alerts](#dob-alerts) - 5 endpoints
9. [Audit Logs](#audit-logs) - 6 endpoints

**Total: 58 Endpoints**

---

## 🔐 Authentication

### 1. Register
```http
POST /auth/register
```

**Purpose:** Create new user account with automatic OTP email verification  
**Auth:** None (Public)  
**Organization Filter:** N/A

**Request:**
```json
{
  "firstName": "John",
  "lastName": "Doe",
  "email": "john@example.com",
  "password": "Pass@123",
  "dob": "1990-01-15",
  "dateOfHire": "2020-03-01",
  "department": "dept-id",
  "jobTitle": "Software Engineer",
  "organizationId": "org-id",  // Auto-builds memberships
  "role": "user"  // Optional: user, org_admin
}
```

**Response:**
```json
{
  "user": {
    "id": "user-id",
    "email": "john@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "user",
    "isEmailVerified": false
  },
  "message": "Registration successful. Please check your email for verification OTP.",
  "otp": {
    "expiresAt": "2024-10-29T12:01:00.000Z",
    "message": "OTP sent to your email (expires in 1 minute)"
  }
}
```

**Notes:**
- Password: Min 8 chars, must have letter + number
- OTP automatically sent to email
- Must verify email before login (403 error otherwise)

---

### 2. Login (Step 1 - Organization Selection)
```http
POST /auth/login
```

**Purpose:** Authenticate user and get available organizations  
**Auth:** None (Public)  
**Organization Filter:** N/A

**Request:**
```json
{
  "email": "john@example.com",
  "password": "Pass@123"
}
```

**Response:**
```json
{
  "user": {
    "id": "user-id",
    "email": "john@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "user"
  },
  "organizations": [
    {
      "_id": "org-id-1",
      "name": "TechCorp Solutions",
      "description": "Leading tech company",
      "logo": "https://...",
      "isActive": true
    },
    {
      "_id": "org-id-2",
      "name": "Innovate Digital",
      "description": "Digital innovation",
      "logo": "https://...",
      "isActive": true
    }
  ],
  "orgSelectionToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Errors:**
- `401` - Incorrect email or password
- `403` - Email not verified

**Notes:**
- Email must be verified before login
- `orgSelectionToken` expires in 30 days
- Use token + organizationId for next step

---

### 3. Select Organization (Step 2 - Get Access Tokens)
```http
POST /auth/select-organization
```

**Purpose:** Choose organization and get access tokens with organization context  
**Auth:** None (uses orgSelectionToken)  
**Organization Filter:** N/A

**Request:**
```json
{
  "orgSelectionToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "organizationId": "org-id-1"
}
```

**Response:**
```json
{
  "user": {
    "id": "user-id",
    "email": "john@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "user"
  },
  "organization": {
    "id": "org-id-1",
    "name": "TechCorp Solutions",
    "description": "Leading tech company",
    "logo": "https://..."
  },
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Token Payload:**
```json
{
  "sub": "user-id",
  "organizationId": "org-id-1",  // ⭐ Organization context
  "type": "access",
  "iat": 1234567890,
  "exp": 1234655890
}
```

**Errors:**
- `401` - Invalid or expired orgSelectionToken
- `403` - User doesn't have access to this organization
- `404` - Organization not found

**Notes:**
- Access token expires in 48 hours (default)
- Refresh token expires in 5 days (default)
- Both tokens are stateless (not stored in DB)

---

### 4. Logout
```http
POST /auth/logout
```

**Purpose:** Client-side logout (stateless)  
**Auth:** None (optional)  
**Organization Filter:** N/A

**Response:**
```json
{
  "message": "Logged out successfully"
}
```

**Notes:**
- Tokens are stateless (not stored in DB)
- Simply discard tokens on client side
- Tokens will expire naturally

---

### 5. Refresh Tokens
```http
POST /auth/refresh-tokens
```

**Purpose:** Get new access token using refresh token  
**Auth:** None (uses refreshToken)  
**Organization Filter:** Maintains org context

**Request:**
```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response:**
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Errors:**
- `401` - Invalid or expired refresh token
- `403` - User no longer has access to organization or email not verified

**Notes:**
- Maintains same organization context
- Verifies user still has access to organization
- Requires verified email

---

### 6. Change Password
```http
POST /auth/change-password
```

**Purpose:** Change password for authenticated user  
**Auth:** Bearer Token (Required)  
**Organization Filter:** N/A

**Request:**
```json
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

**Errors:**
- `401` - Incorrect old password or not authenticated
- `404` - User not found

**Notes:**
- Requires authentication token
- Must provide correct old password
- Different from "Forgot Password" (which uses OTP)

---

## 📧 OTP

### 1. Send Email Verification OTP
```http
POST /otp/email-verification/send
```

**Purpose:** Send 4-digit OTP to verify email address  
**Auth:** None (Public)  
**Organization Filter:** N/A

**Request:**
```json
{
  "email": "john@example.com"
}
```

**Response:**
```json
{
  "message": "OTP sent successfully",
  "email": "john@example.com",
  "expiresAt": "2024-10-29T12:01:00.000Z"
}
```

**Errors:**
- `400` - Email already verified
- `404` - User not found

**Notes:**
- OTP expires in **1 MINUTE**
- Email-based (no auth required)
- Automatically sent on registration

---

### 2. Verify Email OTP
```http
POST /otp/email-verification/verify
```

**Purpose:** Verify email with 4-digit OTP code  
**Auth:** None (Public)  
**Organization Filter:** N/A

**Request:**
```json
{
  "email": "john@example.com",
  "otpCode": "1234"
}
```

**Response:**
```json
{
  "message": "Email verified successfully"
}
```

**Errors:**
- `400` - Invalid OTP code (shows remaining attempts)
- `400` - OTP expired (request new one)
- `400` - Maximum attempts exceeded (3 attempts max)

**Notes:**
- Must use within 1 minute
- Max 3 attempts per OTP
- After verification, can login

---

### 3. Send Password Reset OTP
```http
POST /otp/password-reset/send
```

**Purpose:** Send OTP for forgot password flow  
**Auth:** None (Public)  
**Organization Filter:** N/A

**Request:**
```json
{
  "email": "john@example.com"
}
```

**Response:**
```json
{
  "message": "OTP sent successfully",
  "email": "john@example.com",
  "expiresAt": "2024-10-29T12:01:00.000Z"
}
```

**Errors:**
- `404` - User not found

**Notes:**
- OTP expires in **1 MINUTE**
- No authentication required
- Different from "Change Password" (which requires auth)

---

### 4. Verify Password Reset OTP
```http
POST /otp/password-reset/verify
```

**Purpose:** Verify OTP and reset password in one step  
**Auth:** None (Public)  
**Organization Filter:** N/A

**Request:**
```json
{
  "email": "john@example.com",
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

**Errors:**
- `400` - Invalid or expired OTP
- `400` - Invalid password format

**Notes:**
- Must use within 1 minute
- Max 3 attempts per OTP
- Resets password immediately

---

### 5. Resend OTP
```http
POST /otp/resend
```

**Purpose:** Request new OTP code  
**Auth:** None (Public)  
**Organization Filter:** N/A

**Request:**
```json
{
  "email": "john@example.com",
  "type": "email_verification"  // or "password_reset"
}
```

**Response:**
```json
{
  "message": "OTP sent successfully",
  "email": "john@example.com",
  "expiresAt": "2024-10-29T12:01:00.000Z"
}
```

**Errors:**
- `429` - Rate limit exceeded (wait before requesting new OTP)

**Notes:**
- Rate limited: **1 request per minute**
- Invalidates previous OTP
- New expiration time provided

---

## 👥 Users

### 1. Get All Users
```http
GET /users
```

**Purpose:** Retrieve paginated list of users  
**Auth:** Bearer Token (Required)  
**Organization Filter:** ✅ Automatic (from token)

**Query Parameters:**
```
?name=john              // Filter by name
&role=user              // Filter by role
&sortBy=createdAt:desc  // Sort by field:order
&limit=10               // Items per page (max 100)
&page=1                 // Page number
&populate=[{"path":"department"}]  // Populate references
```

**Response:**
```json
{
  "results": [
    {
      "id": "user-id",
      "firstName": "John",
      "lastName": "Doe",
      "email": "john@example.com",
      "dob": "1990-01-15",
      "role": "user",
      "isEmailVerified": true,
      "isActive": true,
      "organizationMemberships": [
        {
          "organizationId": "org-id",
          "role": "user",
          "isActive": true,
          "jobTitle": "Software Engineer",
          "department": "dept-id",
          "dateOfHire": "2020-03-01",
          "employeeId": "EMP12345",
          "workEmail": "john@org.com"
        }
      ],
      "imageUrl": "https://...",
      "lastLoginAt": "2024-10-29T10:30:00.000Z",
      "createdAt": "2020-03-01T00:00:00.000Z"
    }
  ],
  "page": 1,
  "limit": 10,
  "totalPages": 5,
  "totalResults": 50
}
```

**Access Control:**
- **Superadmin:** All users across all organizations
- **Org Admin/User:** Only users in current organization (from token)

**Notes:**
- `organizationId` automatically set from JWT token
- No need to pass organizationId in query
- Supports dynamic population

---

### 2. Get User by ID
```http
GET /users/{userId}
```

**Purpose:** Get single user details  
**Auth:** Bearer Token (Required)  
**Organization Filter:** ✅ Verified (must be in user's organization)

**Query Parameters:**
```
?populate=[{"path":"department"},{"path":"celebrations"}]
```

**Response:** Same as single user object above

**Access Control:**
- Users can fetch their own data
- Admins can fetch any user in their organization
- Superadmin can fetch any user

---

### 3. Create User
```http
POST /users
```

**Purpose:** Create new user (admin only)  
**Auth:** Bearer Token (Required - Admin)  
**Organization Filter:** ✅ User assigned to organization

**Request:**
```json
{
  "firstName": "Jane",
  "lastName": "Smith",
  "email": "jane@example.com",
  "password": "Pass@123",
  "dob": "1992-05-20",
  "dateOfHire": "2023-06-01",
  "department": "dept-id",
  "jobTitle": "Product Manager",
  "organizationId": "org-id",  // Simple mode
  "role": "user"
}
```

**Response:** User object

**Notes:**
- Only admins can create users
- `organizationId` auto-builds membership arrays
- Email verification required before login

---

### 4. Update User
```http
PATCH /users/{userId}
```

**Purpose:** Update user details  
**Auth:** Bearer Token (Required)  
**Organization Filter:** ✅ Verified

**Request:**
```json
{
  "firstName": "Jane",
  "email": "newemail@example.com",
  "organizationMemberships": [
    {
      "organizationId": "org-id",
      "jobTitle": "Senior Product Manager",
      "department": "new-dept-id"
    }
  ]
}
```

**Response:** Updated user object

**Access Control:**
- Users can update their own basic info
- Admins can update any user in their organization

---

### 5. Delete User
```http
DELETE /users/{userId}
```

**Purpose:** Soft delete user  
**Auth:** Bearer Token (Required - Admin)  
**Organization Filter:** ✅ Verified

**Response:**
```json
{
  "message": "User deleted successfully"
}
```

**Access Control:**
- Users can delete their own account
- Admins can delete users in their organization

**Notes:**
- Soft delete (sets `isDeleted: true`)
- User data preserved for audit

---

## 🏢 Organizations

### 1. Get All Organizations
```http
GET /organizations
```

**Purpose:** Retrieve list of organizations  
**Auth:** Bearer Token (Required)  
**Organization Filter:** ✅ Automatic

**Query Parameters:**
```
?name=TechCorp
&isActive=true
&sortBy=createdAt:desc
&limit=10
&page=1
&populate=[{"path":"adminId","select":"firstName lastName email"}]
```

**Response:**
```json
{
  "results": [
    {
      "id": "org-id",
      "name": "TechCorp Solutions",
      "description": "Leading technology company",
      "domains": ["techcorp.com", "techcorp.io"],
      "isActive": true,
      "adminId": {
        "id": "admin-id",
        "firstName": "John",
        "lastName": "Doe",
        "email": "john@techcorp.com"
      },
      "logo": "https://...",
      "settings": {
        "allowPublicSignup": true,
        "requireEmailVerification": true
      },
      "createdAt": "2024-01-01T00:00:00.000Z",
      "updatedAt": "2024-01-01T00:00:00.000Z"
    }
  ],
  "page": 1,
  "limit": 10,
  "totalPages": 1,
  "totalResults": 1
}
```

**Access Control:**
- **Superadmin:** All organizations
- **Org Admin/User:** Only their current organization (from token)

---

### 2. Get Organization by ID
```http
GET /organizations/{organizationId}
```

**Purpose:** Get single organization details  
**Auth:** Bearer Token (Required)  
**Organization Filter:** ✅ Access verified

**Query Parameters:**
```
?populate=[{"path":"departments"},{"path":"adminId"}]
```

**Response:** Single organization object with departments array

**Access Control:**
- Users can only access organizations they belong to
- Token's organizationId must match (unless superadmin)

---

### 3. Create Organization
```http
POST /organizations
```

**Purpose:** Create new organization (superadmin only)  
**Auth:** Bearer Token (Required - Superadmin)  
**Organization Filter:** N/A

**Request:**
```json
{
  "name": "NewCorp Inc",
  "description": "Innovative company",
  "domains": ["newcorp.com"],
  "adminId": "user-id",
  "isActive": true,
  "settings": {
    "allowPublicSignup": false,
    "requireEmailVerification": true
  }
}
```

**Response:** Organization object

**Notes:**
- Only superadmin can create organizations
- adminId must be existing user

---

### 4. Update Organization
```http
PATCH /organizations/{organizationId}
```

**Purpose:** Update organization details  
**Auth:** Bearer Token (Required - Org Admin)  
**Organization Filter:** ✅ Must be admin of this org

**Request:**
```json
{
  "name": "Updated Corp Name",
  "description": "New description",
  "settings": {
    "maxUsersPerDepartment": 50
  }
}
```

**Response:** Updated organization object

---

### 5. Delete Organization
```http
DELETE /organizations/{organizationId}
```

**Purpose:** Delete organization (superadmin only)  
**Auth:** Bearer Token (Required - Superadmin)  
**Organization Filter:** N/A

**Response:**
```json
{
  "message": "Organization deleted successfully"
}
```

---

### 6. Upload Organization Logo
```http
PATCH /organizations/{organizationId}/logo
```

**Purpose:** Upload organization logo to Cloudinary/S3  
**Auth:** Bearer Token (Required - Org Admin)  
**Organization Filter:** ✅ Must be admin of this org

**Request:**
```
Content-Type: multipart/form-data

logo: <file>  // Image file (jpg, png, gif, webp)
```

**Response:**
```json
{
  "id": "org-id",
  "name": "TechCorp Solutions",
  "logo": "https://res.cloudinary.com/xxx/logo.jpg",
  "logoPublicId": "bridge-bond/organizations/logo"
}
```

**Notes:**
- Automatically deletes old logo
- Optimized for web delivery
- Supports: jpg, png, gif, webp

---

### 7. Get Organization Users
```http
GET /organizations/{organizationId}/users
```

**Purpose:** Get all users in organization  
**Auth:** Bearer Token (Required)  
**Organization Filter:** ✅ Access verified

**Query Parameters:**
```
?sortBy=firstName:asc
&limit=10
&page=1
&populate=[{"path":"department"}]
```

**Response:** Paginated user list

---

### 8. Get Organization Departments
```http
GET /organizations/{organizationId}/departments
```

**Purpose:** Get all departments in organization  
**Auth:** Bearer Token (Required)  
**Organization Filter:** ✅ Access verified

**Query Parameters:**
```
?sortBy=name:asc
&limit=10
&page=1
```

**Response:** Paginated department list

---

## 🏬 Departments

### 1. Get All Departments
```http
GET /departments
```

**Purpose:** Retrieve paginated list of departments  
**Auth:** Bearer Token (Required)  
**Organization Filter:** ✅ Automatic (from token)

**Query Parameters:**
```
?name=Engineering
&isActive=true
&sortBy=name:asc
&limit=10
&page=1
&populate=[{"path":"organizationId"}]
```

**Response:**
```json
{
  "results": [
    {
      "id": "dept-id",
      "name": "Engineering",
      "description": "Engineering department",
      "organizationId": "org-id",
      "isActive": true,
      "settings": {
        "teamSize": 25,
        "budget": 500000
      },
      "createdAt": "2024-01-01T00:00:00.000Z",
      "updatedAt": "2024-01-01T00:00:00.000Z"
    }
  ],
  "page": 1,
  "limit": 10,
  "totalPages": 1,
  "totalResults": 10
}
```

**Access Control:**
- **Superadmin:** All departments across all organizations
- **Org Admin/User:** Only departments in current organization

---

### 2. Get Department by ID
```http
GET /departments/{departmentId}
```

**Purpose:** Get single department details  
**Auth:** Bearer Token (Required)  
**Organization Filter:** ✅ Must be in same organization

**Query Parameters:**
```
?populate=[{"path":"organizationId"}]
```

**Response:** Single department object

---

### 3. Create Department
```http
POST /departments
```

**Purpose:** Create new department (admin only)  
**Auth:** Bearer Token (Required - Org Admin)  
**Organization Filter:** ✅ Created in current organization

**Request:**
```json
{
  "name": "Marketing",
  "description": "Marketing and communications",
  "organizationId": "org-id",
  "isActive": true,
  "settings": {
    "teamSize": 15,
    "budget": 300000
  }
}
```

**Response:** Department object

---

### 4. Update Department
```http
PATCH /departments/{departmentId}
```

**Purpose:** Update department details  
**Auth:** Bearer Token (Required - Org Admin)  
**Organization Filter:** ✅ Must be in same organization

**Request:**
```json
{
  "name": "Marketing & Communications",
  "description": "Updated description",
  "settings": {
    "teamSize": 20
  }
}
```

**Response:** Updated department object

---

### 5. Delete Department
```http
DELETE /departments/{departmentId}
```

**Purpose:** Delete department  
**Auth:** Bearer Token (Required - Org Admin)  
**Organization Filter:** ✅ Must be in same organization

**Response:**
```json
{
  "message": "Department deleted successfully"
}
```

---

### 6. Get Department by Domain
```http
GET /departments/by-domain/{domain}
```

**Purpose:** Find departments by organization domain  
**Auth:** Bearer Token (Required)  
**Organization Filter:** ✅ Automatic

**Response:** Array of departments

---

### 7. Get Department Users
```http
GET /departments/{departmentId}/users
```

**Purpose:** Get all users in department  
**Auth:** Bearer Token (Required)  
**Organization Filter:** ✅ Automatic

**Query Parameters:**
```
?sortBy=firstName:asc
&limit=10
&page=1
```

**Response:** Paginated user list

---

## ❓ Questions

### 1. Get All Questions
```http
GET /questions
```

**Purpose:** Retrieve questions for current organization  
**Auth:** Bearer Token (Required)  
**Organization Filter:** ✅ Automatic (from token)

**Query Parameters:**
```
?isActive=true
&questionType=multiline
&sortBy=order:asc
&limit=10
&page=1
&populate=[{"path":"createdBy","select":"firstName lastName"}]
```

**Response:**
```json
{
  "results": [
    {
      "id": "question-id",
      "organizationId": "org-id",
      "createdBy": {
        "id": "user-id",
        "firstName": "John",
        "lastName": "Doe"
      },
      "title": "What motivates you?",
      "description": "Tell us about your motivation",
      "questionType": "multiline",
      "options": [],
      "isRequired": true,
      "order": 1,
      "isActive": true,
      "isMasterQuestion": false,
      "masterQuestionId": null,
      "metadata": {
        "category": "onboarding",
        "tags": ["motivation", "culture"]
      },
      "myResponses": [
        {
          "id": "response-id",
          "response": "I'm motivated by innovation",
          "createdAt": "2024-05-01T10:00:00.000Z"
        }
      ],
      "reactions": {
        "like": [
          {
            "userId": "user-id",
            "userName": "Jane Smith",
            "userEmail": "jane@example.com",
            "createdAt": "2024-05-01T11:00:00.000Z"
          }
        ]
      },
      "stats": {
        "myResponsesCount": 1,
        "totalReactions": 1,
        "totalResponses": 15,
        "totalViews": 45
      },
      "createdAt": "2024-04-01T00:00:00.000Z"
    }
  ],
  "page": 1,
  "limit": 10,
  "totalPages": 2,
  "totalResults": 15
}
```

**Notes:**
- `myResponses` - Current user's responses only
- `reactions` - All users' reactions with details
- `stats` - Engagement metrics
- Organization-filtered automatically

---

### 2. Get Question by ID
```http
GET /questions/{questionId}
```

**Purpose:** Get single question with responses  
**Auth:** Bearer Token (Required)  
**Organization Filter:** ✅ Must be in same organization

**Response:** Single question object (includes myResponses, reactions, stats)

**Notes:**
- Returns customized version if available
- Falls back to master question if no customization

---

### 3. Create Question
```http
POST /questions
```

**Purpose:** Create new question  
**Auth:** Bearer Token (Required - Admin)  
**Organization Filter:** ✅ Created in current organization

**Request:**
```json
{
  "title": "What are your career goals?",
  "description": "Share your professional aspirations",
  "questionType": "multiline",
  "isRequired": false,
  "order": 5,
  "metadata": {
    "category": "career_development",
    "tags": ["goals", "growth"]
  },
  "isMasterQuestion": false  // True only for superadmin
}
```

**Response:** Question object

**Notes:**
- Superadmin can create master questions (`isMasterQuestion: true`)
- Org admins create organization-specific questions

---

### 4. Update Question
```http
PATCH /questions/{questionId}
```

**Purpose:** Update question (creates customized copy if master)  
**Auth:** Bearer Token (Required - Admin)  
**Organization Filter:** ✅ Organization context

**Request:**
```json
{
  "title": "Updated question title",
  "description": "Updated description",
  "isRequired": true
}
```

**Response:** Question object (customized copy if was master)

**Master Question Behavior:**
- If updating master question as org admin → Creates customized copy
- Original master question remains unchanged
- Customized copy has `masterQuestionId` set

---

### 5. Delete Question
```http
DELETE /questions/{questionId}
```

**Purpose:** Delete question  
**Auth:** Bearer Token (Required - Admin)  
**Organization Filter:** ✅ Must be in same organization

**Response:**
```json
{
  "message": "Question deleted successfully"
}
```

**Notes:**
- Deletes customized copy only (not master)
- Master questions protected from deletion by org admins

---

### 6. Get Question Details
```http
GET /questions/{questionId}/details
```

**Purpose:** Get question with full responses and reactions  
**Auth:** Bearer Token (Required)  
**Organization Filter:** ✅ Automatic

**Response:** Question with complete response and reaction details

---

### 7. Submit Response
```http
POST /questions/{questionId}/respond
```

**Purpose:** Submit answer to question  
**Auth:** Bearer Token (Required)  
**Organization Filter:** ✅ Automatic

**Request:**
```json
{
  "response": "My detailed response here",
  "metadata": {
    "timeTaken": 120  // seconds
  }
}
```

**Response:**
```json
{
  "id": "response-id",
  "questionId": "question-id",
  "userId": "user-id",
  "organizationId": "org-id",
  "response": "My detailed response here",
  "responseText": "My detailed response here",
  "isEdited": false,
  "createdAt": "2024-10-29T12:00:00.000Z"
}
```

---

### 8. Get Question Responses
```http
GET /questions/{questionId}/responses
```

**Purpose:** Get all responses to question  
**Auth:** Bearer Token (Required)  
**Organization Filter:** ✅ Automatic

**Query Parameters:**
```
?sortBy=createdAt:desc
&limit=10
&page=1
```

**Response:** Paginated list of responses

---

### 9. Add Reaction
```http
POST /questions/{questionId}/react
```

**Purpose:** Add reaction to question or response  
**Auth:** Bearer Token (Required)  
**Organization Filter:** ✅ Automatic

**Request:**
```json
{
  "reactionType": "like",  // like, love, helpful, insightful, celebrate, support, curious
  "emoji": "👍",
  "responseId": "response-id"  // Optional: if reacting to response
}
```

**Response:**
```json
{
  "id": "reaction-id",
  "questionId": "question-id",
  "responseId": "response-id",
  "userId": "user-id",
  "reactionType": "like",
  "emoji": "👍",
  "createdAt": "2024-10-29T12:00:00.000Z"
}
```

---

### 10. Remove Reaction
```http
DELETE /questions/{questionId}/react/{reactionId}
```

**Purpose:** Remove own reaction  
**Auth:** Bearer Token (Required)  
**Organization Filter:** ✅ Automatic

**Response:**
```json
{
  "message": "Reaction removed successfully"
}
```

---

### 11. Get All Reactions
```http
GET /questions/reactions
```

**Purpose:** Get all reactions for questions  
**Auth:** Bearer Token (Required)  
**Organization Filter:** ✅ Automatic

**Query Parameters:**
```
?questionId=question-id
&reactionType=like
&sortBy=createdAt:desc
```

**Response:** Paginated list of reactions

---

## 🎉 Celebrations

### 1. Get All Celebrations
```http
GET /celebrations
```

**Purpose:** Retrieve celebrations  
**Auth:** Bearer Token (Required)  
**Organization Filter:** ⚠️ Indirect (via userId)

**Query Parameters:**
```
?userId=user-id
&type=birthday
&isActive=true
&sortBy=date:asc
&limit=10
&page=1
&populate=[{"path":"userId"}]
```

**Response:**
```json
{
  "results": [
    {
      "id": "celebration-id",
      "userId": "user-id",
      "title": "Birthday",
      "type": "birthday",
      "date": "2024-01-15T00:00:00.000Z",
      "isRecurring": true,
      "description": "Annual birthday celebration",
      "reminderDaysBefore": 7,
      "isActive": true,
      "imageUrl": "https://...",
      "createdAt": "2024-01-01T00:00:00.000Z"
    }
  ],
  "page": 1,
  "limit": 10,
  "totalPages": 1,
  "totalResults": 5
}
```

**Notes:**
- Filtered by users in current organization
- Celebration types: birthday, work_anniversary, custom, holiday, achievement

---

### 2. Get Celebration by ID
```http
GET /celebrations/{celebrationId}
```

**Purpose:** Get single celebration  
**Auth:** Bearer Token (Required)  
**Organization Filter:** ⚠️ Indirect (via userId)

**Response:** Single celebration object

---

### 3. Create Celebration
```http
POST /celebrations
```

**Purpose:** Create new celebration  
**Auth:** Bearer Token (Required)  
**Organization Filter:** ⚠️ Indirect

**Request:**
```json
{
  "userId": "user-id",
  "title": "Work Anniversary",
  "type": "work_anniversary",
  "date": "2024-03-01T00:00:00.000Z",
  "isRecurring": true,
  "reminderDaysBefore": 7,
  "description": "5 years with the company"
}
```

**Response:** Celebration object

---

### 4. Update Celebration
```http
PATCH /celebrations/{celebrationId}
```

**Purpose:** Update celebration  
**Auth:** Bearer Token (Required)  
**Organization Filter:** ⚠️ Indirect

**Request:**
```json
{
  "title": "Updated title",
  "reminderDaysBefore": 14
}
```

**Response:** Updated celebration object

---

### 5. Delete Celebration
```http
DELETE /celebrations/{celebrationId}
```

**Purpose:** Delete celebration  
**Auth:** Bearer Token (Required)  
**Organization Filter:** ⚠️ Indirect

**Response:**
```json
{
  "message": "Celebration deleted successfully"
}
```

---

## 📅 DOB Alerts

### 1. Get All DOB Alerts
```http
GET /dob-alerts
```

**Purpose:** Retrieve birthday alerts  
**Auth:** Bearer Token (Required)  
**Organization Filter:** ⚠️ Indirect (via userId)

**Query Parameters:**
```
?userId=user-id
&alertType=birthday_reminder
&isActive=true
&sortBy=alertDate:asc
&limit=10
&page=1
```

**Response:**
```json
{
  "results": [
    {
      "id": "alert-id",
      "userId": "user-id",
      "alertType": "birthday_reminder",
      "alertDate": "2024-01-15T00:00:00.000Z",
      "message": "Happy Birthday!",
      "daysBefore": 7,
      "isActive": true,
      "notificationSent": false,
      "createdAt": "2024-01-01T00:00:00.000Z"
    }
  ],
  "page": 1,
  "limit": 10,
  "totalPages": 1,
  "totalResults": 3
}
```

**Alert Types:**
- `birthday_reminder` - Birthday notifications
- `age_milestone` - Age milestones (30, 40, 50, etc.)
- `custom` - Custom DOB alerts

---

### 2. Get DOB Alert by ID
```http
GET /dob-alerts/{alertId}
```

**Purpose:** Get single DOB alert  
**Auth:** Bearer Token (Required)  
**Organization Filter:** ⚠️ Indirect

**Response:** Single alert object

---

### 3. Create DOB Alert
```http
POST /dob-alerts
```

**Purpose:** Create birthday alert  
**Auth:** Bearer Token (Required)  
**Organization Filter:** ⚠️ Indirect

**Request:**
```json
{
  "userId": "user-id",
  "alertType": "birthday_reminder",
  "alertDate": "2024-01-15T00:00:00.000Z",
  "message": "Don't forget John's birthday!",
  "daysBefore": 7,
  "isActive": true
}
```

**Response:** DOB alert object

---

### 4. Update DOB Alert
```http
PATCH /dob-alerts/{alertId}
```

**Purpose:** Update DOB alert  
**Auth:** Bearer Token (Required)  
**Organization Filter:** ⚠️ Indirect

**Request:**
```json
{
  "daysBefore": 14,
  "message": "Updated reminder message"
}
```

**Response:** Updated alert object

---

### 5. Delete DOB Alert
```http
DELETE /dob-alerts/{alertId}
```

**Purpose:** Delete DOB alert  
**Auth:** Bearer Token (Required)  
**Organization Filter:** ⚠️ Indirect

**Response:**
```json
{
  "message": "DOB alert deleted successfully"
}
```

---

## 📋 Audit Logs

### 1. Get All Audit Logs
```http
GET /audit-logs
```

**Purpose:** Retrieve system activity logs  
**Auth:** Bearer Token (Required - Admin)  
**Organization Filter:** ⚠️ Indirect (via userId)

**Query Parameters:**
```
?userId=user-id
&action=login
&resource=user
&success=true
&sortBy=createdAt:desc
&limit=10
&page=1
```

**Response:**
```json
{
  "results": [
    {
      "id": "log-id",
      "userId": "user-id",
      "action": "login",
      "resource": "user",
      "resourceId": "user-id",
      "success": true,
      "metadata": {
        "ipAddress": "192.168.1.1",
        "userAgent": "Mozilla/5.0..."
      },
      "createdAt": "2024-10-29T10:30:00.000Z"
    }
  ],
  "page": 1,
  "limit": 10,
  "totalPages": 5,
  "totalResults": 50
}
```

**Actions:**
- `create`, `read`, `update`, `delete`
- `login`, `logout`
- `password_change`, `password_reset`
- `email_verification`
- `otp_send`, `otp_verify`

---

### 2. Get Audit Log by ID
```http
GET /audit-logs/{logId}
```

**Purpose:** Get single audit log  
**Auth:** Bearer Token (Required - Admin)  
**Organization Filter:** ⚠️ Indirect

**Response:** Single audit log object

---

### 3. Get User Audit Logs
```http
GET /audit-logs/user/{userId}
```

**Purpose:** Get all logs for specific user  
**Auth:** Bearer Token (Required - Admin)  
**Organization Filter:** ⚠️ Indirect

**Query Parameters:**
```
?action=login
&sortBy=createdAt:desc
&limit=10
&page=1
```

**Response:** Paginated audit logs

---

### 4. Get User Login History
```http
GET /audit-logs/user/{userId}/login-history
```

**Purpose:** Get login history for user  
**Auth:** Bearer Token (Required - Admin)  
**Organization Filter:** ⚠️ Indirect

**Response:** Paginated login logs with IP addresses

---

### 5. Get Resource Audit Logs
```http
GET /audit-logs/resource/{resource}/{resourceId}
```

**Purpose:** Get logs for specific resource  
**Auth:** Bearer Token (Required - Admin)  
**Organization Filter:** ⚠️ Indirect

**Query Parameters:**
```
?sortBy=createdAt:desc
&limit=10
&page=1
```

**Response:** Paginated logs for resource (e.g., all changes to a question)

---

### 6. Get Failed Actions
```http
GET /audit-logs/failed
```

**Purpose:** Get all failed actions (security monitoring)  
**Auth:** Bearer Token (Required - Admin)  
**Organization Filter:** ⚠️ Indirect

**Query Parameters:**
```
?action=login
&sortBy=createdAt:desc
&limit=10
&page=1
```

**Response:** Paginated failed action logs

---

## 🔑 Legend

### Organization Filtering

| Symbol | Meaning |
|--------|---------|
| ✅ **Automatic** | organizationId from JWT token, no manual parameter needed |
| ✅ **Verified** | Access to resource verified against user's organizations |
| ⚠️ **Indirect** | Filtered via related entity (e.g., userId) that is org-filtered |
| N/A | Not applicable (public or superadmin-only) |

### Access Roles

| Role | Description |
|------|-------------|
| **Superadmin** | Full system access, bypasses org filtering |
| **Org Admin** | Manage users, departments, questions in their organization |
| **User** | Basic access, view and respond to questions |

---

## 📖 Additional Resources

- **Swagger UI:** http://localhost:3000/v1/docs
- **Complete Update:** [SWAGGER_COMPLETE_UPDATE_2024.md](SWAGGER_COMPLETE_UPDATE_2024.md)
- **Quick Reference:** [SWAGGER_QUICK_REFERENCE.md](SWAGGER_QUICK_REFERENCE.md)
- **Auth Guide:** [AUTHENTICATION_AND_ORGANIZATION_GUIDE.md](AUTHENTICATION_AND_ORGANIZATION_GUIDE.md)
- **Password Guide:** [PASSWORD_MANAGEMENT_GUIDE.md](PASSWORD_MANAGEMENT_GUIDE.md)

---

**Last Updated:** October 29, 2024  
**Total Endpoints:** 58  
**Version:** 2.0.0

