# Dashboard API - cURL Examples & Responses

Complete cURL examples and response formats for all Dashboard endpoints.

---

## 📋 Table of Contents

1. [Admin Dashboard](#admin-dashboard)
2. [Dashboard Stats](#dashboard-stats)
3. [Weekly Activity](#weekly-activity)
4. [Celebrations](#celebrations)
5. [Department Engagement](#department-engagement)
6. [User Dashboard](#user-dashboard)
7. [Superadmin Dashboard](#superadmin-dashboard)

---

## 🔐 Authentication

All endpoints require Bearer token authentication. Get your token from `/v1/auth/login`:

```bash
# Login and get token
curl -X POST "http://localhost:3000/v1/auth/login" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "admin@example.com",
    "password": "Password123"
  }'
```

**Response:**
```json
{
  "user": {
    "id": "user123",
    "email": "admin@example.com",
    "role": "org_admin"
  },
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

---

## Admin Dashboard

### Get Complete Dashboard (Admin)

**Endpoint:** `GET /v1/dashboard`

**Description:** Get complete dashboard data including stats, weekly activity, celebrations, and department engagement.

**cURL:**
```bash
curl -X GET "http://localhost:3000/v1/dashboard?organizationId=org123&celebrationDate=2024-10-28" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**
```json
{
  "status": "success",
  "data": {
    "stats": {
      "totalUsers": 99,
      "activeToday": 34,
      "departments": 12,
      "reactions": 2223
    },
    "weeklyActivity": {
      "days": [
        {
          "day": "Mon",
          "date": "2024-10-21",
          "reactions": 15,
          "updates": 12,
          "answers": 8
        },
        {
          "day": "Tue",
          "date": "2024-10-22",
          "reactions": 18,
          "updates": 15,
          "answers": 10
        },
        {
          "day": "Wed",
          "date": "2024-10-23",
          "reactions": 12,
          "updates": 9,
          "answers": 6
        },
        {
          "day": "Thu",
          "date": "2024-10-24",
          "reactions": 22,
          "updates": 18,
          "answers": 14
        },
        {
          "day": "Fri",
          "date": "2024-10-25",
          "reactions": 25,
          "updates": 20,
          "answers": 16
        },
        {
          "day": "Sat",
          "date": "2024-10-26",
          "reactions": 8,
          "updates": 5,
          "answers": 3
        },
        {
          "day": "Sun",
          "date": "2024-10-27",
          "reactions": 10,
          "updates": 7,
          "answers": 5
        }
      ],
      "summary": {
        "totalReactions": 110,
        "totalUpdates": 86,
        "totalAnswers": 62
      }
    },
    "celebrations": [
      {
        "type": "birthday",
        "user": {
          "id": "user456",
          "firstName": "Sarah",
          "lastName": "Johnson",
          "imageUrl": "https://example.com/avatar.jpg",
          "department": "Home Office",
          "jobTitle": "Software Engineer"
        }
      },
      {
        "type": "anniversary",
        "yearsOfService": 2,
        "user": {
          "id": "user789",
          "firstName": "Michael",
          "lastName": "Chen",
          "imageUrl": "https://example.com/avatar2.jpg",
          "department": "HR Department",
          "jobTitle": "HR Manager"
        }
      }
    ],
    "departmentEngagement": [
      {
        "id": "dept1",
        "name": "Marketing",
        "engagement": 450,
        "reactions": 300,
        "answers": 150,
        "userCount": 25,
        "percentage": 40
      },
      {
        "id": "dept2",
        "name": "Engineering",
        "engagement": 280,
        "reactions": 200,
        "answers": 80,
        "userCount": 30,
        "percentage": 25
      },
      {
        "id": "dept3",
        "name": "Sales",
        "engagement": 225,
        "reactions": 150,
        "answers": 75,
        "userCount": 20,
        "percentage": 20
      },
      {
        "id": "dept4",
        "name": "HR",
        "engagement": 170,
        "reactions": 100,
        "answers": 70,
        "userCount": 15,
        "percentage": 15
      }
    ]
  }
}
```

---

## Dashboard Stats

### Get Dashboard Statistics

**Endpoint:** `GET /v1/dashboard/stats`

**Description:** Get key dashboard metrics (Total Users, Active Today, Departments, Reactions).

**cURL:**
```bash
curl -X GET "http://localhost:3000/v1/dashboard/stats?organizationId=org123" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**
```json
{
  "status": "success",
  "data": {
    "totalUsers": 99,
    "activeToday": 34,
    "departments": 12,
    "reactions": 2223
  }
}
```

---

## Weekly Activity

### Get Weekly Activity Overview

**Endpoint:** `GET /v1/dashboard/weekly-activity`

**Description:** Get weekly activity data (reactions, profile updates, question answers) for the past 7 days.

**cURL:**
```bash
curl -X GET "http://localhost:3000/v1/dashboard/weekly-activity?organizationId=org123" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**
```json
{
  "status": "success",
  "data": {
    "days": [
      {
        "day": "Mon",
        "date": "2024-10-21",
        "reactions": 15,
        "updates": 12,
        "answers": 8
      },
      {
        "day": "Tue",
        "date": "2024-10-22",
        "reactions": 18,
        "updates": 15,
        "answers": 10
      },
      {
        "day": "Wed",
        "date": "2024-10-23",
        "reactions": 12,
        "updates": 9,
        "answers": 6
      },
      {
        "day": "Thu",
        "date": "2024-10-24",
        "reactions": 22,
        "updates": 18,
        "answers": 14
      },
      {
        "day": "Fri",
        "date": "2024-10-25",
        "reactions": 25,
        "updates": 20,
        "answers": 16
      },
      {
        "day": "Sat",
        "date": "2024-10-26",
        "reactions": 8,
        "updates": 5,
        "answers": 3
      },
      {
        "day": "Sun",
        "date": "2024-10-27",
        "reactions": 10,
        "updates": 7,
        "answers": 5
      }
    ],
    "summary": {
      "totalReactions": 110,
      "totalUpdates": 86,
      "totalAnswers": 62
    }
  }
}
```

---

## Celebrations

### Get Celebrations (Birthdays and Work Anniversaries)

**Endpoint:** `GET /v1/dashboard/celebrations`

**Description:** Get birthdays and work anniversaries for a specific date.

**cURL:**
```bash
# Get today's celebrations
curl -X GET "http://localhost:3000/v1/dashboard/celebrations?organizationId=org123" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json"

# Get celebrations for specific date
curl -X GET "http://localhost:3000/v1/dashboard/celebrations?organizationId=org123&date=2024-10-28" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**
```json
{
  "status": "success",
  "results": 4,
  "data": [
    {
      "type": "birthday",
      "user": {
        "id": "user456",
        "firstName": "Sarah",
        "lastName": "Johnson",
        "imageUrl": "https://example.com/avatar.jpg",
        "department": "Home Office",
        "jobTitle": "Software Engineer"
      }
    },
    {
      "type": "anniversary",
      "yearsOfService": 2,
      "user": {
        "id": "user789",
        "firstName": "Michael",
        "lastName": "Chen",
        "imageUrl": "https://example.com/avatar2.jpg",
        "department": "HR Department",
        "jobTitle": "HR Manager"
      }
    },
    {
      "type": "birthday",
      "user": {
        "id": "user101",
        "firstName": "Emily",
        "lastName": "Rodriguez",
        "imageUrl": "https://example.com/avatar3.jpg",
        "department": "Admin",
        "jobTitle": "Administrative Assistant"
      }
    },
    {
      "type": "anniversary",
      "yearsOfService": 3,
      "user": {
        "id": "user202",
        "firstName": "David",
        "lastName": "Kim",
        "imageUrl": "https://example.com/avatar4.jpg",
        "department": "Home Office",
        "jobTitle": "Product Manager"
      }
    }
  ]
}
```

---

## Department Engagement

### Get Department Engagement Statistics

**Endpoint:** `GET /v1/dashboard/department-engagement`

**Description:** Get engagement distribution across departments (based on reactions and question answers).

**cURL:**
```bash
curl -X GET "http://localhost:3000/v1/dashboard/department-engagement?organizationId=org123" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**
```json
{
  "status": "success",
  "results": 5,
  "data": [
    {
      "id": "dept1",
      "name": "Marketing",
      "engagement": 450,
      "reactions": 300,
      "answers": 150,
      "userCount": 25,
      "percentage": 40
    },
    {
      "id": "dept2",
      "name": "Engineering",
      "engagement": 280,
      "reactions": 200,
      "answers": 80,
      "userCount": 30,
      "percentage": 25
    },
    {
      "id": "dept3",
      "name": "Sales",
      "engagement": 225,
      "reactions": 150,
      "answers": 75,
      "userCount": 20,
      "percentage": 20
    },
    {
      "id": "dept4",
      "name": "HR",
      "engagement": 170,
      "reactions": 100,
      "answers": 70,
      "userCount": 15,
      "percentage": 15
    },
    {
      "id": "dept5",
      "name": "Finance",
      "engagement": 56,
      "reactions": 30,
      "answers": 26,
      "userCount": 9,
      "percentage": 5
    }
  ]
}
```

---

## User Dashboard

### Get User Dashboard (Personal Dashboard)

**Endpoint:** `GET /v1/dashboard/user`

**Description:** Get personal dashboard data for a regular user. Shows user's personal stats, upcoming celebrations, and recent activity.

**cURL:**
```bash
curl -X GET "http://localhost:3000/v1/dashboard/user?organizationId=org123" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**
```json
{
  "status": "success",
  "data": {
    "user": {
      "id": "user123",
      "firstName": "John",
      "lastName": "Doe",
      "email": "john.doe@example.com",
      "imageUrl": "https://example.com/avatar.jpg",
      "jobTitle": "Software Engineer",
      "department": "Engineering"
    },
    "upcomingCelebrations": [
      {
        "type": "birthday",
        "date": "2024-11-15T00:00:00.000Z",
        "daysUntil": 18
      },
      {
        "type": "anniversary",
        "date": "2024-12-01T00:00:00.000Z",
        "daysUntil": 34,
        "yearsOfService": 5
      }
    ],
    "recentActivity": {
      "reactions": 12,
      "answers": 8,
      "period": "7 days"
    }
  }
}
```

---

## Superadmin Dashboard

### Get Superadmin Dashboard (System-wide Statistics)

**Endpoint:** `GET /v1/dashboard/superadmin`

**Description:** Get system-wide dashboard data for superadmin. Shows total organizations, total users, active users, suspended organizations, monthly signups, daily active users chart, monthly signup trends, and recent activity.

**Note:** Superadmin only endpoint.

**Optional Query Parameters:**
- `organizationId` - Filter daily active users by organization
- `date` - Filter daily active users by specific date (YYYY-MM-DD format)
- `year` - Filter monthly signup trend by year (YYYY format, defaults to last 12 months)

**cURL Examples:**

```bash
# Get complete dashboard (all data)
curl -X GET "http://localhost:3000/v1/dashboard/superadmin" \
  -H "Authorization: Bearer YOUR_SUPERADMIN_ACCESS_TOKEN" \
  -H "Content-Type: application/json"

# Filter daily active users by organization
curl -X GET "http://localhost:3000/v1/dashboard/superadmin?organizationId=org123" \
  -H "Authorization: Bearer YOUR_SUPERADMIN_ACCESS_TOKEN" \
  -H "Content-Type: application/json"

# Filter daily active users by specific date
curl -X GET "http://localhost:3000/v1/dashboard/superadmin?date=2024-10-28" \
  -H "Authorization: Bearer YOUR_SUPERADMIN_ACCESS_TOKEN" \
  -H "Content-Type: application/json"

# Filter monthly signup trend by year
curl -X GET "http://localhost:3000/v1/dashboard/superadmin?year=2024" \
  -H "Authorization: Bearer YOUR_SUPERADMIN_ACCESS_TOKEN" \
  -H "Content-Type: application/json"

# Combine filters (organization + date for daily chart)
curl -X GET "http://localhost:3000/v1/dashboard/superadmin?organizationId=org123&date=2024-10-28" \
  -H "Authorization: Bearer YOUR_SUPERADMIN_ACCESS_TOKEN" \
  -H "Content-Type: application/json"
```

**Response (200 OK):**
```json
{
  "status": "success",
  "data": {
    "stats": {
      "totalOrganizations": 147,
      "totalUsers": 23453,
      "activeUsers": 23453,
      "suspendedOrgs": 147,
      "monthlySignups": 147
    },
    "dailyActiveUsers": [
      {
        "time": "2024-10-28 08:00",
        "count": 5
      },
      {
        "time": "2024-10-28 09:00",
        "count": 12
      },
      {
        "time": "2024-10-28 10:00",
        "count": 18
      },
      {
        "time": "2024-10-28 11:00",
        "count": 22
      },
      {
        "time": "2024-10-28 12:00",
        "count": 25
      },
      {
        "time": "2024-10-28 13:00",
        "count": 28
      },
      {
        "time": "2024-10-28 14:00",
        "count": 30
      },
      {
        "time": "2024-10-28 15:00",
        "count": 32
      },
      {
        "time": "2024-10-28 16:00",
        "count": 35
      },
      {
        "time": "2024-10-28 17:00",
        "count": 19
      },
      {
        "time": "2024-10-28 18:00",
        "count": 15
      },
      {
        "time": "2024-10-28 19:00",
        "count": 10
      },
      {
        "time": "2024-10-28 20:00",
        "count": 8
      },
      {
        "time": "2024-10-28 21:00",
        "count": 5
      },
      {
        "time": "2024-10-28 22:00",
        "count": 3
      }
    ],
    "monthlySignupTrend": [
      {
        "month": "2023-11",
        "count": 10
      },
      {
        "month": "2023-12",
        "count": 12
      },
      {
        "month": "2024-01",
        "count": 15
      },
      {
        "month": "2024-02",
        "count": 18
      },
      {
        "month": "2024-03",
        "count": 20
      },
      {
        "month": "2024-04",
        "count": 22
      },
      {
        "month": "2024-05",
        "count": 25
      },
      {
        "month": "2024-06",
        "count": 28
      },
      {
        "month": "2024-07",
        "count": 30
      },
      {
        "month": "2024-08",
        "count": 32
      },
      {
        "month": "2024-09",
        "count": 35
      },
      {
        "month": "2024-10",
        "count": 15
      }
    ],
    "recentActivity": [
      {
        "id": "activity1",
        "icon": "🔐",
        "description": "John Doe logged into TechCorp Solutions",
        "timestamp": "2024-10-28T10:30:00.000Z",
        "timeAgo": "5 minutes ago",
        "action": "LOGIN",
        "resource": "Auth",
        "user": {
          "id": "user123",
          "email": "john.doe@techcorp.com",
          "firstName": "John",
          "lastName": "Doe",
          "role": "org_admin"
        },
        "organization": {
          "id": "org123",
          "name": "TechCorp Solutions"
        },
        "metadata": {}
      },
      {
        "id": "activity2",
        "icon": "🏢",
        "description": "New organization created: GlobalTech Inc by Jane Smith",
        "timestamp": "2024-10-28T08:00:00.000Z",
        "timeAgo": "2 hours ago",
        "action": "ORGANIZATION_CREATE",
        "resource": "Organization",
        "user": {
          "id": "user456",
          "email": "jane.smith@example.com",
          "firstName": "Jane",
          "lastName": "Smith",
          "role": "superadmin"
        },
        "organization": {
          "id": "org456",
          "name": "GlobalTech Inc"
        },
        "metadata": {}
      },
      {
        "id": "activity3",
        "icon": "👤",
        "description": "New user john.doe@example.com added to TechCorp Solutions by Admin User",
        "timestamp": "2024-10-28T09:00:00.000Z",
        "timeAgo": "1 hour ago",
        "action": "USER_CREATE",
        "resource": "User",
        "user": {
          "id": "user789",
          "email": "admin@techcorp.com",
          "firstName": "Admin",
          "lastName": "User",
          "role": "org_admin"
        },
        "organization": {
          "id": "org123",
          "name": "TechCorp Solutions"
        },
        "metadata": {}
      },
      {
        "id": "activity4",
        "icon": "📧",
        "description": "Weekly Digest sent to GlobalTech Inc (2,145 employees)",
        "timestamp": "2024-10-28T08:00:00.000Z",
        "timeAgo": "4 hours ago",
        "action": "WEEKLY_DIGEST",
        "resource": "Notification",
        "user": null,
        "organization": {
          "id": "org456",
          "name": "GlobalTech Inc"
        },
        "metadata": {
          "recipients": {
            "total": 2145,
            "sent": 2145,
            "failed": 0
          },
          "status": "sent"
        }
      },
      {
        "id": "activity5",
        "icon": "🔗",
        "description": "Workday sync completed for Innovation Labs (125 users synced)",
        "timestamp": "2024-10-28T06:00:00.000Z",
        "timeAgo": "6 hours ago",
        "action": "HRIS_SYNC",
        "resource": "HRIntegration",
        "user": null,
        "organization": {
          "id": "org789",
          "name": "Innovation Labs"
        },
        "metadata": {
          "platform": "workday",
          "status": "success",
          "syncedUsers": 125
        }
      }
    ]
  }
}
```

---

## Error Responses

### 401 Unauthorized
```json
{
  "status": "error",
  "message": "Please authenticate"
}
```

### 403 Forbidden
```json
{
  "status": "error",
  "message": "Access denied to this organization"
}
```

### 404 Not Found
```json
{
  "status": "error",
  "message": "Organization not found"
}
```

### 400 Bad Request
```json
{
  "status": "error",
  "message": "Organization ID is required"
}
```

---

## Frontend Integration Notes

### 1. **Token Management**
- Store the `accessToken` from login response
- Include it in the `Authorization` header for all requests: `Bearer YOUR_ACCESS_TOKEN`
- Token expires after a certain period, refresh using `/v1/auth/refresh-tokens`

### 2. **Organization Context**
- For Org Admin: `organizationId` is automatically set from the token
- For Superadmin: Can optionally specify `organizationId` in query params
- For User: Uses the organization from their token

### 3. **Date Formatting**
- All dates are in ISO 8601 format: `YYYY-MM-DD` or `YYYY-MM-DDTHH:mm:ss.sssZ`
- For celebrations: Use `date` query parameter in `YYYY-MM-DD` format

### 4. **Response Handling**
- All successful responses have `status: "success"`
- Check `status` field before processing data
- Error responses have `status: "error"` and `message` field

### 5. **Polling/Refresh**
- Dashboard data can be cached and refreshed periodically
- Recommended refresh interval: 5-10 minutes for stats
- Real-time updates may be added in future versions

### 6. **Superadmin Dashboard**
- Only accessible to users with `role: "superadmin"`
- Returns 403 Forbidden if accessed by non-superadmin users

---

## Quick Reference

### Base URL
```
Development: http://localhost:3000/v1
Production: https://api.bridgebond.techsufi.pro/v1
```

### All Dashboard Endpoints
```
GET /v1/dashboard                    # Complete dashboard (Admin)
GET /v1/dashboard/stats              # Dashboard statistics
GET /v1/dashboard/weekly-activity    # Weekly activity overview
GET /v1/dashboard/celebrations       # Birthdays & anniversaries
GET /v1/dashboard/department-engagement  # Department engagement
GET /v1/dashboard/user               # User personal dashboard
GET /v1/dashboard/superadmin          # Superadmin dashboard
```

---

## Example Frontend Code (JavaScript/Fetch)

```javascript
// Example: Fetch complete dashboard
async function getDashboard(organizationId, celebrationDate = null) {
  const url = new URL('http://localhost:3000/v1/dashboard');
  if (organizationId) url.searchParams.append('organizationId', organizationId);
  if (celebrationDate) url.searchParams.append('celebrationDate', celebrationDate);
  
  const response = await fetch(url, {
    method: 'GET',
    headers: {
      'Authorization': `Bearer ${accessToken}`,
      'Content-Type': 'application/json'
    }
  });
  
  const data = await response.json();
  return data;
}

// Example: Fetch superadmin dashboard
async function getSuperadminDashboard() {
  const response = await fetch('http://localhost:3000/v1/dashboard/superadmin', {
    method: 'GET',
    headers: {
      'Authorization': `Bearer ${superadminToken}`,
      'Content-Type': 'application/json'
    }
  });
  
  const data = await response.json();
  return data;
}
```

---

**Last Updated:** October 28, 2024

