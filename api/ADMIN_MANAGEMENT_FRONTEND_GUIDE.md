# Admin Management Frontend Integration Guide

## Overview
This guide provides frontend developers with all the information needed to build an admin management module where super admins can create and manage organization admins.

## Key Features & Requirements

### 1. Create Organization Admin (Without Organization) ✅
**Scenario:** Super admin needs to create an `org_admin` **before organizations exist** in the system.

- Super admin can create an `org_admin` user **without** requiring an organization at creation time
- `organizationId` is **completely optional** when `role` is `org_admin`
- Organization can be assigned later via update
- This allows creating admins before organizations exist

**Example Use Case:**
```
1. System is new, no organizations created yet
2. Super admin creates org_admin: "John Doe" (no organizationId)
3. Later, organizations are created
4. Super admin assigns organizations to "John Doe" via update
```

### 2. Assign Organizations at Update Time ✅
**Scenario:** Super admin assigns organizations to admins after they are created.

- Super admin can assign one or multiple organizations to an admin during update
- Admins can be assigned to organizations after creation
- Use `PATCH /v1/users/:userId` with `organizationMemberships` array

**Example Use Case:**
```
1. Admin "John Doe" created without organization
2. Super admin later creates "TechCorp Inc" organization
3. Super admin updates "John Doe" and assigns "TechCorp Inc"
4. "John Doe" is now admin of "TechCorp Inc"
```

### 3. Advanced Filtering for Super Admin ✅
**Scenario:** Super admin needs to filter and view different types of users.

**Filter Options:**
1. **All Admins** - Show all organization admins
   - `GET /v1/users?role=org_admin`

2. **Organization-Based Admins** - Show admins assigned to a specific organization
   - `GET /v1/users?role=org_admin&organizationId=org-id-123`

3. **Organization-Based Users** - Show all users (admins + regular users) in a specific organization
   - `GET /v1/users?organizationId=org-id-123`

4. **Unassigned Admins** - Show admins without any organization assignment
   - `GET /v1/users?role=org_admin` (then filter frontend: `organizationMemberships.length === 0`)

5. **All Users by Role** - Filter by any role
   - `GET /v1/users?role=user` - All regular users
   - `GET /v1/users?role=org_admin` - All org admins
   - `GET /v1/users?role=superadmin` - All super admins (superadmin only)

6. **Combined Filters** - Combine role + organization + status
   - `GET /v1/users?role=org_admin&organizationId=org-id&isActive=true`

---

## API Endpoints

### 1. Create Admin (POST `/v1/users`)

**Purpose:** Create a new organization admin (or regular user)

**Request Body:**
```json
{
  "firstName": "John",
  "lastName": "Doe",
  "email": "john.doe@example.com",
  "password": "SecurePass123",
  "dob": "1990-05-15",
  "dateOfHire": "2024-01-15",
  "jobTitle": "Administrator",
  "role": "org_admin",  // ← Can be "user", "org_admin", or "superadmin"
  "organizationId": null,  // ← OPTIONAL for org_admin (can be null)
  "isEmailVerified": false,
  "isActive": true
}
```

**Key Points:**
- `organizationId` is **optional** when `role` is `org_admin`
- If `organizationId` is provided, admin is immediately assigned to that organization
- If `organizationId` is `null`, admin is created without organization assignment
- `department` is also optional for `org_admin`
- **Email Verification:** Verification email is **automatically sent** by the backend when creating a new admin
  - Admin receives OTP code via email (expires in 10 minutes)
  - Admin must login and verify their email using the OTP code
  - Response includes `message` field if verification email was sent successfully

**Response:**
```json
{
  "id": "user-id",
  "email": "john.doe@example.com",
  "role": "org_admin",
  "organizationMemberships": [],  // ← Empty if no org assigned
  "isActive": true,
  "isEmailVerified": false,
  "createdAt": "2024-01-15T10:00:00.000Z",
  "message": "User created successfully. Verification email sent to john.doe@example.com. The user must login and verify their email."
}
```

**Note:** The `message` field is included in the response if verification email was sent successfully. If email sending fails, the user is still created but no message is included.

---

### 2. Get All Users/Admins (GET `/v1/users`)

**Purpose:** Retrieve list of users with filtering capabilities

**Query Parameters:**
```
?role=org_admin              // Filter by role
&organizationId=org-id      // Filter by organization
&isActive=true              // Filter by active status
&page=1                     // Pagination
&limit=10                    // Items per page
&sortBy=createdAt:desc      // Sorting
&populate=[{"path":"organizationMemberships.organizationId","select":"name"}]
```

**Filtering Examples:**

**Get all organization admins:**
```
GET /v1/users?role=org_admin
```

**Get admins assigned to specific organization:**
```
GET /v1/users?role=org_admin&organizationId=org-id-123
```

**Get all users in a specific organization:**
```
GET /v1/users?organizationId=org-id-123
```

