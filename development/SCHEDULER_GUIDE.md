# Automated Notifications & Scheduler Guide

## Overview
The system includes an automated scheduler that runs background jobs to send notifications for birthdays and work anniversaries based on user preferences.

## Architecture

The notification system is built on three main components:

1. **User Preferences** (`UserSettings` model) - User-configurable notification settings
2. **Notification Service** - OneSignal integration for sending notifications
3. **Scheduler Service** - Cron job that processes and sends notifications

## Features

### ✅ User-Controlled Notification Preferences

Users have complete control over:
- **Who** they receive notifications for (all company, department, or selected users)
- **When** they receive notifications (customizable reminder time)
- **What** notifications they receive (birthdays, anniversaries, reactions, questions)
- **Timezone** preferences (auto-detected or manually set)

### ✅ Flexible Notification Scopes

**`getReminderFor` Options:**
- `all_company` - Receive notifications for all users in the organization
- `my_department` - Receive notifications only for department members
- `who_i_selected` - Receive notifications only for manually selected users

### ✅ Granular Control for Selected Users

When using `who_i_selected` mode, users can configure per-person:
```javascript
{
  userId: "user123",
  birthdayReminder: true,    // Receive birthday notifications
  anniversaryReminder: false // Don't receive anniversary notifications
}
```

### ✅ Automatic Timezone Detection

The system automatically detects and uses the user's current timezone for sending notifications.

**How it Works:**

1. **Client Auto-Detection (Recommended)**
   - The client (mobile app/browser) automatically detects the device's timezone
   - Sends timezone in `X-Timezone` header with every authenticated request (e.g., `Asia/Karachi`, `America/New_York`)
   - **Auth middleware** automatically:
     - Extracts timezone and attaches it to `req.userTimezone`
     - Updates the timezone in database if it has changed (fire-and-forget, no request delay)
     - Skips update if timezone is 'UTC' (fallback value)
   - **Benefit:** Timezone is always current - if user travels from Karachi to America, notifications automatically adjust to local time on their next API request

2. **Manual Override (Optional)**
   - User can manually set a specific timezone in request body
   - Use case: User wants notifications in home timezone even when traveling
   - Manual setting takes priority over auto-detected timezone

**Client Implementation Example:**

```javascript
// JavaScript/React Native - Auto-detect timezone
const timezone = Intl.DateTimeFormat().resolvedOptions().timeZone;
// Returns: "Asia/Karachi", "America/New_York", etc.

// Send with every request to update settings
fetch('/api/v1/user-settings', {
  method: 'PATCH',
  headers: {
    'Authorization': 'Bearer YOUR_TOKEN',
    'X-Timezone': timezone,  // Auto-detected
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    remindMeTime: "09:00"
  })
});

// Manual override example (lock to specific timezone)
fetch('/api/v1/user-settings', {
  method: 'PATCH',
  headers: {
    'Authorization': 'Bearer YOUR_TOKEN',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    remindMeTime: "09:00",
    timezone: "Asia/Karachi"  // Manual override
  })
});
```

**Priority Order:**
1. Manual `timezone` in request body (highest priority)
2. Auto-detected `X-Timezone` header
3. UTC fallback

**Performance Optimization:**
- Timezone updates run in the background (fire-and-forget pattern)
- Database updates only happen if timezone has actually changed
- No impact on API response time
- UTC fallback values are not saved to prevent unnecessary updates

**Example Scenario:**

```
User sets remindMeTime = "09:00" while in Pakistan (Asia/Karachi):
- X-Timezone header sent: "Asia/Karachi"
- Auth middleware auto-updates UserSettings timezone to: "Asia/Karachi"
- Notification sent at: 09:00 Pakistan time

Same user travels to America and makes any API request:
- X-Timezone header sent: "America/New_York" (auto-detected by device)
- Auth middleware detects timezone change
- UserSettings timezone auto-updated to: "America/New_York"
- Notification sent at: 09:00 America time (no manual update needed!)

User manually overrides timezone while in America:
- Sends: { timezone: "Asia/Karachi" } in PATCH /user-settings
- Timezone locked to: "Asia/Karachi"
- Notification sent at: 09:00 Pakistan time (even when in America)
- Note: Manual override persists until user updates settings again
```

## Scheduled Jobs

### 1. **Celebration Notifications** (Hourly)

The scheduler runs every hour and processes notifications based on each user's `remindMeTime` preference.

**Schedule:** `0 * * * *` (every hour at the start of the hour)

**How it works:**

1. **Check Reminder Time**
   - For each user, check if current time matches their `remindMeTime` (±30 minutes)
   - Skip users whose reminder time doesn't match

2. **Determine Scope**
   - Get list of users to check based on `getReminderFor` setting
   - `all_company`: All users in user's organizations
   - `my_department`: All users in user's departments  
   - `who_i_selected`: Only selected users from `whoISelected` array

