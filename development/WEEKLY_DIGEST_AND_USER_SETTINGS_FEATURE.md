# Weekly Digest & User Settings Feature - Implementation Guide

## 🎯 Feature Overview

A comprehensive system including:
- ✅ **Weekly Digest Configuration** (Admin controls)
- ✅ **User Personal Settings**
- ✅ **Dashboard APIs** for all user types
- ✅ **Automated weekly digest sending**
- ✅ **Department-based filtering**
- ✅ **Multi-language support**

---

## 📦 Models Created

### 1. WeeklyDigest Model (`src/models/weeklyDigest.model.js`)

**Organization-level configuration for weekly digests:**

```javascript
{
  organizationId: ObjectId,           // Unique per organization
  enabled: Boolean,                   // Turn on/off
  sendDay: String,                    // monday, tuesday, etc.
  sendTime: String,                   // "10:00" (24-hour format)
  audience: {
    type: String,                     // all_organization, specific_departments, specific_users
    departmentIds: [ObjectId],        // If specific_departments
    userIds: [ObjectId]               // If specific_users
  },
  contentSettings: {
    includeBirthdays: Boolean,
    includeWorkAnniversaries: Boolean,
    includeNewMembers: Boolean,
    includeQuestions: Boolean,
    includeReactions: Boolean
  },
  lastSentAt: Date,
  nextScheduledAt: Date,              // Auto-calculated
  createdBy: ObjectId
}
```

### 2. WeeklyDigestLog Model (`src/models/weeklyDigestLog.model.js`)

**Tracks every digest sent (history):**

```javascript
{
  organizationId: ObjectId,
  digestId: ObjectId,
  sentAt: Date,
  scheduledAt: Date,
  status: String,                     // sent, failed, partial
  audience: { type, departmentIds, userIds },
  contentIncluded: {
    birthdays: Number,
    workAnniversaries: Number,
    newMembers: Number,
    questions: Number,
    reactions: Number
  },
  recipients: {
    total: Number,
    sent: Number,
    failed: Number
  },
  errors: [{
    userId, email, error, timestamp
  }],
  metadata: {
    sendDuration: Number,
    emailProvider: String,
    templateUsed: String
  }
}
```

### 3. UserSettings Model (`src/models/userSettings.model.js`)

**User personal preferences:**

```javascript
{
  userId: ObjectId,
  organizationId: ObjectId,
  
  // Question settings
  questionSettings: {
    allowReactionsOnMyAnswers: Boolean,
    allowNotificationsForNewQuestions: Boolean
  },
  
  // Notification preferences
  notifications: {
    remindersFor: String,             // all_company, my_department, none
    anniversaryReminder: String,       // day_of, before_1_day, before_2_days, before_week, none
    birthdayReminder: String,          // day_of, before_1_day, before_2_days, before_week, none
    emailNotifications: Boolean,
    pushNotifications: Boolean,
    weeklyDigest: Boolean              // Receive weekly digest?
  },
  
  // Language
  language: String,                    // en-US, en-GB, es-ES, es-LA, fr-FR, it-IT, pt-BR
  
  // Privacy
  privacy: {
    showBirthdayToTeam: Boolean,
    showWorkAnniversaryToTeam: Boolean,
    showProfileInDirectory: Boolean
  },
  
  // Department filters
  departmentFilters: [ObjectId]
}
```

---

## 🛣️ API Endpoints (To Be Created)

### Weekly Digest Management

#### 1. Get Digest Configuration
```http
GET /v1/weekly-digest/:organizationId
Authorization: Bearer <admin-token>
```
**Response:**
```json
{
  "id": "...",
  "organizationId": "...",
  "enabled": true,
  "sendDay": "monday",
  "sendTime": "10:00",
  "audience": {
    "type": "all_organization",
    "departmentIds": [],
    "userIds": []
  },
  "contentSettings": {
    "includeBirthdays": true,
    "includeWorkAnniversaries": true,
    "includeNewMembers": true,
    "includeQuestions": false,
    "includeReactions": false
  },
  "lastSentAt": "2024-12-12T09:00:00Z",
  "nextScheduledAt": "2024-12-19T10:00:00Z"
}
```

