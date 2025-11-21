# HRIS Field Mapping API Reference

## API Endpoints

### 1. Get Field Mappings
**Endpoint:** `GET /v1/hr-integrations/integrations/:integrationId/field-mappings`

**Response Structure:**
```json
{
  "status": "success",
  "data": {
    "standardFields": [
      {
        "value": "email",
        "label": "Email",
        "required": true,
        "mappedTo": "user_email"  // Current mapping (empty string if not mapped)
      },
      {
        "value": "fullName",
        "label": "Full Name",
        "required": false,
        "mappedTo": "employee_name"
      },
      // ... more fields
    ],
    "currentMappings": {
      "email": "user_email",
      "fullName": "employee_name",
      "jobTitle": "designation",
      "department": "orgunit",
      "location": "office",
      "dateOfHire": "onboarding_date",
      "dob": "date_of_birth"
    },
    "currentPlatform": {
      "platform": "bamboohr",
      "platformName": "Bamboohr",
      "isConnected": true,
      "availableFields": [
        { "value": "user_email", "label": "User Email" },
        { "value": "employee_name", "label": "Employee Name" },
        // ... more fields
      ]
    },
    "connectedPlatforms": [...],
    "allPlatforms": [...]
  }
}
```

### 2. Update Field Mappings
**Endpoint:** `PATCH /v1/hr-integrations/integrations/:integrationId/field-mappings`

**Request Body:**
```json
{
  "fieldMappings": {
    "email": "user_email",
    "firstName": "first_name",
    "lastName": "last_name",
    "fullName": "employee_name",
    "jobTitle": "designation",
    "department": "orgunit",
    "location": "office",
    "dateOfHire": "onboarding_date",
    "dob": "date_of_birth"
  }
}
```

## ⚠️ Field Name Mismatch

### Frontend Expects (WRONG):
```javascript
{
  employeeName: "Full Name",  // ❌ WRONG - API uses "fullName"
  email: "Email",
  jobTitle: "Designation",
  department: "Orgunit",
  location: "Office",
  hireDate: "Onboarding Date",  // ❌ WRONG - API uses "dateOfHire"
  birthday: "DOB"  // ❌ WRONG - API uses "dob"
}
```

### API Actually Uses (CORRECT):
```javascript
{
  fullName: "Full Name",      // ✅ CORRECT
  email: "Email",
  jobTitle: "Designation",
  department: "Orgunit",
  location: "Office",
  dateOfHire: "Onboarding Date",  // ✅ CORRECT
  dob: "Date of Birth"  // ✅ CORRECT
}
```

## Correct BridgeBond Field Names

The API uses these field names (from `getStandardBridgeBondFields()`):

| Frontend Expects | API Uses | Label |
|-----------------|----------|-------|
| `employeeName` | `fullName` | Full Name |
| `hireDate` | `dateOfHire` | Onboarding Date |
| `birthday` | `dob` | Date of Birth |
| ✅ `email` | `email` | Email |
| ✅ `jobTitle` | `jobTitle` | Designation |
| ✅ `department` | `department` | Org Unit |
| ✅ `location` | `location` | Office |

## Complete List of BridgeBond Fields

```javascript
[
  { value: 'email', label: 'Email', required: true },
  { value: 'firstName', label: 'First Name', required: true },
  { value: 'lastName', label: 'Last Name', required: true },
  { value: 'fullName', label: 'Full Name', required: false },  // ← Use this, not "employeeName"
  { value: 'jobTitle', label: 'Designation', required: false },
  { value: 'department', label: 'Org Unit', required: false },
  { value: 'location', label: 'Office', required: false },
  { value: 'dateOfHire', label: 'Onboarding Date', required: false },  // ← Use this, not "hireDate"
  { value: 'dob', label: 'Date of Birth', required: false },  // ← Use this, not "birthday"
  { value: 'phone', label: 'Phone', required: false },
  { value: 'employeeId', label: 'Employee ID', required: false },
  { value: 'manager', label: 'Manager', required: false },
  { value: 'address', label: 'Address', required: false },
  { value: 'city', label: 'City', required: false },
  { value: 'state', label: 'State', required: false },
  { value: 'zipCode', label: 'Zip Code', required: false },
  { value: 'country', label: 'Country', required: false },
  { value: 'gender', label: 'Gender', required: false },
  { value: 'imageUrl', label: 'Profile Image', required: false },
]
```

## Frontend Fix Required

Update your frontend `BRIDGEBOND_FIELDS` constant to use:

```javascript
// ❌ WRONG (Current Frontend)
const BRIDGEBOND_FIELDS = {
  employeeName: "Full Name",
  hireDate: "Onboarding Date",
  birthday: "DOB"
};

// ✅ CORRECT (Should be)
const BRIDGEBOND_FIELDS = {
  fullName: "Full Name",        // Changed from "employeeName"
  dateOfHire: "Onboarding Date", // Changed from "hireDate"
  dob: "Date of Birth"          // Changed from "birthday"
};
```

## API Response Structure

When you call `GET /v1/hr-integrations/integrations/:id/field-mappings`, you get:

```json
{
  "status": "success",
  "data": {
    "standardFields": [
      {
        "value": "fullName",      // ← Use this field name
        "label": "Full Name",
        "required": false,
        "mappedTo": "employee_name"  // Current mapping
      },
      {
        "value": "dateOfHire",    // ← Use this field name
        "label": "Onboarding Date",
        "required": false,
        "mappedTo": "onboarding_date"
      },
      {
        "value": "dob",           // ← Use this field name
        "label": "Date of Birth",
        "required": false,
        "mappedTo": "date_of_birth"
      }
    ],
    "currentMappings": {
      "fullName": "employee_name",
      "dateOfHire": "onboarding_date",
      "dob": "date_of_birth"
    }
  }
}
```

## Update Request Format

When updating field mappings, use the correct field names:

```json
{
  "fieldMappings": {
    "fullName": "employee_name",      // ✅ Correct
    "dateOfHire": "onboarding_date",  // ✅ Correct
    "dob": "date_of_birth"            // ✅ Correct
  }
}
```

NOT:
```json
{
  "fieldMappings": {
    "employeeName": "employee_name",  // ❌ Wrong field name
    "hireDate": "onboarding_date",    // ❌ Wrong field name
    "birthday": "date_of_birth"        // ❌ Wrong field name
  }
}
```

