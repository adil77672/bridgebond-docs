# HR Integration Sync Guide

## How to Sync HRIS Data to Your Database

This guide explains how the HR Integration sync process works and how to use it.

---

## Overview

The sync process fetches employee data from your HR platform (ADP, BambooHR, or Workday) and syncs it to your database. It automatically:

- ✅ Creates new users if they don't exist
- ✅ Updates existing users if they're already in the system
- ✅ Prevents duplicates based on email and external ID
- ✅ Creates departments automatically if they don't exist
- ✅ Maps platform fields to your system fields using field mappings
- ✅ Tracks sync statistics

---

## Step-by-Step Process

### 1. **Connect to HR Platform**

First, connect your organization to an HR platform:

```http
POST /v1/hr-integrations/organizations/{organizationId}/connect
Authorization: Bearer <your-token>
Content-Type: application/json

{
  "platform": "adp",  // or "bamboohr" or "workday"
  "credentials": {
    // Platform-specific credentials
    // See examples below
  }
}
```

**Example for ADP:**
```json
{
  "platform": "adp",
  "credentials": {
    "clientId": "your-client-id",
    "clientSecret": "your-client-secret",
    "apiKey": "your-api-key",
    "baseUrl": "https://api.adp.com"
  }
}
```

**Example for BambooHR:**
```json
{
  "platform": "bamboohr",
  "credentials": {
    "apiKey": "your-api-key",
    "subdomain": "your-company"
  }
}
```

**Example for Workday:**
```json
{
  "platform": "workday",
  "credentials": {
    "tenant": "your-tenant",
    "username": "your-username",
    "password": "your-password",
    "baseUrl": "https://your-tenant.workday.com/ccx/service"
  }
}
```

---

### 2. **Configure Field Mappings (Optional but Recommended)**

Map platform fields to your system fields. This tells the system which platform field corresponds to which system field.

```http
PATCH /v1/hr-integrations/integrations/{integrationId}/field-mappings
Authorization: Bearer <your-token>
Content-Type: application/json

{
  "fieldMappings": {
    "email": "user_email",        // Platform's "user_email" → System's "email"
    "firstName": "first_name",     // Platform's "first_name" → System's "firstName"
    "lastName": "last_name",       // Platform's "last_name" → System's "lastName"
    "jobTitle": "position",        // Platform's "position" → System's "jobTitle"
    "dateOfHire": "hire_date",     // Platform's "hire_date" → System's "dateOfHire"
    "department": "department_name", // Platform's "department_name" → System's "department"
    "dob": "date_of_birth",        // Platform's "date_of_birth" → System's "dob"
    "gender": "sex"                 // Platform's "sex" → System's "gender"
  }
}
```

**Note:** If you don't configure field mappings, the system will use default mappings (common field names like `email`, `firstName`, etc.).

---

### 3. **Sync Users**

Once connected and field mappings are configured, sync users:

```http
POST /v1/hr-integrations/integrations/{integrationId}/sync
Authorization: Bearer <your-token>
```

**Response:**
```json
{
  "status": "success",
  "data": {
    "integration": {
      "id": "...",
      "platform": "adp",
      "lastSyncAt": "2024-10-28T10:30:00.000Z",
      "lastSyncStatus": "success",
      "syncStats": {
        "totalUsers": 100,
        "syncedUsers": 95,
        "duplicateUsers": 3,
        "failedUsers": 2
      }
    },
    "stats": {
      "totalUsers": 100,
      "syncedUsers": 95,
      "duplicateUsers": 3,
      "failedUsers": 2
    }
  },
  "message": "Sync completed: 95 users synced, 3 duplicates, 2 failed"
}
```

---

## How Sync Works Internally

### Step 1: Fetch Employees from Platform
```javascript
// Platform service fetches all employees
const platformEmployees = await platformService.fetchEmployees();
// Returns array of employee objects from the HR platform
```

### Step 2: Process Each Employee
For each employee, the system:

1. **Map Fields** - Maps platform fields to system fields using field mappings
   ```javascript
   // Example: Platform has "user_email", system needs "email"
   mappedData.email = employee.user_email; // Using field mapping
   ```

2. **Check for Duplicates** - Prevents duplicate users
   - **By External ID**: Checks if employee ID already exists in `externalUserIds` map
   - **By Email**: Checks if email already exists in database