**Get admins without any organization assignment:**
```
GET /v1/users?role=org_admin
// Then filter in frontend: organizationMemberships.length === 0
```

**Response:**
```json
{
  "results": [
    {
      "id": "user-id",
      "email": "john.doe@example.com",
      "role": "org_admin",
      "organizationMemberships": [
        {
          "organizationId": "org-id-123",
          "role": "org_admin",
          "isActive": true,
          "joinedAt": "2024-01-15T10:00:00.000Z"
        }
      ],
      "firstName": "John",
      "lastName": "Doe",
      "jobTitle": "Administrator",
      "isActive": true
    }
  ],
  "page": 1,
  "limit": 10,
  "totalPages": 5,
  "totalResults": 50
}
```

---

### 3. Update Admin (PATCH `/v1/users/:userId`)

**Purpose:** Update admin details and assign organizations

**Request Body - Assign Organization:**
```json
{
  "organizationMemberships": [
    {
      "organizationId": "org-id-123",
      "role": "org_admin",
      "isActive": true
    },
    {
      "organizationId": "org-id-456",
      "role": "org_admin",
      "isActive": true
    }
  ]
}
```

**Request Body - Update Admin Info:**
```json
{
  "firstName": "John",
  "lastName": "Smith",
  "email": "john.smith@example.com",
  "jobTitle": "Senior Administrator",
  "isActive": true
}
```

**Request Body - Add Organization to Existing Admin:**
```json
{
  "organizationId": "org-id-789"  // ← Simple way to add one org
}
```

**Note:** When using `organizationId` in update, it will add a new organization membership if the admin doesn't already have it.

---

### 4. Get Single Admin (GET `/v1/users/:userId`)

**Purpose:** Get detailed information about a specific admin with all organization data

**Query Parameters:**
```
?populate=[{"path":"organizationMemberships.organizationId","select":"name email phone address isActive subscriptionId","populate":{"path":"subscriptionId","select":"planName planId status"}}]
```

**Full Populate Example (Recommended):**
```javascript
const populateOptions = [
  {
    path: "organizationMemberships.organizationId",
    select: "name email phone address domains isActive subscriptionId",
    populate: {
      path: "subscriptionId",
      select: "planName planId status"
    }
  }
];

// URL encoded
?populate=[{"path":"organizationMemberships.organizationId","select":"name email phone address domains isActive subscriptionId","populate":{"path":"subscriptionId","select":"planName planId status"}}]
```

**Note:** `domains` is an array of strings - organizations can have multiple domains.

**Response:**
```json
{
  "id": "user-id",
  "email": "john.doe@example.com",
  "role": "org_admin",
  "firstName": "John",
  "lastName": "Doe",
  "dob": "1990-01-01T00:00:00.000Z",
  "dateOfHire": "2024-01-01T00:00:00.000Z",
  "jobTitle": "Administrator",
  "isActive": true,
  "isEmailVerified": true,
  "createdAt": "2024-01-15T10:00:00.000Z",
  "updatedAt": "2024-01-20T10:00:00.000Z",
  "organizationMemberships": [
    {
      "organizationId": {
        "_id": "org-id-123",
        "id": "org-id-123",
        "name": "TechCorp Inc",
        "email": "contact@techcorp.com",
        "phone": "+1-234-567-8900",
        "address": "123 Tech Street, San Francisco, CA 94105",
        "domains": ["techcorp.com", "techcorp.io", "techcorp.net"],
        "isActive": true,
        "subscriptionId": {
          "_id": "sub-id-123",
          "planName": "Enterprise",
          "planId": "enterprise",
          "status": "active"
        }
      },
      "role": "org_admin",
      "isActive": true,
      "joinedAt": "2024-01-15T10:00:00.000Z"
    },
    {
      "organizationId": {
        "_id": "org-id-456",
        "id": "org-id-456",
        "name": "StartupHub",
        "email": "hello@startuphub.com",
        "phone": "+1-234-567-8901",
        "address": "456 Startup Ave, New York, NY 10001",
        "domains": ["startuphub.com"],
        "isActive": true,
        "subscriptionId": {
          "_id": "sub-id-456",
          "planName": "Professional",
          "planId": "professional",
          "status": "active"
        }
      },
      "role": "org_admin",
      "isActive": true,
      "joinedAt": "2024-01-18T10:00:00.000Z"
    }
  ]
}
```

**If Admin Has No Organizations:**
```json
{
  "id": "user-id",
  "email": "jane.admin@example.com",
  "role": "org_admin",
  "firstName": "Jane",
  "lastName": "Admin",
  "organizationMemberships": [],  // ← Empty array
  "isActive": true,
  "isEmailVerified": false
}
```

---

### 5. Delete Admin (DELETE `/v1/users/:userId`)

**Purpose:** Soft delete an admin

**Response:** 204 No Content

---

## Frontend Implementation Ideas

### 1. Admin List View

