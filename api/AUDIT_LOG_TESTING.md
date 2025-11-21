# Audit Log Testing Guide

## Prerequisites

1. **Start the server:**
```bash
yarn dev
```

2. **Login to get auth token:**
```bash
POST http://localhost:2323/v1/auth/login
Content-Type: application/json

{
  "email": "superadmin@bridgebond.com",
  "password": "Super@123"
}

# Save the access token from response
```

## API Endpoints for Viewing Logs

### 1. Get All Audit Logs

```bash
GET http://localhost:2323/v1/audit-logs
Authorization: Bearer YOUR_ACCESS_TOKEN

# With pagination
GET http://localhost:2323/v1/audit-logs?page=1&limit=20&sortBy=createdAt:desc
```

**Response Example:**
```json
{
  "results": [
    {
      "_id": "64a7f89b1c2d3e4f5a6b7c8d",
      "userId": "64a7f89b1c2d3e4f5a6b7c8e",
      "userEmail": "user@example.com",
      "action": "LOGIN",
      "resource": "Auth",
      "method": "POST",
      "endpoint": "/v1/auth/login",
      "statusCode": 200,
      "ipAddress": "::1",
      "userAgent": "Mozilla/5.0...",
      "duration": 234,
      "success": true,
      "createdAt": "2024-10-28T15:30:45.123Z"
    }
  ],
  "page": 1,
  "limit": 20,
  "totalPages": 5,
  "totalResults": 98
}
```

### 2. Filter by Action Type

```bash
# View all logins
GET http://localhost:2323/v1/audit-logs?action=LOGIN
Authorization: Bearer YOUR_ACCESS_TOKEN

# View all notifications sent
GET http://localhost:2323/v1/audit-logs?action=NOTIFICATION_SENT
Authorization: Bearer YOUR_ACCESS_TOKEN

# View all user creations
GET http://localhost:2323/v1/audit-logs?action=USER_CREATE
Authorization: Bearer YOUR_ACCESS_TOKEN
```

### 3. View User Activity

```bash
GET http://localhost:2323/v1/audit-logs/user/64a7f89b1c2d3e4f5a6b7c8d
Authorization: Bearer YOUR_ACCESS_TOKEN

# With date range
GET http://localhost:2323/v1/audit-logs/user/64a7f89b1c2d3e4f5a6b7c8d?startDate=2024-10-01&endDate=2024-10-28
```

**Response Example:**
```json
[
  {
    "_id": "...",
    "action": "LOGIN",
    "endpoint": "/v1/auth/login",
    "statusCode": 200,
    "ipAddress": "192.168.1.100",
    "createdAt": "2024-10-28T09:15:23.456Z"
  },
  {
    "_id": "...",
    "action": "USER_UPDATE",
    "endpoint": "/v1/users/64a7f89b1c2d3e4f5a6b7c8d",
    "statusCode": 200,
    "changes": {
      "firstName": { "from": "John", "to": "Johnny" }
    },
    "createdAt": "2024-10-28T10:22:11.789Z"
  }
]
```

### 4. View Login History

```bash
GET http://localhost:2323/v1/audit-logs/user/64a7f89b1c2d3e4f5a6b7c8d/login-history
Authorization: Bearer YOUR_ACCESS_TOKEN
```

**Response Example:**
```json
[
  {
    "_id": "...",
    "action": "LOGIN",
    "ipAddress": "192.168.1.100",
    "userAgent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7)",
    "success": true,
    "createdAt": "2024-10-28T09:15:23.456Z"
  },
  {
    "_id": "...",
    "action": "LOGOUT",
    "ipAddress": "192.168.1.100",
    "success": true,
    "createdAt": "2024-10-28T18:45:12.123Z"
  }
]
```

### 5. View Resource History

```bash
# View who modified a specific user
GET http://localhost:2323/v1/audit-logs/resource/User/64a7f89b1c2d3e4f5a6b7c8d
Authorization: Bearer YOUR_ACCESS_TOKEN

# View celebration history
GET http://localhost:2323/v1/audit-logs/resource/Celebration/64a7f89b1c2d3e4f5a6b7c8e
Authorization: Bearer YOUR_ACCESS_TOKEN
```

