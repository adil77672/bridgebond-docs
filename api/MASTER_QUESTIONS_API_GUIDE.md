# Master Questions API Usage Guide

## Quick Reference

### 1. Create Master Question (Superadmin Only)
```http
POST /v1/questions
Authorization: Bearer <superadmin-token>
Content-Type: application/json

{
  "title": "How satisfied are you with your work-life balance?",
  "description": "Rate your satisfaction on a scale of 1-10",
  "questionType": "rating",
  "isMasterQuestion": true,
  "isActive": true
}
```

**Response**:
```json
{
  "_id": "master123",
  "title": "How satisfied are you with your work-life balance?",
  "isMasterQuestion": true,
  "masterQuestionId": null,
  "organizationId": null,
  "assignedOrganizations": [],
  "createdBy": "superadmin-id"
}
```

---

### 2. Assign Master to Organizations (Superadmin Only)
```http
POST /v1/questions/master123/assign
Authorization: Bearer <superadmin-token>
Content-Type: application/json

{
  "organizationIds": [
    "org-abc123",
    "org-def456",
    "org-ghi789"
  ]
}
```

**Response**:
```json
{
  "_id": "master123",
  "title": "How satisfied are you with your work-life balance?",
  "isMasterQuestion": true,
  "assignedOrganizations": [
    "org-abc123",
    "org-def456",
    "org-ghi789"
  ]
}
```

---

### 3. Get All Master Questions (Superadmin Only)
```http
GET /v1/questions/master
Authorization: Bearer <superadmin-token>
```

**Query Parameters**:
- `isActive` (boolean): Filter by active status
- `questionType` (string): Filter by question type
- `page` (number): Page number for pagination
- `limit` (number): Items per page
- `sortBy` (string): Sort field and order (e.g., `createdAt:desc`)

**Response**:
```json
{
  "results": [
    {
      "_id": "master123",
      "title": "How satisfied are you with your work-life balance?",
      "isMasterQuestion": true,
      "assignedOrganizations": ["org-abc123", "org-def456"]
    }
  ],
  "page": 1,
  "limit": 10,
  "totalPages": 1,
  "totalResults": 1
}
```

---

### 4. Get Questions (Organization Admin/User)
```http
GET /v1/questions
Authorization: Bearer <org-admin-token>
```

**What You'll See**:
- ✅ Organization-specific questions
- ✅ Your customized versions of master questions
- ✅ Master questions assigned to your org (if not customized)
- ❌ Master questions for other organizations
- ❌ Other organizations' customized versions

**Response**:
```json
{
  "results": [
    {
      "_id": "custom456",
      "title": "Our Team's Work-Life Balance Check",
      "organizationId": "org-abc123",
      "isMasterQuestion": false,
      "masterQuestionId": "master123"
    },
    {
      "_id": "master789",
      "title": "Weekly Team Satisfaction",
      "isMasterQuestion": true,
      "assignedOrganizations": ["org-abc123", "org-def456"]
    }
  ]
}
```

---

### 5. Get Question by ID
```http
GET /v1/questions/master123
Authorization: Bearer <org-admin-token>
```

**Behavior**:
- If you request a **master question** and your org has a **customized version**, you'll receive the **customized version** automatically
- If no customized version exists, you'll receive the master question

**Example Response (Customized Version)**:
```json
{
  "_id": "custom456",
  "title": "Our Team's Work-Life Balance Check",
  "organizationId": "org-abc123",
  "isMasterQuestion": false,
  "masterQuestionId": {
    "_id": "master123",
    "title": "How satisfied are you with your work-life balance?"
  },
  "viewsCount": 45,
  "createdBy": {
    "_id": "admin-id",
    "firstName": "John",
    "lastName": "Doe"
  }
}
```

---

### 6. Update Master Question → Creates Customized Copy
```http
PATCH /v1/questions/master123
Authorization: Bearer <org-admin-token>
Content-Type: application/json

{
  "title": "Custom Title for Our Organization",
  "description": "Modified description",
  "organizationId": "org-abc123"
}
```

**What Happens**:
1. System checks if you're updating a master question
2. If yes, creates a new customized copy for your organization
3. If a customized copy already exists, updates that instead
4. Master question remains unchanged

**Response**:
```json
{
  "_id": "custom456",
  "title": "Custom Title for Our Organization",
  "description": "Modified description",
  "organizationId": "org-abc123",
  "isMasterQuestion": false,
  "masterQuestionId": "master123"
}
```

---

### 7. Update Customized Question (Direct Update)
```http
PATCH /v1/questions/custom456
Authorization: Bearer <org-admin-token>
Content-Type: application/json

{
  "title": "Updated Custom Title",
  "isActive": false
}
```

**Response**:
```json
{
  "_id": "custom456",
  "title": "Updated Custom Title",
  "isActive": false,
  "organizationId": "org-abc123",
  "masterQuestionId": "master123"
}
```

---

### 8. Delete Customized Question (Org Admin)
```http
DELETE /v1/questions/custom456
Authorization: Bearer <org-admin-token>
```

**What Happens**:
1. Only your organization's customized copy is deleted
2. Master question remains active
3. Other organizations' versions are unaffected
4. After deletion, your org will see the master question again