**Features:**
- Table/grid showing all admins
- Filter dropdowns:
  - **Role Filter:** All / Super Admin / Org Admin / User
  - **Organization Filter:** All Organizations / Specific Organization
  - **Status Filter:** Active / Inactive / All
- Search bar (by name, email)
- Pagination
- Action buttons: Edit, Delete, Assign Organization

**Table Columns:**
- Name (First + Last)
- Email
- Role
- Assigned Organizations (comma-separated list or badges)
- Status (Active/Inactive badge)
- Created Date
- Actions (View Details, Edit, Delete, Assign Org)

**Admin List Item Structure:**
Each admin in the list should display:
```
┌─────────────────────────────────────────────────────────────┐
│ [Avatar] John Doe                    [Active] [Verified]    │
│          john.doe@example.com                               │
│          Administrator                                      │
│          Organizations: TechCorp Inc, StartupHub (2)       │
│          Created: Jan 15, 2024                              │
│                              [View Details] [Edit] [Delete] │
└─────────────────────────────────────────────────────────────┘
```

**Or in Table Format:**
| Name | Email | Role | Organizations | Status | Created | Actions |
|------|-------|------|----------------|--------|---------|---------|
| John Doe | john@example.com | Org Admin | TechCorp Inc, StartupHub | Active | Jan 15, 2024 | [View] [Edit] [Delete] |
| Jane Smith | jane@example.com | Org Admin | Unassigned | Active | Jan 10, 2024 | [View] [Edit] [Delete] |

**Filter Combinations for Super Admin:**

**Quick Filter Buttons:**
```
1. "All Organization Admins" 
   → GET /v1/users?role=org_admin
   → Shows all org admins across all organizations

2. "Admins in [Organization Name]" 
   → GET /v1/users?role=org_admin&organizationId=org-id
   → Shows only admins assigned to specific organization

3. "Unassigned Admins" 
   → GET /v1/users?role=org_admin
   → Filter in frontend: organizationMemberships.length === 0
   → Shows admins without any organization assignment

4. "All Users in [Organization Name]" 
   → GET /v1/users?organizationId=org-id
   → Shows ALL users (admins + regular users) in that organization

5. "Active Admins" 
   → GET /v1/users?role=org_admin&isActive=true
   → Shows only active organization admins

6. "All Regular Users" 
   → GET /v1/users?role=user
   → Shows all regular users (not admins)

7. "Users in [Organization]" 
   → GET /v1/users?organizationId=org-id&role=user
   → Shows only regular users in specific organization
```

**Advanced Filter Panel:**
```
Role Filter: [All | Super Admin | Org Admin | User]
Organization Filter: [All | TechCorp | StartupHub | ...]
Status Filter: [All | Active | Inactive]
Unassigned Only: [Toggle checkbox]
```

---

### 2. Create Admin Form

**Form Fields:**
- **Required:**
  - First Name (text input)
  - Last Name (text input)
  - Email (email input)
  - Password (password input with strength indicator)
  - Date of Birth (date picker - optional for org_admin/superadmin, required for user)
  - Date of Hire (date picker - optional for org_admin/superadmin, required for user)
  - Job Title (text input - optional for org_admin/superadmin, required for user)
  - Role (dropdown: `org_admin` / `user` / `superadmin`)

- **Optional:**
  - Organization (dropdown - can be left empty for org_admin)
  - Department (dropdown - disabled if no org selected)
  - Email Verified (toggle - default: false)
  - Active Status (toggle - default: true)

**Form Behavior:**
- When `role` is `org_admin`:
  - Organization field becomes **optional** (can be left empty)
  - Show info message: "Organization can be assigned later if not available now"
  - Department field disabled until organization is selected
  - **This allows creating admins before organizations exist**
  - **Date of Birth, Date of Hire, and Job Title become optional** (not required for org_admin)
  - **Email Verification:** Show note: "A verification email will be automatically sent to the admin. They must login and verify their email using the OTP code."
- When `role` is `user`:
  - Organization field required
  - Department field optional but enabled
- When `role` is `superadmin`:
  - Organization and Department fields hidden

**Validation:**
- Email must be unique
- Password: min 8 chars, at least one letter and one number
- Date of Birth must be in the past (optional for org_admin/superadmin)
- Date of Hire must be in the past or today (optional for org_admin/superadmin)

---

### 3. Edit Admin Form

**Form Sections:**

**Section 1: Basic Information**
- First Name
- Last Name
- Email
- Job Title (optional for org_admin/superadmin, required for user)
- Date of Birth (optional for org_admin/superadmin, required for user)
- Date of Hire (optional for org_admin/superadmin, required for user)
- Active Status (toggle)

**Section 2: Organization Assignment**
- Current Organizations (list/badges with remove button)
- Add Organization (dropdown + "Add" button)
- Show message if no organizations assigned: "This admin is not assigned to any organization yet"