#### 2. Update Digest Configuration
```http
PATCH /v1/weekly-digest/:organizationId
Authorization: Bearer <admin-token>

Body:
{
  "enabled": true,
  "sendDay": "monday",
  "sendTime": "10:00",
  "audience": {
    "type": "specific_departments",
    "departmentIds": ["dept1", "dept2"]
  },
  "contentSettings": {
    "includeBirthdays": true,
    "includeWorkAnniversaries": true,
    "includeNewMembers": true
  }
}
```

#### 3. Get Digest History
```http
GET /v1/weekly-digest/:organizationId/history?page=1&limit=10
Authorization: Bearer <admin-token>
```
**Response:**
```json
{
  "results": [
    {
      "id": "...",
      "sentAt": "2024-12-12T09:00:00Z",
      "status": "sent",
      "audience": {
        "type": "all_organization"
      },
      "recipients": {
        "total": 50,
        "sent": 50,
        "failed": 0
      },
      "contentIncluded": {
        "birthdays": 3,
        "workAnniversaries": 2,
        "newMembers": 1
      }
    }
  ],
  "page": 1,
  "limit": 10,
  "totalPages": 3,
  "totalResults": 25
}
```

#### 4. Send Digest Manually
```http
POST /v1/weekly-digest/:organizationId/send
Authorization: Bearer <admin-token>
```

---

### User Settings

#### 5. Get User Settings
```http
GET /v1/user-settings
Authorization: Bearer <token>
```

#### 6. Update User Settings
```http
PATCH /v1/user-settings
Authorization: Bearer <token>

Body:
{
  "questionSettings": {
    "allowReactionsOnMyAnswers": true,
    "allowNotificationsForNewQuestions": true
  },
  "notifications": {
    "remindersFor": "all_company",
    "anniversaryReminder": "day_of",
    "birthdayReminder": "before_1_day",
    "weeklyDigest": true
  },
  "language": "en-US",
  "privacy": {
    "showBirthdayToTeam": true
  }
}
```

---

### Dashboard APIs

#### 7. User Dashboard
```http
GET /v1/dashboard/user
Authorization: Bearer <user-token>
```
**Response:**
```json
{
  "user": {
    "firstName": "John",
    "lastName": "Doe",
    "role": "user"
  },
  "upcomingBirthdays": [
    {
      "user": { "firstName": "Jane", "lastName": "Smith" },
      "date": "2024-12-25",
      "daysUntil": 3
    }
  ],
  "upcomingAnniversaries": [...],
  "newMembers": [...],
  "unansweredQuestions": 5,
  "recentReactions": [...]
}
```

#### 8. Admin Dashboard
```http
GET /v1/dashboard/admin/:organizationId
Authorization: Bearer <admin-token>
```
**Response:**
```json
{
  "organization": {
    "name": "TechCorp",
    "totalUsers": 150,
    "activeDepartments": 10
  },
  "stats": {
    "totalUsers": 150,
    "activeUsers": 145,
    "newUsersThisMonth": 5,
    "totalDepartments": 10,
    "questionResponseRate": 85.5,
    "weeklyDigestEnabled": true,
    "lastDigestSent": "2024-12-12T09:00:00Z"
  },
  "upcomingBirthdays": [...],
  "upcomingAnniversaries": [...],
  "recentQuestions": [...],
  "topReactions": [...]
}
```

#### 9. SuperAdmin Dashboard
```http
GET /v1/dashboard/superadmin
Authorization: Bearer <superadmin-token>
```
**Response:**
```json
{
  "globalStats": {
    "totalOrganizations": 10,
    "totalUsers": 1500,
    "activeUsers": 1425,
    "totalDepartments": 100
  },
  "organizations": [
    {
      "id": "...",
      "name": "TechCorp",
      "users": 150,
      "activeUsers": 145,
      "digestEnabled": true
    }
  ],
  "recentActivity": [...],
  "systemHealth": {
    "database": "healthy",
    "emailService": "healthy",
    "scheduler": "healthy"
  }
}
```

---

## 🔄 Weekly Digest Workflow

### Admin Configuration Flow

