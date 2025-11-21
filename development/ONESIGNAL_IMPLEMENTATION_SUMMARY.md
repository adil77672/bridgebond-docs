# OneSignal Implementation Summary

## Overview
This document summarizes the OneSignal push notification implementation updates made to use the `onesignal-node` package (v3.x) instead of `@onesignal/node-onesignal` (v5.x).

## Changes Made

### 1. Package Dependencies (`package.json`)
- **Replaced**: `@onesignal/node-onesignal` (v5.3.1-beta1) 
- **With**: `onesignal-node` (v3.4.0)
- **Reason**: The v3.x package has a simpler API that matches your working test script

**Action Required**: Run `npm install` or `yarn install` to install the new package.

### 2. OneSignal Configuration (`src/config/onesignal.js`)
Updated to use the simpler `onesignal-node` client:

```javascript
import OneSignal from 'onesignal-node';

const client = new OneSignal.Client(
  config.notification.onesignal.appId,
  config.notification.onesignal.restApiKey
);
```

### 3. Notification Service (`src/services/notification.service.js`)
- Removed the `@onesignal/node-onesignal` import
- Updated `sendOneSignalNotification` to use plain JavaScript objects:
  ```javascript
  const notification = {
    contents: { en: content },
    headings: { en: heading },
    include_player_ids: playerIds,
    data: { ...data, notificationTime: new Date().toISOString() },
  };
  ```
- Simplified all notification functions to use the centralized `sendPushNotification` helper

### 4. Device Controller (`src/controllers/device.controller.js`)
Added comprehensive player ID management:

#### New Features:
- **`cleanupPlayerIds(userId)`**: Validates player IDs against OneSignal API
  - Checks if subscriptions are still enabled
  - Removes invalid/disabled player IDs automatically
  - Runs in background after device registration

- **Enhanced `registerDevice`**: 
  - Uses `$addToSet` to prevent duplicate player IDs
  - Triggers automatic cleanup after registration
  - Better error handling and validation

#### How Cleanup Works:
1. Gets all player IDs for a user
2. For each player ID:
   - Fetches OneSignal user identity
   - Checks if subscription is still enabled
   - Keeps only valid, enabled subscriptions
3. Updates user document with clean list

### 5. Test Script (`testNotification.js`)
Created a standalone test script based on your working example:

```bash
# Run the test
npm run test-notification

# Or directly
node testNotification.js
```

**Before running**: Update the `testPlayerIds` array with a valid player ID from your OneSignal dashboard.

## Environment Variables Required

Ensure these are set in your `.env` file:

```env
# Notification Provider
NOTIFICATION_PROVIDER=onesignal

# OneSignal Configuration
ONESIGNAL_APP_ID=your-app-id-here
ONESIGNAL_REST_API_KEY=your-rest-api-key-here
ONESIGNAL_USER_AUTH_KEY=your-user-auth-key-here (optional)
```

## User Model Structure

The user model already has the `oneSignalPlayerIds` field:

```javascript
{
  oneSignalPlayerIds: {
    type: [String],
    default: [],
    description: 'Array of OneSignal player IDs for multiple devices',
  }
}
```

## API Endpoints

### Register Device
```http
POST /v1/devices/register
Authorization: Bearer <token>
Content-Type: application/json

{
  "playerId": "f9329032-aa6d-43fd-8ba9-55613b3a44ab"
}
```

### Unregister Device
```http
DELETE /v1/devices/:playerId
Authorization: Bearer <token>
```

### Get All Devices
```http
GET /v1/devices
Authorization: Bearer <token>
```

### Remove All Devices
```http
DELETE /v1/devices
Authorization: Bearer <token>
```

## Notification Types Supported

1. **Birthday Notifications**: When it's someone's birthday
2. **Anniversary Notifications**: Work anniversary celebrations
3. **Upcoming Birthday Reminders**: 1+ days before birthday
4. **Upcoming Anniversary Reminders**: 1+ days before anniversary
5. **Reaction Notifications**: When someone reacts to content
6. **New Question Notifications**: When a new question is posted

## How to Send Notifications

### From Your Code:
```javascript
import notificationService from '../services/notification.service.js';

// Send birthday notification
await notificationService.sendBirthdayNotification(
  recipientUser,  // User receiving notification
  celebrationUser // User whose birthday it is
);

// Send custom notification (using the helper directly)
const playerIds = user.oneSignalPlayerIds;
const heading = 'Custom Notification';
const content = 'This is a custom message';
const data = { type: 'custom', userId: user.id };

await sendPushNotification(playerIds, heading, content, data);
```

### Using the Test Script:
```javascript
// Edit testNotification.js
const testPlayerIds = ['your-player-id-here']; // or [] for all users

await sendNotification({
  playerIds: testPlayerIds,
  message: 'Your message here',
  heading: 'Your title here',
  data: { customKey: 'customValue' },
});
```

## Installation Steps

1. **Install new package**:
   ```bash
   npm install
   # or
   yarn install
   ```

2. **Verify environment variables**:
   ```bash
   # Check your .env file has OneSignal credentials
   cat .env | grep ONESIGNAL
   ```

3. **Test the notification system**:
   ```bash
   # Update testPlayerIds in testNotification.js first
   npm run test-notification
   ```

4. **Start your server**:
   ```bash
   npm run dev
   ```

## Troubleshooting

### "OneSignal credentials missing" Warning
- Check that `ONESIGNAL_APP_ID` and `ONESIGNAL_REST_API_KEY` are set in `.env`
- Restart your server after updating environment variables

### Notifications Not Received
1. Verify the player ID is correct (check OneSignal dashboard)
2. Check that the user has the player ID saved: `GET /v1/devices`
3. Look at server logs for error messages
4. Test with the standalone script first: `npm run test-notification`

### Cleanup Not Working
- Ensure `axios` is installed (already in dependencies)
- Check OneSignal API credentials are valid
- Look for warning/error logs in the console
- The cleanup runs asynchronously and won't block registration

## Benefits of This Implementation

1. ✅ **Simpler API**: Plain JavaScript objects instead of complex SDK classes
2. ✅ **Automatic Cleanup**: Invalid player IDs are automatically removed
3. ✅ **No Duplicates**: Uses `$addToSet` to prevent duplicate registrations
4. ✅ **Better Error Handling**: Comprehensive error logging and recovery
5. ✅ **Background Processing**: Cleanup doesn't block the registration response
6. ✅ **Flexible Targeting**: Send to specific users or all subscribed users
7. ✅ **Easy Testing**: Standalone test script for quick verification

## Next Steps

1. Install the new package: `npm install`
2. Test with the test script: `npm run test-notification`
3. Test device registration via API endpoints
4. Monitor logs for any issues
5. Deploy to your staging environment first

## Files Modified

- ✏️ `package.json` - Updated OneSignal dependency
- ✏️ `src/config/onesignal.js` - Simplified client initialization
- ✏️ `src/services/notification.service.js` - Updated notification sending logic
- ✏️ `src/controllers/device.controller.js` - Added cleanup functionality
- ➕ `testNotification.js` - New test script
- ➕ `ONESIGNAL_IMPLEMENTATION_SUMMARY.md` - This documentation

## Support

If you encounter any issues:
1. Check the server logs for error messages
2. Verify OneSignal credentials in dashboard
3. Test with a known valid player ID
4. Check OneSignal dashboard for delivery status

---

**Last Updated**: October 29, 2025