**Organization Assignment UI:**
```
Current Organizations:
┌─────────────────────────────────┐
│ TechCorp Inc          [Remove]  │
│ StartupHub            [Remove]  │
└─────────────────────────────────┘

Add Organization:
[Dropdown: Select Organization...] [Add]
```

**Section 3: Security**
- Change Password (optional, with "Change Password" toggle)
- Email Verified (toggle)

---

### 4. Assign Organization Modal/Dialog

**Purpose:** Quick way to assign organizations to an admin

**UI:**
```
Assign Organizations to John Doe

Available Organizations:
☑ TechCorp Inc
☐ StartupHub
☐ Global Dynamics
☐ Creative Agency

[Cancel] [Assign Selected]
```

**Alternative - Multi-Select Dropdown:**
```
Organizations: [Select multiple... ▼]
  ☑ TechCorp Inc
  ☑ StartupHub
  ☐ Global Dynamics
  ☐ Creative Agency
```

---

### 5. Filter Panel

**Location:** Above the admin list table

**Filter Options:**

**Quick Filters (Buttons):**
- All Admins
- Organization Admins
- Unassigned Admins
- Active Admins
- Inactive Admins

**Advanced Filters (Dropdowns):**
- Role: [All / Super Admin / Org Admin / User]
- Organization: [All / TechCorp / StartupHub / ...]
- Status: [All / Active / Inactive]
- Email Verified: [All / Verified / Unverified]

**Search:**
- Search by name or email (real-time search)

**Filter State Management:**
```typescript
interface FilterState {
  role?: 'org_admin' | 'user' | 'superadmin';
  organizationId?: string;
  isActive?: boolean;
  search?: string;
  page: number;
  limit: number;
}
```

---

### 6. Admin Detail View

**Purpose:** Show complete admin information with all their organization details

**API Call:**
```
GET /v1/users/:userId?populate=[{"path":"organizationMemberships.organizationId","select":"name email phone address isActive subscriptionId","populate":{"path":"subscriptionId","select":"planName planId status"}}]
```

**UI Structure:**

**Header Section:**
```
┌─────────────────────────────────────────────────────────────┐
│ [Back]  Admin Details                                       │
│                                                              │
│ [Avatar] John Doe                                           │
│          john.doe@example.com                                │
│          Administrator                                      │
│          [Active Badge] [Email Verified Badge]              │
│                                                              │
│ [Edit] [Delete] [Assign Organization]                      │
└─────────────────────────────────────────────────────────────┘
```

**Main Content - Two Column Layout:**

**Left Column - Admin Information:**
```
┌─────────────────────────────────────┐
│ Personal Information                │
├─────────────────────────────────────┤
│ First Name:        John             │
│ Last Name:         Doe              │
│ Email:             john@example.com │
│ Date of Birth:     Jan 1, 1990      │
│ Date of Hire:      Jan 1, 2024      │
│ Job Title:         Administrator    │
│ Role:              Org Admin        │
│ Status:            Active           │
│ Email Verified:    Yes              │
│ Created:           Jan 15, 2024     │
│ Last Updated:      Jan 20, 2024     │
└─────────────────────────────────────┘
```

**Right Column - Organizations:**
```
┌─────────────────────────────────────┐
│ Assigned Organizations (2)          │
│ [+ Add Organization]                │
├─────────────────────────────────────┤
│ ┌─────────────────────────────────┐ │
│ │ TechCorp Inc                    │ │
│ │ contact@techcorp.com             │ │
│ │ +1-234-567-8900                 │ │
│ │ 123 Tech Street, San Francisco  │ │
│ │ Domains: techcorp.com,          │ │
│ │          techcorp.io             │ │
│ │ Status: Active                  │ │
│ │ Plan: Enterprise                │ │
│ │ Joined: Jan 15, 2024            │ │
│ │ [Remove] [View Org Details]     │ │
│ └─────────────────────────────────┘ │
│                                     │
│ ┌─────────────────────────────────┐ │
│ │ StartupHub                      │ │
│ │ hello@startuphub.com             │ │
│ │ +1-234-567-8901                 │ │
│ │ 456 Startup Ave, New York       │ │
│ │ Domains: startuphub.com         │ │
│ │ Status: Active                  │ │
│ │ Plan: Professional              │ │
│ │ Joined: Jan 18, 2024            │ │
│ │ [Remove] [View Org Details]     │ │
│ └─────────────────────────────────┘ │
└─────────────────────────────────────┘
```

**If No Organizations:**
```
┌─────────────────────────────────────┐
│ Assigned Organizations (0)          │
│ [+ Add Organization]                │
├─────────────────────────────────────┤
│                                     │
│   No organizations assigned yet     │
│                                     │
│   [Assign Organization]             │
│                                     │
└─────────────────────────────────────┘
```