```mermaid
Admin → Configure Digest Settings
   ↓
Set: Day, Time, Audience, Content
   ↓
System calculates nextScheduledAt
   ↓
Scheduler monitors nextScheduledAt
   ↓
When time arrives → Send digest
   ↓
Create log entry
   ↓
Update nextScheduledAt (next week)
```

### User Subscription Flow

```javascript
// User enables weekly digest in their settings
PATCH /v1/user-settings
{
  "notifications": {
    "weeklyDigest": true
  }
}

// When digest is sent:
// 1. System checks user's weeklyDigest setting
// 2. If true, includes user in recipients
// 3. If false, skips user (even if in audience)
```

### Content Generation

**For each enabled content type:**

1. **Birthdays:** Find users with birthdays in next 7 days
2. **Work Anniversaries:** Find users with anniversaries in next 7 days
3. **New Members:** Find users who joined in past 7 days
4. **Questions:** Get questions created in past 7 days
5. **Reactions:** Get top reactions from past 7 days

---

## 🎨 UI Requirements (Frontend)

### Admin: Weekly Digest Page

```
┌─────────────────────────────────────────────┐
│  Weekly Digest Configuration               │
├─────────────────────────────────────────────┤
│                                             │
│  Enable Weekly Digest    [Toggle: ON]      │
│                                             │
│  Send Day:      [Monday ▼]                 │
│  Send Time:     [10:00 AM]                 │
│                                             │
│  Audience:      [All Organization ▼]       │
│                 (or Specific Departments)   │
│                                             │
│  Content Settings:                          │
│    ☑ Birthdays                             │
│    ☑ Work Anniversaries                    │
│    ☑ New Team Members                      │
│    ☐ Recent Questions                      │
│    ☐ Top Reactions                         │
│                                             │
│  [Save Configuration]  [Send Test Digest]  │
│                                             │
├─────────────────────────────────────────────┤
│  Recent Digest History                      │
├─────────────────────────────────────────────┤
│  Dec 12, 2025  09:00 AM                    │
│  Status: Sent                               │
│  Audience: All Organization                 │
│  Recipients: 150 sent, 0 failed            │
│  ─────────────────────────────────────────  │
│  Dec 05, 2025  09:00 AM                    │
│  Status: Sent                               │
│  Audience: All Organization                 │
│  Recipients: 145 sent, 5 failed            │
└─────────────────────────────────────────────┘
```

### User: Settings Page

```
┌─────────────────────────────────────────────┐
│  My Settings                                │
├─────────────────────────────────────────────┤
│  Question Settings                          │
│    ☑ Allow reactions on my answers         │
│    ☑ Notify me of new questions            │
│                                             │
│  Notifications                              │
│    Get reminders for: [All Company ▼]      │
│    Anniversary reminder: [Day of ▼]        │
│    Birthday reminder: [Before a day ▼]     │
│    ☑ Email notifications                   │
│    ☑ Push notifications                    │
│    ☑ Weekly digest                         │
│                                             │
│  Language                                   │
│    [English (US) ▼]                        │
│                                             │
│  Privacy                                    │
│    ☑ Show birthday to team                 │
│    ☑ Show work anniversary to team         │
│    ☑ Show profile in directory             │
│                                             │
│  [Save Settings]                            │
└─────────────────────────────────────────────┘
```

---

## 🔔 Email Template (Weekly Digest)

```html
<!DOCTYPE html>
<html>
<head>
  <title>Weekly Digest - {{organizationName}}</title>
</head>
<body>
  <h1>This Week at {{organizationName}}</h1>
  
  <!-- Birthdays -->
  {{#if birthdays}}
  <h2>🎂 Upcoming Birthdays</h2>
  <ul>
    {{#each birthdays}}
    <li>{{firstName}} {{lastName}} - {{date}}</li>
    {{/each}}
  </ul>
  {{/if}}
  
  <!-- Work Anniversaries -->
  {{#if workAnniversaries}}
  <h2>🎉 Work Anniversaries</h2>
  <ul>
    {{#each workAnniversaries}}
    <li>{{firstName}} {{lastName}} - {{years}} years!</li>
    {{/each}}
  </ul>
  {{/if}}
  
  <!-- New Members -->
  {{#if newMembers}}
  <h2>👋 New Team Members</h2>
  <ul>
    {{#each newMembers}}
    <li>{{firstName}} {{lastName}} - {{jobTitle}}</li>
    {{/each}}
  </ul>
  {{/if}}
  
  <p>Have a great week!</p>
</body>
</html>
```

