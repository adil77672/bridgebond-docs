# Feature Guides

## Soft Delete

All major models support soft delete:

```javascript
// Soft delete (default)
await user.softDelete();

// Restore
await user.restore();

// Hard delete (permanent)
await User.hardDelete(userId);
```

Models: User, Organization, Department, Celebration, DobAlert

## Multi-Organization Users

Users can belong to multiple organizations with different profiles:

```javascript
{
  "firstName": "John",
  "lastName": "Doe",
  "email": "john@email.com",
  "organizationMemberships": [
    {
      "organizationId": "org1",
      "jobTitle": "Senior Engineer",
      "employeeId": "EMP-001",
      "role": "user"
    },
    {
      "organizationId": "org2",
      "jobTitle": "Consultant",
      "employeeId": "CON-100",
      "role": "org_admin"
    }
  ]
}
```

## OTP System

Unified endpoints:

```bash
# Send OTP
POST /v1/otp/send
{"email": "user@email.com", "type": "email_verification"}

# Verify OTP
POST /v1/otp/verify
{"email": "user@email.com", "code": "1234", "type": "email_verification"}
```

Types: `email_verification`, `password_reset`

## Image Upload

```bash
# Upload user profile image
PATCH /v1/users/:userId/image
-F "image=@photo.jpg"

# Upload organization logo
PATCH /v1/organizations/:orgId/logo
-F "logo=@logo.png"
```

- Max size: 10MB
- Auto-optimization
- Cloudinary CDN delivery

## Dynamic Populate

See [POPULATE_GUIDE.md](../api/POPULATE_GUIDE.md)