**Response**: `204 No Content`

---

### 9. Delete Master Question (Superadmin Only)
```http
DELETE /v1/questions/master123
Authorization: Bearer <superadmin-token>
```

**What Happens**:
1. Master question is soft-deleted
2. **All customized versions** across all organizations are also soft-deleted
3. Use with caution!

**Response**: `204 No Content`

---

### 10. Unassign Master from Organizations (Superadmin Only)
```http
POST /v1/questions/master123/unassign
Authorization: Bearer <superadmin-token>
Content-Type: application/json

{
  "organizationIds": [
    "org-abc123"
  ]
}
```

**Response**:
```json
{
  "_id": "master123",
  "title": "How satisfied are you with your work-life balance?",
  "assignedOrganizations": [
    "org-def456",
    "org-ghi789"
  ]
}
```

---

### 11. Get Customized Versions (Superadmin Only)
```http
GET /v1/questions/master123/versions
Authorization: Bearer <superadmin-token>
```

**Response**:
```json
[
  {
    "_id": "custom456",
    "title": "Custom Title - Org A",
    "organizationId": "org-abc123",
    "masterQuestionId": "master123"
  },
  {
    "_id": "custom789",
    "title": "Custom Title - Org B",
    "organizationId": "org-def456",
    "masterQuestionId": "master123"
  }
]
```

---

### 12. Get Question Details with Responses and Reactions
```http
GET /v1/questions/master123/details
Authorization: Bearer <user-token>
```

**Query Parameters**:
- `responseLimit` (number): Limit number of responses

**Response**:
```json
{
  "question": {
    "_id": "custom456",
    "title": "Our Team's Work-Life Balance Check",
    "organizationId": "org-abc123"
  },
  
  "myResponses": [
    {
      "_id": "resp123",
      "response": "I'm very satisfied - 9/10",
      "userId": {
        "_id": "user-id",
        "firstName": "John",
        "lastName": "Doe"
      },
      "reactions": {
        "like": [
          {
            "userId": "user2-id",
            "userName": "Jane Smith",
            "userEmail": "jane@example.com",
            "createdAt": "2025-10-28T10:00:00Z"
          }
        ],
        "love": [
          {
            "userId": "user3-id",
            "userName": "Bob Wilson",
            "userEmail": "bob@example.com",
            "createdAt": "2025-10-28T11:00:00Z"
          }
        ]
      }
    }
  ],
  
  "reactions": {
    "like": [
      {
        "userId": "user4-id",
        "userName": "Alice Johnson",
        "userEmail": "alice@example.com",
        "createdAt": "2025-10-28T09:00:00Z"
      },
      {
        "userId": "user5-id",
        "userName": "Charlie Brown",
        "userEmail": "charlie@example.com",
        "createdAt": "2025-10-28T09:30:00Z"
      }
    ],
    "love": [
      {
        "userId": "user6-id",
        "userName": "David Lee",
        "userEmail": "david@example.com",
        "createdAt": "2025-10-28T10:15:00Z"
      }
    ]
  },
  
  "stats": {
    "myResponsesCount": 1,
    "totalQuestionReactions": 3,
    "totalResponseReactions": 2
  }
}
```

**Key Points**:
- `myResponses`: Only YOUR responses (not all users)
- `reactions`: ALL users' reactions to the question (key-value pairs by reaction type)
- Each response includes reactions from all users

---

## Common Scenarios

### Scenario 1: Rolling Out a Company-Wide Question
```bash
# 1. Superadmin creates master question
POST /v1/questions
{ "title": "Q4 Goals", "isMasterQuestion": true }

# 2. Assign to all organizations
POST /v1/questions/{id}/assign
{ "organizationIds": ["org1", "org2", "org3"] }

# 3. Each org sees the question and can customize if needed
```

### Scenario 2: Organization Customizes Then Reverts
```bash
# 1. Org admin customizes
PATCH /v1/questions/{masterId}
{ "title": "Custom Q4 Goals for Engineering" }

# 2. Later, they want the original
DELETE /v1/questions/{customizedId}

# 3. Now they see the master question again
GET /v1/questions/{masterId}
```

### Scenario 3: Superadmin Updates Master
```bash
# Superadmin can directly update master
PATCH /v1/questions/{masterId}
{ "description": "Updated description" }

# This ONLY updates the master, not customized versions
```

---

## Error Responses

### 403 Forbidden - Org Admin Tries to Delete Master
```json
{
  "code": 403,
  "message": "Only superadmin can delete master questions"
}
```

### 404 Not Found - Question Doesn't Exist
```json
{
  "code": 404,
  "message": "Question not found"
}
```

### 403 Forbidden - No Access to Question
```json
{
  "code": 403,
  "message": "You do not have access to this question"
}
```

---

## Tips

1. **Always pass `organizationId`** when org admins update master questions to trigger customization
2. **Use GET /questions/master** to see all master questions and their assignments (superadmin only)
3. **Check `masterQuestionId`** field to identify customized versions
4. **Reactions are grouped by type** - makes it easy to display counts and user lists
5. **Soft delete** means data is preserved - can be recovered if needed

