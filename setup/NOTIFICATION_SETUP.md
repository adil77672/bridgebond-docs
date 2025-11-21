# Notification Provider Configuration

## Overview

The system supports multiple notification providers configured via environment variables.

## Supported Providers

- **OneSignal** (default) - Push notifications
- **FCM** (Firebase Cloud Messaging) - Coming soon
- **none** - Disable push notifications

## Environment Variables

### Required for OneSignal

```env
# Notification Provider (onesignal, fcm, or none)
NOTIFICATION_PROVIDER=onesignal

# OneSignal Credentials
ONESIGNAL_APP_ID=your-onesignal-app-id
ONESIGNAL_REST_API_KEY=your-rest-api-key
ONESIGNAL_USER_AUTH_KEY=your-user-auth-key
```

### Optional - Firebase Cloud Messaging (FCM)

```env
NOTIFICATION_PROVIDER=fcm
FCM_SERVER_KEY=your-fcm-server-key
```

### Disable Notifications

```env
NOTIFICATION_PROVIDER=none
```

## OneSignal Setup

### 1. Create OneSignal Account
1. Go to [OneSignal.com](https://onesignal.com)
2. Sign up for free account
3. Create new app

### 2. Get Credentials

**App ID:**
- Dashboard → Settings → Keys & IDs
- Copy "OneSignal App ID"

**REST API Key:**
- Dashboard → Settings → Keys & IDs  
- Copy "REST API Key"

**User Auth Key:**
- Account Settings (top right) → Keys & IDs
- Copy "User Auth Key"

### 3. Add to Environment

```env
NOTIFICATION_PROVIDER=onesignal
ONESIGNAL_APP_ID=abcd1234-5678-90ef-ghij-klmnopqrstuv
ONESIGNAL_REST_API_KEY=MWFiY2RlZmc1234567890
ONESIGNAL_USER_AUTH_KEY=YzQ2MTU2MDAtM2Q3Yi00NjY3LTk1MjMtMGU5YzU3YzMwYjQx
```

### 4. Restart Server

```bash
yarn dev
# or
npm run dev
```

You should see:
```
📱 Notification Provider: ONESIGNAL
✅ OneSignal initialized successfully
```

## Email Configuration

Email still uses Nodemailer with SMTP:

```env
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_SERVICE=gmail
SMTP_USERNAME=your-email@gmail.com
SMTP_PASSWORD=your-app-password
EMAIL_FROM=noreply@yourdomain.com
```

## Notification Types

The system sends notifications for:

- 🎂 Birthdays
- 🎊 Work anniversaries
- 📅 Upcoming celebrations
- 👍 Reactions to posts
- ❓ New questions

## Testing

Check server logs:

```bash
# OneSignal enabled
📱 Notification Provider: ONESIGNAL
✅ OneSignal initialized successfully

# Notifications disabled
📱 Notification Provider: NONE
Push notifications disabled (provider=none)

# Missing credentials
⚠️  OneSignal credentials missing. Push notifications will not work.
```

## Switching Providers

Just change the environment variable:

```env
# Use OneSignal
NOTIFICATION_PROVIDER=onesignal

# Disable notifications
NOTIFICATION_PROVIDER=none

# Use FCM (when implemented)
NOTIFICATION_PROVIDER=fcm
```

No code changes needed!

## Troubleshooting

### "OneSignal credentials missing"
- Check all three credentials are set in `.env`
- Restart server after adding credentials

### "OneSignal initialization failed"
- Verify credentials are correct
- Check OneSignal dashboard is accessible
- Check API keys haven't expired

### No notifications received
- Ensure `NOTIFICATION_PROVIDER=onesignal`
- Check user has `oneSignalPlayerIds` in database
- Verify OneSignal app is configured for your platform (iOS/Android/Web)

## Production Deployment (Vercel)

Add to Vercel environment variables:

```
NOTIFICATION_PROVIDER=onesignal
ONESIGNAL_APP_ID=your-app-id
ONESIGNAL_REST_API_KEY=your-rest-key
ONESIGNAL_USER_AUTH_KEY=your-user-auth-key
```

## Resources

- [OneSignal Docs](https://documentation.onesignal.com/)
- [OneSignal Node.js SDK](https://github.com/OneSignal/onesignal-node-api)
- [Get OneSignal Keys](https://documentation.onesignal.com/docs/accounts-and-keys)