**Organization Card Details:**
Each organization card should show:
- Organization Name
- Email
- Phone
- Address
- **Domains** (array - can have multiple domains)
  - Display as: "Domains: domain1.com, domain2.com, domain3.com"
  - Or as badges/chips: [domain1.com] [domain2.com] [domain3.com]
  - If empty: "No domains configured"
- Status (Active/Inactive badge)
- Subscription Plan Name
- Subscription Status
- Joined Date (when admin was assigned)
- Actions: Remove, View Full Organization Details

**Full Organization Data Structure:**
```typescript
interface AdminDetailResponse {
  id: string;
  email: string;
  firstName: string;
  lastName: string;
  dob: string;
  dateOfHire: string;
  jobTitle: string;
  role: 'org_admin';
  isActive: boolean;
  isEmailVerified: boolean;
  createdAt: string;
  updatedAt: string;
  organizationMemberships: Array<{
      organizationId: {
        id: string;
        name: string;
        email: string;
        phone: string;
        address: string;
        domains: string[]; // Array of domain strings
        isActive: boolean;
        subscriptionId?: {
          planName: string;
          planId: string;
          status: string;
        };
      };
    role: 'org_admin';
    isActive: boolean;
    joinedAt: string;
  }>;
}
```

---

## TypeScript Types/Interfaces

```typescript
interface Admin {
  id: string;
  email: string;
  firstName: string;
  lastName: string;
  jobTitle: string;
  role: 'org_admin' | 'user' | 'superadmin';
  organizationMemberships: OrganizationMembership[];
  isActive: boolean;
  isEmailVerified: boolean;
  createdAt: string;
  updatedAt: string;
}

interface OrganizationMembership {
  organizationId: string | Organization;
  role: 'org_admin' | 'user';
  isActive: boolean;
  joinedAt: string;
}

interface Organization {
  id: string;
  name: string;
  email: string;
  phone?: string;
  address?: string;
  domains: string[]; // Array of domain strings - organizations can have multiple domains
  isActive: boolean;
  subscriptionId?: {
    planName: string;
    planId: string;
    status: string;
  };
}

interface AdminDetailResponse extends Admin {
  dob: string;
  dateOfHire: string;
  createdAt: string;
  updatedAt: string;
  organizationMemberships: Array<{
    organizationId: Organization;
    role: 'org_admin' | 'user';
    isActive: boolean;
    joinedAt: string;
  }>;
}

interface CreateAdminRequest {
  firstName: string;
  lastName: string;
  email: string;
  password: string;
  dob: string; // ISO date
  dateOfHire: string; // ISO date
  jobTitle: string;
  role: 'org_admin' | 'user' | 'superadmin';
  organizationId?: string | null; // Optional for org_admin
  department?: string | null;
  isEmailVerified?: boolean;
  isActive?: boolean;
}

interface UpdateAdminRequest {
  firstName?: string;
  lastName?: string;
  email?: string;
  password?: string;
  jobTitle?: string;
  isActive?: boolean;
  isEmailVerified?: boolean;
  organizationId?: string; // Simple way to add one org
  organizationMemberships?: OrganizationMembership[]; // Full control
}

interface AdminListResponse {
  results: Admin[];
  page: number;
  limit: number;
  totalPages: number;
  totalResults: number;
}

interface FilterOptions {
  role?: 'org_admin' | 'user' | 'superadmin';
  organizationId?: string;
  isActive?: boolean;
  search?: string;
  page?: number;
  limit?: number;
}
```

---

## React Component Examples

### AdminDetailView Component