**Response Example:**
```json
[
  {
    "_id": "...",
    "userId": {
      "_id": "...",
      "firstName": "Admin",
      "lastName": "User",
      "email": "admin@example.com"
    },
    "action": "USER_UPDATE",
    "oldValues": {
      "firstName": "John",
      "email": "john@old.com"
    },
    "newValues": {
      "firstName": "Johnny",
      "email": "johnny@new.com"
    },
    "changes": {
      "firstName": { "from": "John", "to": "Johnny" },
      "email": { "from": "john@old.com", "to": "johnny@new.com" }
    },
    "createdAt": "2024-10-28T10:22:11.789Z"
  }
]
```

### 6. View Failed Actions

```bash
GET http://localhost:2323/v1/audit-logs/failed
Authorization: Bearer YOUR_ACCESS_TOKEN

# With date range
GET http://localhost:2323/v1/audit-logs/failed?startDate=2024-10-01&endDate=2024-10-28
```

**Response Example:**
```json
[
  {
    "_id": "...",
    "action": "LOGIN",
    "endpoint": "/v1/auth/login",
    "statusCode": 401,
    "errorMessage": "Incorrect email or password",
    "success": false,
    "ipAddress": "192.168.1.200",
    "createdAt": "2024-10-28T08:30:15.456Z"
  },
  {
    "_id": "...",
    "action": "NOTIFICATION_SENT",
    "resource": "Celebration",
    "errorMessage": "OneSignal API error: Invalid player_id",
    "success": false,
    "createdAt": "2024-10-28T09:00:23.789Z"
  }
]
```

### 7. View System Statistics

```bash
GET http://localhost:2323/v1/audit-logs/stats
Authorization: Bearer YOUR_ACCESS_TOKEN

# With date range
GET http://localhost:2323/v1/audit-logs/stats?startDate=2024-10-01&endDate=2024-10-28
```

**Response Example:**
```json
{
  "totalRequests": 15420,
  "successfulRequests": 14890,
  "failedRequests": 530,
  "successRate": "96.56",
  "uniqueUsers": 245,
  "topActions": [
    { "_id": "LOGIN", "count": 3450 },
    { "_id": "USER_VIEW", "count": 2890 },
    { "_id": "USER_UPDATE", "count": 1234 },
    { "_id": "NOTIFICATION_SENT", "count": 987 },
    { "_id": "USER_CREATE", "count": 654 }
  ],
  "resourceBreakdown": [
    { "_id": "User", "count": 8900 },
    { "_id": "Auth", "count": 3500 },
    { "_id": "Organization", "count": 2100 },
    { "_id": "Celebration", "count": 1200 }
  ]
}
```

## Postman Collection

### Setup

1. **Create Environment:**
   - Variable: `baseUrl` = `http://localhost:2323`
   - Variable: `accessToken` = (will be set after login)

2. **Login Request:**
```
POST {{baseUrl}}/v1/auth/login
Content-Type: application/json

{
  "email": "superadmin@bridgebond.com",
  "password": "Super@123"
}
```

**Tests (auto-save token):**
```javascript
if (pm.response.code === 200) {
    var jsonData = pm.response.json();
    pm.environment.set("accessToken", jsonData.tokens.access.token);
}
```

3. **All Other Requests:**
```
Authorization: Bearer {{accessToken}}
```

## cURL Examples

```bash
# 1. Login
curl -X POST http://localhost:2323/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "superadmin@bridgebond.com",
    "password": "Super@123"
  }'

# Save the token from response
TOKEN="your_access_token_here"

# 2. View all audit logs
curl -X GET http://localhost:2323/v1/audit-logs \
  -H "Authorization: Bearer $TOKEN"

# 3. View recent logins
curl -X GET "http://localhost:2323/v1/audit-logs?action=LOGIN&limit=10" \
  -H "Authorization: Bearer $TOKEN"

# 4. View user activity
curl -X GET http://localhost:2323/v1/audit-logs/user/USER_ID_HERE \
  -H "Authorization: Bearer $TOKEN"

# 5. View failed actions
curl -X GET http://localhost:2323/v1/audit-logs/failed \
  -H "Authorization: Bearer $TOKEN"

# 6. View system stats
curl -X GET http://localhost:2323/v1/audit-logs/stats \
  -H "Authorization: Bearer $TOKEN"

# 7. View notifications sent today
curl -X GET "http://localhost:2323/v1/audit-logs?action=NOTIFICATION_SENT&sortBy=createdAt:desc" \
  -H "Authorization: Bearer $TOKEN"
```