3. **Check Dates**
   - For each user in scope, check if their birthday or anniversary is:
     - Today
     - Tomorrow (1 day away)
     - 2 days away
     - 7 days away

4. **Respect Individual Preferences**
   - For `who_i_selected` mode, check `birthdayReminder` and `anniversaryReminder` flags
   - Only send notifications if the flag is `true`

5. **Send Notification**
   - Send push notification via OneSignal to user's device
   - Requires user to have `oneSignalPlayerId` set

**Example Flow:**

```javascript
// User A Settings:
{
  getReminderFor: "my_department",
  remindMeTime: "09:00"
}

// At 09:00, scheduler:
// 1. Gets all users in User A's departments
// 2. Checks each user's birthday/anniversary
// 3. Sends notifications for upcoming events
// 4. Logs activity
```

## Notification Types

### Birthday Notifications

**Same Day:**
```
🎂 Birthday Reminder
Today is John Doe's birthday! 🎉
```

**Upcoming:**
```
🎂 Upcoming Birthday
John Doe's birthday is in 2 days! 🎉
```

### Anniversary Notifications

**Same Day:**
```
🎊 Work Anniversary
Jane Smith is celebrating 5 years with the company! 🎉
```

**Upcoming:**
```
🎊 Upcoming Work Anniversary
Jane Smith's 5 year anniversary is in 7 days! 🎉
```

### Other Notifications

**Reaction Notification:**
```
👍 New Reaction
John Doe reacted to your answer
```

**New Question:**
```
❓ New Question
Jane Smith posted: How do I configure the API?
```

## Schedule Times

| Job | Schedule | Cron Expression | Description |
|-----|----------|----------------|-------------|
| Birthday/Anniversary Notifications | Every hour | `0 * * * *` | Checks for upcoming events and sends notifications based on user preferences |

## Notification Delivery

All notifications are sent via **OneSignal**:

### Requirements

1. **OneSignal Configuration**
   ```env
   ONESIGNAL_APP_ID=your_app_id
   ONESIGNAL_REST_API_KEY=your_rest_api_key
   ONESIGNAL_USER_AUTH_KEY=your_user_auth_key
   ```

2. **User Setup**
   - User must have `oneSignalPlayerId` set
   - This is obtained when user subscribes to push notifications in the app

### Channels

- **Mobile Push** (iOS/Android)
- **Web Push** (Browser notifications)

## Configuration

### User Settings Management

Users can configure their preferences via the API:

**Get Settings:**
```http
GET /v1/user-settings?organizationId={orgId}
```

**Update Settings:**
```http
PATCH /v1/user-settings
Content-Type: application/json

{
  "getReminderFor": "my_department",
  "remindMeTime": "09:00",
  "reactionNotification": true,
  "newQuestionNotifications": true
}
```

**Add Selected User:**
```http
POST /v1/user-settings/selected-users
Content-Type: application/json

{
  "userId": "user123",
  "birthdayReminder": true,
  "anniversaryReminder": false
}
```

**Remove Selected User:**
```http
DELETE /v1/user-settings/selected-users/{userId}
```

### Default Settings

When a user is created, default settings are:
```javascript
{
  getReminderFor: "all_company",
  whoISelected: [],
  reactionNotification: true,
  newQuestionNotifications: true,
  remindMeTime: "09:00",
  notifications: {
    emailNotifications: true,
    pushNotifications: true,
    weeklyDigest: true
  },
  privacy: {
    showBirthdayToTeam: true,
    showWorkAnniversaryToTeam: true,
    showProfileInDirectory: true
  }
}
```

## Incoming Events API

Users can view upcoming events via the API:

```http
GET /v1/incoming-events?daysAhead=30
```

**Response:**
```json
{
  "events": [
    {
      "type": "birthday",
      "date": "2025-11-05T00:00:00.000Z",
      "daysUntil": 7,
      "user": {
        "id": "user123",
        "firstName": "John",
        "lastName": "Doe",
        "imageUrl": "https://...",
        "jobTitle": "Engineer",
        "department": "Engineering"
      }
    },
    {
      "type": "anniversary",
      "date": "2025-11-15T00:00:00.000Z",
      "daysUntil": 17,
      "yearsOfService": 5,
      "user": {
        "id": "user456",
        "firstName": "Jane",
        "lastName": "Smith",
        ...
      }
    }
  ],
  "total": 2,
  "daysAhead": 30
}
```

## Implementation Details

### Scheduler Service

**Location:** `src/services/scheduler.service.js`

**Main Functions:**
- `startCelebrationScheduler()` - Starts the cron job
- `processCelebrationNotifications()` - Main processing function
- `sendTestNotification(userId)` - Send test notification for debugging