```typescript
import React, { useEffect, useState } from 'react';
import { adminService } from '../services/adminService';

interface AdminDetailViewProps {
  adminId: string;
  onBack: () => void;
}

const AdminDetailView: React.FC<AdminDetailViewProps> = ({ adminId, onBack }) => {
  const [admin, setAdmin] = useState<AdminDetailResponse | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const fetchAdmin = async () => {
      try {
        const response = await adminService.getAdmin(adminId);
        setAdmin(response.data);
      } catch (error) {
        console.error('Failed to fetch admin:', error);
      } finally {
        setLoading(false);
      }
    };

    fetchAdmin();
  }, [adminId]);

  if (loading) return <div>Loading...</div>;
  if (!admin) return <div>Admin not found</div>;

  return (
    <div className="admin-detail-view">
      {/* Header */}
      <div className="admin-header">
        <button onClick={onBack}>← Back</button>
        <h1>Admin Details</h1>
        <div className="admin-avatar-section">
          <img src={admin.imageUrl || '/default-avatar.png'} alt="Avatar" />
          <h2>{admin.firstName} {admin.lastName}</h2>
          <p>{admin.email}</p>
          <p>{admin.jobTitle}</p>
          <div className="badges">
            {admin.isActive && <span className="badge active">Active</span>}
            {admin.isEmailVerified && <span className="badge verified">Verified</span>}
          </div>
          <div className="actions">
            <button>Edit</button>
            <button>Delete</button>
            <button>Assign Organization</button>
          </div>
        </div>
      </div>

      {/* Two Column Layout */}
      <div className="admin-content">
        {/* Left Column - Admin Info */}
        <div className="admin-info-column">
          <div className="info-card">
            <h3>Personal Information</h3>
            <div className="info-row">
              <label>First Name:</label>
              <span>{admin.firstName}</span>
            </div>
            <div className="info-row">
              <label>Last Name:</label>
              <span>{admin.lastName}</span>
            </div>
            <div className="info-row">
              <label>Email:</label>
              <span>{admin.email}</span>
            </div>
            <div className="info-row">
              <label>Date of Birth:</label>
              <span>{new Date(admin.dob).toLocaleDateString()}</span>
            </div>
            <div className="info-row">
              <label>Date of Hire:</label>
              <span>{new Date(admin.dateOfHire).toLocaleDateString()}</span>
            </div>
            <div className="info-row">
              <label>Job Title:</label>
              <span>{admin.jobTitle}</span>
            </div>
            <div className="info-row">
              <label>Role:</label>
              <span>{admin.role}</span>
            </div>
            <div className="info-row">
              <label>Status:</label>
              <span>{admin.isActive ? 'Active' : 'Inactive'}</span>
            </div>
            <div className="info-row">
              <label>Email Verified:</label>
              <span>{admin.isEmailVerified ? 'Yes' : 'No'}</span>
            </div>
            <div className="info-row">
              <label>Created:</label>
              <span>{new Date(admin.createdAt).toLocaleDateString()}</span>
            </div>
            <div className="info-row">
              <label>Last Updated:</label>
              <span>{new Date(admin.updatedAt).toLocaleDateString()}</span>
            </div>
          </div>
        </div>

        {/* Right Column - Organizations */}
        <div className="organizations-column">
          <div className="organizations-header">
            <h3>Assigned Organizations ({admin.organizationMemberships.length})</h3>
            <button>+ Add Organization</button>
          </div>

          {admin.organizationMemberships.length === 0 ? (
            <div className="empty-state">
              <p>No organizations assigned yet</p>
              <button>Assign Organization</button>
            </div>
          ) : (
            <div className="organization-cards">
              {admin.organizationMemberships.map((membership) => {
                const org = membership.organizationId;
                return (
                  <div key={org.id} className="organization-card">
                    <h4>{org.name}</h4>
                    <div className="org-details">
                      <p><strong>Email:</strong> {org.email}</p>
                      {org.phone && <p><strong>Phone:</strong> {org.phone}</p>}
                      {org.address && <p><strong>Address:</strong> {org.address}</p>}
                      <p>
                        <strong>Domains:</strong>{' '}
                        {org.domains && org.domains.length > 0 ? (
                          <span className="domains-list">
                            {org.domains.map((domain, idx) => (
                              <span key={idx} className="domain-badge">
                                {domain}
                              </span>
                            ))}
                          </span>
                        ) : (
                          <span className="text-muted">No domains configured</span>
                        )}
                      </p>
                      <p>
                        <strong>Status:</strong>{' '}
                        <span className={org.isActive ? 'badge active' : 'badge inactive'}>
                          {org.isActive ? 'Active' : 'Inactive'}
                        </span>
                      </p>
                      {org.subscriptionId && (
                        <>
                          <p><strong>Plan:</strong> {org.subscriptionId.planName}</p>
                          <p>
                            <strong>Subscription:</strong>{' '}
                            <span className={`badge ${org.subscriptionId.status}`}>
                              {org.subscriptionId.status}
                            </span>
                          </p>
                        </>
                      )}
                      <p>
                        <strong>Joined:</strong>{' '}
                        {new Date(membership.joinedAt).toLocaleDateString()}
                      </p>
                    </div>
                    <div className="org-actions">
                      <button>Remove</button>
                      <button>View Org Details</button>
                    </div>
                  </div>
                );
              })}
            </div>
          )}
        </div>
      </div>
    </div>
  );
};

export default AdminDetailView;
```

---

## React Hook Examples

### useAdmins Hook

```typescript
const useAdmins = (filters: FilterOptions) => {
  const [admins, setAdmins] = useState<Admin[]>([]);
  const [loading, setLoading] = useState(false);
  const [pagination, setPagination] = useState({...});

  const fetchAdmins = async () => {
    // Build query string from filters
    // GET /v1/users?role=org_admin&organizationId=...
  };

  const createAdmin = async (data: CreateAdminRequest) => {
    // POST /v1/users
  };

  const updateAdmin = async (id: string, data: UpdateAdminRequest) => {
    // PATCH /v1/users/:id
  };

  const deleteAdmin = async (id: string) => {
    // DELETE /v1/users/:id
  };

  return {
    admins,
    loading,
    pagination,
    fetchAdmins,
    createAdmin,
    updateAdmin,
    deleteAdmin,
  };
};
```