## Common Use Cases

### Track a Specific User's Activity
```bash
# Get user ID first
GET /v1/users?email=user@example.com

# Then get their activity
GET /v1/audit-logs/user/{userId}

# Or their login history
GET /v1/audit-logs/user/{userId}/login-history
```

### Monitor Notification Delivery
```bash
# View all notifications sent today
GET /v1/audit-logs?action=NOTIFICATION_SENT&startDate=2024-10-28

# View failed notifications
GET /v1/audit-logs?action=NOTIFICATION_SENT&success=false
```

### Security Audit
```bash
# View all failed login attempts
GET /v1/audit-logs?action=LOGIN&success=false

# View all password reset requests
GET /v1/audit-logs?action=PASSWORD_RESET_REQUEST

# View all user deletions
GET /v1/audit-logs?action=USER_DELETE
```

### Performance Monitoring
```bash
# Get system stats for the month
GET /v1/audit-logs/stats?startDate=2024-10-01&endDate=2024-10-31

# Find slow requests (will need custom query in MongoDB)
db.auditlogs.find({ duration: { $gt: 1000 } }).sort({ duration: -1 })
```

## Available Action Types

```javascript
// Auth actions
'LOGIN', 'LOGOUT', 'REGISTER', 'PASSWORD_RESET_REQUEST', 
'PASSWORD_RESET_CONFIRM', 'EMAIL_VERIFY', 'TOKEN_REFRESH'

// User actions
'USER_CREATE', 'USER_UPDATE', 'USER_DELETE', 'USER_SOFT_DELETE', 
'USER_RESTORE', 'USER_VIEW'

// Organization actions
'ORGANIZATION_CREATE', 'ORGANIZATION_UPDATE', 'ORGANIZATION_DELETE', 
'ORGANIZATION_SOFT_DELETE', 'ORGANIZATION_RESTORE'

// Department actions
'DEPARTMENT_CREATE', 'DEPARTMENT_UPDATE', 'DEPARTMENT_DELETE', 
'DEPARTMENT_SOFT_DELETE', 'DEPARTMENT_RESTORE'

// Celebration actions
'CELEBRATION_CREATE', 'CELEBRATION_UPDATE', 'CELEBRATION_DELETE', 
'CELEBRATION_SOFT_DELETE', 'CELEBRATION_RESTORE'

// DOB Alert actions
'DOB_ALERT_CREATE', 'DOB_ALERT_UPDATE', 'DOB_ALERT_DELETE', 
'DOB_ALERT_SOFT_DELETE', 'DOB_ALERT_RESTORE'

// Notification actions
'NOTIFICATION_SENT', 'EMAIL_SENT', 'SMS_SENT'

// System actions
'SYSTEM_ERROR', 'SYSTEM_WARNING', 'API_ERROR'
```

## Available Resource Types

```javascript
'User', 'Organization', 'Department', 'Celebration', 
'DobAlert', 'Auth', 'System', 'Notification'
```

## Tips

1. **Use pagination** for large result sets:
   ```
   ?page=1&limit=50
   ```

2. **Sort by date** (newest first):
   ```
   ?sortBy=createdAt:desc
   ```

3. **Combine filters**:
   ```
   ?action=LOGIN&success=false&startDate=2024-10-01
   ```

4. **Check scheduler logs** in console when notifications run at 9 AM daily

5. **View real-time logs** by keeping terminal open while server runs

## Troubleshooting

### No logs appearing?
1. Make sure audit middleware is active (check `src/app.js`)
2. Check MongoDB connection
3. Verify requests are going through the API

### Can't access audit logs?
1. Make sure you're logged in as superadmin
2. Check authorization header is included
3. Verify token hasn't expired

### Notifications not logged?
1. Check scheduler is running (should see in console at startup)
2. Verify OneSignal is configured
3. Check for errors in server logs

## Next Steps

Consider building a dashboard to visualize these logs with charts and real-time updates!