**Startup:**
The scheduler starts automatically when the application launches:
```javascript
// src/index.js
import schedulerService from './services/scheduler.service.js';

mongoose.connect(config.mongoose.url).then(() => {
  schedulerService.startCelebrationScheduler();
  // ...
});
```

### Notification Service

**Location:** `src/services/notification.service.js`

**Functions:**
- `sendBirthdayNotification(recipientUser, celebrationUser)`
- `sendAnniversaryNotification(recipientUser, celebrationUser, years)`
- `sendUpcomingBirthdayReminder(recipientUser, celebrationUser, daysUntil)`
- `sendUpcomingAnniversaryReminder(recipientUser, celebrationUser, daysUntil, years)`
- `sendReactionNotification(recipientUser, reactorUser, contentType)`
- `sendNewQuestionNotification(recipientUser, author, questionTitle)`

### Incoming Events Service

**Location:** `src/services/incomingEvents.service.js`

**Functions:**
- `getIncomingEvents(userId, organizationId, daysAhead)`

Returns upcoming birthdays and anniversaries filtered by user's preferences.

## Testing

### Manual Testing

**Test Notification:**
```javascript
import schedulerService from './src/services/scheduler.service.js';

// Send test notification
await schedulerService.sendTestNotification('userId');
```

**Trigger Scheduler Manually:**
```javascript
import schedulerService from './src/services/scheduler.service.js';

// Run scheduler check immediately
await schedulerService.processCelebrationNotifications();
```

### Check Logs

The scheduler logs all activities:
```
Running celebration notifications check at 09:00
Birthday notification sent to user123 for user456, notification ID: abc123
Celebration notifications check completed
```

## Monitoring

### Successful Notifications

Check logs for:
- `Birthday notification sent to {userId}`
- `Anniversary notification sent to {userId}`
- `notification ID: {oneSignalId}`

### Failed Notifications

Common issues:
- `User {userId} has no OneSignal Player ID` - User hasn't subscribed to notifications
- `Failed to send birthday notification: {error}` - OneSignal API error

### Debug Mode

To see detailed logs, set log level to `debug` in config:
```javascript
// config.js
logger: {
  level: 'debug'
}
```

## Best Practices

### 1. User Onboarding
- Prompt users to set their `oneSignalPlayerId` on first login
- Encourage users to configure their notification preferences
- Explain the different `getReminderFor` options

### 2. Privacy
- Respect `privacy.showBirthdayToTeam` setting
- Don't send notifications for users who have hidden their birthday
- Allow users to opt out of all notifications

### 3. Performance
- Scheduler runs hourly, not every minute
- 30-minute tolerance window for reminder times
- Efficient querying using indexes on dates

### 4. Time Zones
- Current implementation uses server time
- Consider implementing time zone support for global teams
- Store user's preferred time zone in settings

## Troubleshooting

### Notifications Not Sending

1. **Check OneSignal Configuration**
   ```bash
   echo $ONESIGNAL_APP_ID
   echo $ONESIGNAL_REST_API_KEY
   ```

2. **Check User Settings**
   ```javascript
   const settings = await UserSettings.findOne({ userId });
   console.log(settings.remindMeTime);
   console.log(settings.getReminderFor);
   ```

3. **Check OneSignal Player ID**
   ```javascript
   const user = await User.findById(userId);
   console.log(user.oneSignalPlayerId);
   ```

4. **Check Scheduler Logs**
   ```bash
   tail -f logs/combined.log | grep "celebration"
   ```

### Duplicate Notifications

- Ensure only one instance of the application is running
- Check that cron job isn't duplicated
- Consider implementing distributed locks for multi-server setups

### Wrong Users Receiving Notifications

1. **Check `getReminderFor` Setting**
   - Verify user's preference is set correctly
   
2. **Check Organization/Department Memberships**
   - Ensure user belongs to correct organization/department
   
3. **Check `whoISelected` Array**
   - Verify selected users list is correct

## Future Enhancements

- [ ] Email notifications in addition to push
- [ ] SMS notifications via OneSignal
- [ ] Weekly digest of upcoming events
- [ ] Time zone support
- [ ] Notification history/logs per user
- [ ] A/B testing for notification timing
- [ ] Custom notification messages
- [ ] Notification analytics dashboard

## Summary

The new notification system provides:

✅ **User Control** - Users configure their own preferences  
✅ **Flexibility** - Multiple scopes and granular control  
✅ **Scalability** - Efficient hourly processing  
✅ **Privacy** - Respect user privacy settings  
✅ **Reliability** - Built on OneSignal's infrastructure  
✅ **Simplicity** - No manual alert creation needed  

---

For more details, see:
- [User Model Documentation](../models/)
- [OneSignal Setup Guide](../setup/ONESIGNAL_SETUP.md)
- [API Documentation](../api/)