---

## UI/UX Considerations

### 1. Empty States

**No Admins:**
- Show "No admins found" message
- "Create Admin" button

**Unassigned Admin:**
- Show badge: "Unassigned"
- Show "Assign Organization" button prominently

### 2. Status Indicators

**Active/Inactive:**
- Green badge: "Active"
- Red badge: "Inactive"

**Email Verified:**
- Checkmark icon: Verified
- Warning icon: Unverified

### 3. Organization Display

**Single Organization:**
- Show organization name as text

**Multiple Organizations:**
- Show first 2 names + "and X more"
- Or show as badges/chips
- On hover/click: Show full list

### 4. Confirmation Dialogs

**Delete Admin:**
- "Are you sure you want to delete [Name]?"
- "This action cannot be undone"

**Remove Organization:**
- "Remove [Admin] from [Organization]?"
- "They will lose access to this organization"

### 5. Success/Error Messages

**Create Success:**
- "Admin created successfully"
- "Verification email sent to [email]. The admin must login and verify their email." (automatically included in response `message` field from backend)
- "Organization assigned" (if org was provided)

**Note:** The backend automatically sends verification email. Frontend should display the `message` from the API response.

**Update Success:**
- "Admin updated successfully"
- "Organization assigned" (if org was added)

**Errors:**
- Email already exists
- Organization not found
- Validation errors

---

## Workflow Examples

### Workflow 1: Create Admin Without Organization (No Orgs Exist Yet)

1. Super admin clicks "Create Admin"
2. Fills form:
   - Role: `org_admin`
   - Organization: **Leave empty** (no organizations exist yet)
   - Other required fields filled
3. Submits → Admin created successfully
4. Admin appears in list with "Unassigned" badge
5. Later, when organizations are created:
   - Super admin opens admin detail/edit page
   - Clicks "Assign Organization"
   - Selects organization(s) → Admin assigned

**API Call:**
```json
POST /v1/users
{
  "firstName": "John",
  "lastName": "Doe",
  "email": "john@example.com",
  "password": "Secure123",
  "dob": "1990-01-01",
  "dateOfHire": "2024-01-01",
  "jobTitle": "Administrator",
  "role": "org_admin"
  // organizationId: NOT PROVIDED (optional)
}
```

**Response:**
```json
{
  "id": "admin-id",
  "email": "john@example.com",
  "role": "org_admin",
  "organizationMemberships": []  // ← Empty array
}
```

### Workflow 2: Create Admin With Organization

1. Super admin clicks "Create Admin"
2. Fills form (role: org_admin, organization: selected)
3. Submits → Admin created and assigned
4. Admin appears in list with organization name

### Workflow 3: Filter Admins by Organization

1. Super admin selects "TechCorp Inc" from organization filter
2. List shows only admins assigned to TechCorp
3. Can further filter by role (e.g., only org_admins)

### Workflow 4: Assign Organization at Update Time

1. Super admin opens admin detail/edit page (admin was created without org)
2. Clicks "Assign Organization" button
3. Selects one or multiple organizations from dropdown
4. Saves → Admin now assigned to selected organization(s)

**API Call:**
```json
PATCH /v1/users/admin-id
{
  "organizationMemberships": [
    {
      "organizationId": "org-id-123",
      "role": "org_admin",
      "isActive": true
    }
  ]
}
```

**Or simpler way (single org):**
```json
PATCH /v1/users/admin-id
{
  "organizationId": "org-id-123"
}
```

### Workflow 5: Filter Admins by Organization

1. Super admin selects "TechCorp Inc" from organization filter dropdown
2. Selects "Org Admins" from role filter
3. List shows only admins assigned to TechCorp Inc
4. Can see: "John Doe - TechCorp Inc", "Jane Smith - TechCorp Inc"

**API Call:**
```
GET /v1/users?role=org_admin&organizationId=techcorp-id
```

### Workflow 6: View All Users in Organization

1. Super admin selects "TechCorp Inc" from organization filter
2. Leaves role filter as "All"
3. List shows ALL users in TechCorp (admins + regular users)

**API Call:**
```
GET /v1/users?organizationId=techcorp-id
```

---

## API Service Functions