3. **Create or Update User**
   - **New User**: Creates new user with:
     - Email (required)
     - Temporary password (user must reset)
     - Organization membership
     - All mapped fields (firstName, lastName, jobTitle, etc.)
     - Department (created automatically if doesn't exist)
   
   - **Existing User**: Updates existing user's organization membership
     - Updates profile fields (name, job title, etc.)
     - Updates department
     - Marks membership as active

4. **Store External ID Mapping** - Maps platform employee ID to system user ID
   ```javascript
   integration.externalUserIds.set(externalId, user._id.toString());
   ```

### Step 3: Update Sync Statistics
After processing all employees:
- Updates `lastSyncAt` timestamp
- Updates `lastSyncStatus` (success, failed, or partial)
- Updates `syncStats` with counts

---

## Sync Statistics Explained

| Field | Description |
|-------|-------------|
| `totalUsers` | Total number of employees found in the HR platform |
| `syncedUsers` | Number of employees successfully synced (created or updated) |
| `duplicateUsers` | Number of employees skipped because they already exist |
| `failedUsers` | Number of employees that failed to sync (missing email, validation errors, etc.) |

---

## Duplicate Prevention

The system prevents duplicates in two ways:

### 1. External ID Mapping
- Stores mapping: `platformEmployeeId → systemUserId`
- On subsequent syncs, checks this mapping first
- If found, updates existing user instead of creating new one

### 2. Email Deduplication
- Checks if email already exists in database
- If user exists but not in this organization, adds organization membership
- If user already in organization, skips (counts as duplicate)

---

## Automatic Department Creation

If an employee has a department field, the system:
1. Checks if department exists in the organization
2. If exists, links user to existing department
3. If not exists, creates new department automatically
4. Links user to the department

---

## Field Mapping Examples

### Default Mappings (if not configured)
The system tries these common field names automatically:
- `email`: `email`, `user_email`, `workEmail`, `emailAddress`
- `firstName`: `firstName`, `first_name`, `givenName`, `firstname`
- `lastName`: `lastName`, `last_name`, `familyName`, `surname`, `lastname`
- `jobTitle`: `jobTitle`, `job_title`, `position`, `title`
- `dateOfHire`: `dateOfHire`, `date_of_hire`, `hireDate`, `startDate`
- `department`: `department`, `departmentName`, `dept`, `division`
- `dob`: `dob`, `dateOfBirth`, `birthDate`, `date_of_birth`
- `gender`: `gender`, `sex`

### Custom Mappings
If your platform uses different field names, configure them:
```json
{
  "fieldMappings": {
    "email": "work_email_address",      // Custom field name
    "firstName": "legal_first_name",     // Custom field name
    "lastName": "legal_last_name",       // Custom field name
    "jobTitle": "job_title_name",        // Custom field name
    "department": "department_name_text" // Custom field name
  }
}
```

---

## Error Handling

### Missing Email
- Employee is skipped
- Counted in `failedUsers`
- Logged as warning

### Validation Errors
- Employee is skipped
- Counted in `failedUsers`
- Error logged with employee details

### Platform Connection Errors
- Sync fails completely
- `lastSyncStatus` set to `failed`
- `lastSyncError` stores error message

---

## Best Practices

1. **Configure Field Mappings First**
   - Set up field mappings before syncing
   - This ensures accurate data mapping

2. **Sync Regularly**
   - Run sync periodically to keep data up-to-date
   - Consider setting up automated sync (cron job)

3. **Monitor Sync Statistics**
   - Check `syncStats` after each sync
   - Investigate `failedUsers` if count is high
   - Review `duplicateUsers` to ensure expected behavior

4. **Handle New Users**
   - New users get temporary passwords
   - Users should reset password on first login
   - Consider sending welcome email with password reset link

---

## API Endpoints Summary

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/hr-integrations/organizations/{orgId}/connect` | POST | Connect to HR platform |
| `/hr-integrations/organizations/{orgId}/integrations` | GET | List all integrations |
| `/hr-integrations/integrations/{integrationId}` | GET | Get integration details |
| `/hr-integrations/integrations/{integrationId}/field-mappings` | PATCH | Update field mappings |
| `/hr-integrations/integrations/{integrationId}/sync` | POST | **Sync users** |
| `/hr-integrations/integrations/{integrationId}/disconnect` | POST | Disconnect platform |
| `/hr-integrations/integrations/{integrationId}` | DELETE | Delete integration |

---

## Example Complete Flow

```bash
# 1. Connect to ADP
curl -X POST http://localhost:3000/v1/hr-integrations/organizations/ORG123/connect \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "platform": "adp",
    "credentials": {
      "clientId": "client-id",
      "clientSecret": "client-secret",
      "apiKey": "api-key"
    }
  }'

# Response: { "status": "success", "data": { "id": "INTEGRATION123", ... } }

# 2. Configure field mappings
curl -X PATCH http://localhost:3000/v1/hr-integrations/integrations/INTEGRATION123/field-mappings \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "fieldMappings": {
      "email": "user_email",
      "firstName": "first_name",
      "lastName": "last_name"
    }
  }'

# 3. Sync users
curl -X POST http://localhost:3000/v1/hr-integrations/integrations/INTEGRATION123/sync \
  -H "Authorization: Bearer TOKEN"

# Response: {
#   "status": "success",
#   "data": {
#     "stats": {
#       "totalUsers": 100,
#       "syncedUsers": 95,
#       "duplicateUsers": 3,
#       "failedUsers": 2
#     }
#   },
#   "message": "Sync completed: 95 users synced, 3 duplicates, 2 failed"
# }
```

---

## Troubleshooting

### Sync Returns 0 Users
- Check platform credentials are correct
- Verify platform connection is successful
- Check platform API endpoints are accessible

### High Failed Users Count
- Check field mappings are configured correctly
- Verify required fields (email) exist in platform data
- Check platform data format matches expected format

### Duplicates Not Detected
- Ensure external ID field is present in platform data
- Check external ID field name matches platform's actual field name
- Verify external ID mapping is being stored correctly

---

## Next Steps

After syncing, users will:
1. Have accounts created in your system
2. Be linked to the organization
3. Need to reset password (temporary password generated)
4. Receive email verification (if email verification is enabled)

Consider:
- Setting up automated sync (cron job)
- Sending welcome emails to new users
- Configuring password reset flow for synced users
- Setting up webhooks for real-time updates (future enhancement)

