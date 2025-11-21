# API Reference

## Base URL
```
Production: https://your-app.vercel.app/v1
Local: http://localhost:3000/v1
Docs: /v1/docs
```

## Authentication

All endpoints (except public) require Bearer token:
```
Authorization: Bearer <token>
```

## Endpoints

### Auth
- `POST /auth/register` - Register user
- `POST /auth/login` - Login
- `POST /auth/logout` - Logout
- `POST /auth/refresh-tokens` - Refresh access token

### OTP
- `POST /otp/send` - Send OTP (email_verification | password_reset)
- `POST /otp/verify` - Verify OTP

### Users
- `GET /users` - List users (paginated)
- `GET /users/:id` - Get user
- `POST /users` - Create user
- `PATCH /users/:id` - Update user
- `DELETE /users/:id` - Soft delete user
- `PATCH /users/:id/image` - Upload profile image

### Organizations
- `GET /organizations` - List organizations
- `GET /organizations/:id` - Get organization
- `POST /organizations` - Create organization
- `PATCH /organizations/:id` - Update organization
- `DELETE /organizations/:id` - Soft delete organization
- `PATCH /organizations/:id/logo` - Upload logo
- `GET /organizations/:id/users` - Get users in organization
- `GET /organizations/:id/departments` - Get departments

### Departments
- `GET /departments` - List departments
- `GET /departments/:id` - Get department
- `POST /departments` - Create department
- `PATCH /departments/:id` - Update department
- `DELETE /departments/:id` - Soft delete department
- `GET /departments/:id/users` - Get users in department

### Questions
- `GET /questions` - List questions
- `GET /questions/:id` - Get question
- `POST /questions` - Create question
- `PATCH /questions/:id` - Update question
- `DELETE /questions/:id` - Delete question
- `GET /questions/:id/responses` - Get responses
- `POST /questions/:id/responses` - Submit response

### Celebrations & Alerts
- `GET /celebrations` - List celebrations
- `POST /celebrations` - Create celebration
- `GET /dob-alerts` - List birthday alerts
- `POST /dob-alerts` - Create alert

## Query Parameters

### Pagination
```
?page=1&limit=10&sortBy=createdAt:desc
```

### Filtering
```
?name=John&role=user&isActive=true
```

### Population
```
?populate=[{"path":"department","populate":["organizationId"]}]
```

See [POPULATE_GUIDE.md](POPULATE_GUIDE.md) for details.

## Response Format

### Success
```json
{
  "results": [...],
  "page": 1,
  "limit": 10,
  "totalPages": 5,
  "totalResults": 50
}
```

### Error
```json
{
  "code": 400,
  "message": "Error description"
}
```

## Status Codes

- `200` - Success
- `201` - Created
- `204` - No Content
- `400` - Bad Request
- `401` - Unauthorized
- `403` - Forbidden
- `404` - Not Found
- `500` - Internal Server Error

## Environment Variables

See `ENVIRONMENT_VARIABLES.md` in root or Vercel dashboard.

## Complete Documentation

Visit `/v1/docs` for interactive Swagger documentation.