---

## ⏰ Scheduler Integration

Update `src/services/scheduler.service.js`:

```javascript
import cron from 'node-cron';
import weeklyDigestService from './weeklyDigest.service.js';

// Check every hour for digests to send
cron.schedule('0 * * * *', async () => {
  try {
    const digests = await weeklyDigestService.getDigestsToSend();
    
    for (const digest of digests) {
      try {
        await weeklyDigestService.sendWeeklyDigest(digest.organizationId);
        console.log(`Weekly digest sent for org: ${digest.organizationId}`);
      } catch (error) {
        console.error(`Failed to send digest for org ${digest.organizationId}:`, error);
      }
    }
  } catch (error) {
    console.error('Scheduler error:', error);
  }
});
```

---

## 🔐 Permissions

| Action | SuperAdmin | Org Admin | User |
|--------|-----------|-----------|------|
| **Weekly Digest** ||||
| View config | ✅ All orgs | ✅ Own org | ❌ |
| Update config | ✅ All orgs | ✅ Own org | ❌ |
| View history | ✅ All orgs | ✅ Own org | ❌ |
| Send manually | ✅ All orgs | ✅ Own org | ❌ |
| **User Settings** ||||
| View own | ✅ | ✅ | ✅ |
| Update own | ✅ | ✅ | ✅ |
| **Dashboards** ||||
| User dashboard | ✅ | ✅ | ✅ |
| Admin dashboard | ✅ All orgs | ✅ Own org | ❌ |
| SuperAdmin dashboard | ✅ | ❌ | ❌ |

---

## 📊 Database Indexes

```javascript
// WeeklyDigest
organizationId: unique
{ enabled: 1, nextScheduledAt: 1 }

// WeeklyDigestLog
{ organizationId: 1, sentAt: -1 }
{ status: 1 }

// UserSettings
{ userId: 1, organizationId: 1 }: unique
userId: 1
organizationId: 1
```

---

## ✅ Implementation Checklist

### Models ✅ COMPLETE
- ✅ WeeklyDigest model
- ✅ WeeklyDigestLog model
- ✅ UserSettings model

### Services (To Complete)
- ✅ weeklyDigest.service.js (basic structure)
- ⏳ userSettings.service.js
- ⏳ dashboard.service.js
- ⏳ Email template for weekly digest

### Controllers (To Create)
- ⏳ weeklyDigest.controller.js
- ⏳ userSettings.controller.js
- ⏳ dashboard.controller.js

### Validations (To Create)
- ⏳ weeklyDigest.validation.js
- ⏳ userSettings.validation.js

### Routes (To Create)
- ⏳ weekly-digest.route.js
- ⏳ user-settings.route.js
- ⏳ dashboard.route.js

### Swagger Docs (To Create)
- ⏳ Add schemas to components.yml
- ⏳ Document all endpoints

### Updates Needed
- ⏳ Update models/index.js
- ⏳ Update services/index.js
- ⏳ Update controllers/index.js
- ⏳ Update validations/index.js
- ⏳ Update routes/v1/index.js
- ⏳ Update scheduler.service.js

---

## 🚀 Next Steps

Due to the complexity and size of this feature, I recommend breaking it into phases:

### Phase 1 (Current)
- ✅ Models created
- ✅ Basic service structure
- Documentation complete

### Phase 2 (Next)
1. Complete remaining services
2. Create controllers
3. Create validations
4. Create routes
5. Add Swagger documentation
6. Update index files

### Phase 3 (Final)
1. Integrate scheduler
2. Create email templates
3. Add comprehensive testing
4. Deploy and monitor

---

**Current Status:** 📦 Models Complete, Service Structure Ready  
**Remaining Work:** Controllers, Validations, Routes, Integration  
**Estimated Time:** 4-6 hours for full completion

Would you like me to continue with Phase 2 to complete the controllers, validations, and routes?

