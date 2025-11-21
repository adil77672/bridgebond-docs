# Password Reset Flow (Two-Step Process)

## Overview

The password reset flow is now a **two-step process** for better UX:

1. **Step 1**: Verify OTP code (get reset token)
2. **Step 2**: Reset password using the reset token

## Step-by-Step Flow

### Step 1: Verify OTP Code

**Endpoint**: `POST /v1/otp/verify`

**Request**:
```json
{
  "email": "user@example.com",
  "code": "1234",
  "type": "password_reset"
}
```

**Response** (when `type: "password_reset"`):
```json
{
  "message": "OTP verified successfully. Please proceed to reset your password.",
  "resetToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expiresAt": "2025-11-05T10:15:00.000Z"
}
```

**Important**: 
- Save the `resetToken` from the response
- This token is valid for **10 minutes**
- Use this token in Step 2

### Step 2: Reset Password

**Endpoint**: `POST /v1/otp/reset-password`

**Request**:
```json
{
  "resetToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "password": "NewPassword@123",
  "confirmPassword": "NewPassword@123"
}
```

**Response**:
```json
{
  "message": "Password reset successful"
}
```

## Complete Example (cURL)

### Step 1: Verify OTP
```bash
curl 'http://localhost:2323/v1/otp/verify' \
  -H 'Content-Type: application/json' \
  --data-raw '{
    "email": "user@example.com",
    "code": "1234",
    "type": "password_reset"
  }'
```

**Response**:
```json
{
  "message": "OTP verified successfully. Please proceed to reset your password.",
  "resetToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI2OTAyMWM1NGNmNjA1YjZlM2UwNTNjNjUiLCJpYXQiOjE3NjIzMzIwOTQsImV4cCI6MTc2MjMzMzg5NCwidHlwZSI6InJlc2V0UGFzc3dvcmQifQ...",
  "expiresAt": "2025-11-05T10:15:00.000Z"
}
```

### Step 2: Reset Password
```bash
curl 'http://localhost:2323/v1/otp/reset-password' \
  -H 'Content-Type: application/json' \
  --data-raw '{
    "resetToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI2OTAyMWM1NGNmNjA1YjZlM2UwNTNjNjUiLCJpYXQiOjE3NjIzMzIwOTQsImV4cCI6MTc2MjMzMzg5NCwidHlwZSI6InJlc2V0UGFzc3dvcmQifQ...",
    "password": "NewPassword@123",
    "confirmPassword": "NewPassword@123"
  }'
```

**Response**:
```json
{
  "message": "Password reset successful"
}
```

## Frontend Implementation

### React Example

```javascript
// Step 1: Verify OTP
const verifyOTP = async (email, code) => {
  const response = await fetch('http://localhost:2323/v1/otp/verify', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      email,
      code,
      type: 'password_reset',
    }),
  });
  
  const data = await response.json();
  
  if (response.ok) {
    // Save reset token for Step 2
    localStorage.setItem('resetToken', data.resetToken);
    return data;
  } else {
    throw new Error(data.message);
  }
};

// Step 2: Reset Password
const resetPassword = async (password, confirmPassword) => {
  const resetToken = localStorage.getItem('resetToken');
  
  if (!resetToken) {
    throw new Error('Reset token not found. Please verify OTP first.');
  }
  
  const response = await fetch('http://localhost:2323/v1/otp/reset-password', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      resetToken,
      password,
      confirmPassword,
    }),
  });
  
  const data = await response.json();
  
  if (response.ok) {
    // Clear reset token after successful reset
    localStorage.removeItem('resetToken');
    return data;
  } else {
    throw new Error(data.message);
  }
};
```

## Error Handling

### Step 1 Errors
- `400` - Invalid OTP code
- `400` - OTP expired
- `400` - Maximum attempts exceeded
- `404` - User not found

### Step 2 Errors
- `400` - Passwords do not match
- `400` - Invalid password format
- `401` - Invalid or expired reset token
- `404` - User not found

## Security Notes

1. **Reset Token Expiration**: Reset tokens are valid for **10 minutes** only
2. **One-Time Use**: Each OTP can only be used once
3. **Token Validation**: Reset tokens are JWT-based and cryptographically signed
4. **Password Requirements**: Must meet password validation rules (min 8 chars, contains letter and number)

## UI Flow Recommendations

### Page 1: Enter Email
- User enters email
- Click "Send OTP"
- OTP is sent to email

### Page 2: Verify OTP
- User enters OTP code received via email
- Click "Verify OTP"
- On success: Save `resetToken` and navigate to password reset page

### Page 3: Reset Password
- User enters new password
- User confirms new password
- Click "Reset Password"
- On success: Redirect to login page