```typescript
// adminService.ts

export const adminService = {
  // Get all admins with filters
  getAdmins: (filters: FilterOptions) => {
    const params = new URLSearchParams();
    if (filters.role) params.append('role', filters.role);
    if (filters.organizationId) params.append('organizationId', filters.organizationId);
    if (filters.isActive !== undefined) params.append('isActive', String(filters.isActive));
    if (filters.page) params.append('page', String(filters.page));
    if (filters.limit) params.append('limit', String(filters.limit));
    
    return api.get(`/v1/users?${params.toString()}`);
  },

  // Create admin
  createAdmin: (data: CreateAdminRequest) => {
    return api.post('/v1/users', data);
  },

  // Get single admin with full organization details
  getAdmin: (id: string) => {
    const populateOptions = [
      {
      path: 'organizationMemberships.organizationId',
      select: 'name email phone address domains isActive subscriptionId',
      populate: {
        path: 'subscriptionId',
        select: 'planName planId status'
      }
      }
    ];
    
    return api.get(`/v1/users/${id}`, {
      params: {
        populate: JSON.stringify(populateOptions)
      }
    });
  },

  // Update admin
  updateAdmin: (id: string, data: UpdateAdminRequest) => {
    return api.patch(`/v1/users/${id}`, data);
  },

  // Delete admin
  deleteAdmin: (id: string) => {
    return api.delete(`/v1/users/${id}`);
  },

  // Assign organization (helper function)
  assignOrganization: (adminId: string, organizationId: string) => {
    return api.patch(`/v1/users/${adminId}`, {
      organizationId: organizationId
    });
  },
};
```

---

## Testing Scenarios

1. **Create org_admin without organization** → Should succeed
2. **Create org_admin with organization** → Should succeed and assign
3. **Filter by role=org_admin** → Should show only org admins
4. **Filter by organizationId** → Should show only users in that org
5. **Assign organization to unassigned admin** → Should update successfully
6. **Remove organization from admin** → Should update successfully
7. **Update admin with multiple organizations** → Should handle correctly
8. **Delete admin** → Should soft delete

---

## Summary

### ✅ Three Main Requirements

**1. Create Org-Admin Without Organization**
- When super admin creates `org_admin`, `organizationId` is **completely optional**
- This allows creating admins **before organizations exist** in the system
- Admin will have empty `organizationMemberships: []` array
- Show "Unassigned" badge in UI

**2. Assign Organizations at Update Time**
- Super admin can assign organizations to admins **after creation**
- Use `PATCH /v1/users/:userId` with:
  - `organizationId: "org-id"` (simple - adds one org)
  - OR `organizationMemberships: [...]` (advanced - full control)
- This allows workflow: Create admin → Create organizations → Assign orgs to admin

**3. Super Admin Filtering Capabilities**
- **All Admins:** `GET /v1/users?role=org_admin`
- **Organization-Based Admins:** `GET /v1/users?role=org_admin&organizationId=org-id`
- **Organization-Based Users:** `GET /v1/users?organizationId=org-id` (all users in org)
- **Unassigned Admins:** `GET /v1/users?role=org_admin` + filter frontend for empty memberships
- **Combine Filters:** Role + Organization + Status

---

### Key Points for Frontend Implementation

1. ✅ `organizationId` is **optional** when creating `org_admin` (can be `null` or omitted)
2. ✅ Use `organizationMemberships` array to assign multiple organizations
3. ✅ Use `role` query parameter to filter by admin type (`org_admin`, `user`, `superadmin`)
4. ✅ Use `organizationId` query parameter to filter by organization
5. ✅ Combine filters for advanced filtering (role + organizationId + isActive)
6. ✅ Show "Unassigned" state for admins without organizations (`organizationMemberships.length === 0`)
7. ✅ Provide easy way to assign organizations after creation (update form/modal)
8. ✅ Support assigning multiple organizations to one admin

---

### API Endpoints Summary

| Endpoint | Purpose | Key Notes |
|----------|---------|-----------|
| `POST /v1/users` | Create admin | `organizationId` optional for `org_admin` |
| `GET /v1/users?role=org_admin` | Get all org admins | Shows all admins across all orgs |
| `GET /v1/users?role=org_admin&organizationId=xxx` | Get admins in org | Shows only admins in specific org |
| `GET /v1/users?organizationId=xxx` | Get all users in org | Shows admins + regular users |
| `PATCH /v1/users/:id` | Update admin | Assign organizations via `organizationMemberships` or `organizationId` |
| `GET /v1/users/:id` | Get admin details | Include organizations in response |
| `DELETE /v1/users/:id` | Delete admin | Soft delete |

---

### Frontend UI Requirements

**Create Form:**
- Organization dropdown should be **optional** when role is `org_admin`
- Show message: "Organization can be assigned later if not available now"
- Allow form submission without organization

**List View:**
- Show "Unassigned" badge for admins without organizations
- Filter dropdowns: Role, Organization, Status
- Quick filter buttons for common scenarios

**Edit/Detail View:**
- Show complete admin information (personal details, status, dates)
- Show all assigned organizations with full details:
  - Organization name, email, phone, address
  - **Domains** (array - display as badges or comma-separated list)
  - Subscription plan and status
  - Joined date for each organization
- "Assign Organization" button/modal
- Ability to add/remove organizations
- "View Organization Details" link for each organization
- Display "Unassigned" state if no organizations

**Admin Detail View Layout:**
- Two-column layout: Admin Info (left) | Organizations (right)
- Organization cards showing all organization data
- Empty state when no organizations assigned
- Action buttons: Edit, Delete, Assign Organization

