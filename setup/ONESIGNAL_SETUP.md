# OneSignal Integration Guide

## Overview
This application integrates OneSignal for push notifications and email delivery for celebrations and birthday alerts.

## Setup Instructions

### 1. OneSignal Account Setup
1. Go to [OneSignal](https://onesignal.com/) and create an account
2. Create a new app in your OneSignal dashboard
3. Get your credentials:
   - **App ID**: Found in Settings > Keys & IDs
   - **REST API Key**: Found in Settings > Keys & IDs
   - **User Auth Key**: Found in your Account Settings

### 2. Environment Variables
Add the following to your `.env` file:

```env
# OneSignal Configuration
ONESIGNAL_APP_ID=your_app_id_here
ONESIGNAL_REST_API_KEY=your_rest_api_key_here
ONESIGNAL_USER_AUTH_KEY=your_user_auth_key_here
```

### 3. Install Dependencies
Install the official OneSignal Node SDK:

```bash
npm install @onesignal/node-onesignal
```

This is already included in `package.json`, just run:
```bash
npm install
```

## Features

### 1. Celebrations Management
Users can create, update, and delete celebrations with automatic reminders.

#### Celebration Types:
- `birthday` - Birthday celebrations
- `work_anniversary` - Work anniversaries
- `custom` - Custom celebrations
- `holiday` - Holiday celebrations
- `achievement` - Achievement milestones

#### API Endpoints:

**Create Celebration**
```bash
POST /v1/celebrations
Authorization: Bearer <token>

{
  "userId": "user_id",
  "title": "Wedding Anniversary",
  "type": "custom",
  "date": "2024-06-15T00:00:00.000Z",
  "description": "Celebrate our special day",
  "reminderDaysBefore": 7,
  "isRecurring": true
}
```

**Get User Celebrations**
```bash
GET /v1/celebrations/user/:userId
Authorization: Bearer <token>
```

**Update Celebration**
```bash
PATCH /v1/celebrations/:celebrationId
Authorization: Bearer <token>

{
  "title": "Updated Title",
  "reminderDaysBefore": 14
}
```

**Delete Celebration**
```bash
DELETE /v1/celebrations/:celebrationId
Authorization: Bearer <token>
```

### 2. DOB Alerts Management
Create and manage date of birth related alerts.

#### Alert Types:
- `birthday_reminder` - Birthday reminders
- `age_milestone` - Age milestone alerts
- `custom` - Custom DOB alerts

#### API Endpoints:

**Create DOB Alert**
```bash
POST /v1/dob-alerts
Authorization: Bearer <token>

{
  "userId": "user_id",
  "alertType": "birthday_reminder",
  "alertDate": "2024-01-15T00:00:00.000Z",
  "message": "Don't forget to celebrate!",
  "daysBefore": 7
}
```

**Get User DOB Alerts**
```bash
GET /v1/dob-alerts/user/:userId
Authorization: Bearer <token>
```

**Update DOB Alert**
```bash
PATCH /v1/dob-alerts/:dobAlertId
Authorization: Bearer <token>

{
  "daysBefore": 14,
  "message": "Updated reminder message"
}
```

**Delete DOB Alert**
```bash
DELETE /v1/dob-alerts/:dobAlertId
Authorization: Bearer <token>
```

## OneSignal Service Usage

The service is built on the official `@onesignal/node-onesignal` SDK.

### Send Push Notification
```javascript
import onesignalService from './services/onesignal.service.js';

await onesignalService.sendPushNotification({
  playerIds: ['player_id_1', 'player_id_2'],
  heading: 'Notification Title',
  content: 'Notification message',
  data: { customKey: 'customValue' }
});
```

### Send Email
```javascript
await onesignalService.sendEmail({
  emails: ['user@example.com'],
  subject: 'Email Subject',
  content: '<h1>HTML Content</h1><p>This is HTML email body</p>',
  data: { customKey: 'customValue' }
});
```

### Send SMS
```javascript
await onesignalService.sendSMS({
  phoneNumbers: ['+1234567890'],
  content: 'Your SMS message here',
  smsFrom: '+1234567890', // Your OneSignal SMS sender number
  data: { customKey: 'customValue' }
});
```

### Send to Segments
```javascript
await onesignalService.sendToSegments({
  segments: ['All', 'Active Users'],
  heading: 'Announcement',
  content: 'Important update for all users',
  data: { type: 'announcement' }
});
```

### Send by External User IDs
```javascript
// Use your own user IDs (from your database)
await onesignalService.sendByExternalUserIds({
  externalUserIds: ['user_123', 'user_456'],
  heading: 'Personal Message',
  content: 'This is for specific users',
  data: { customKey: 'value' }
});
```

### Send Celebration Reminder
```javascript
await onesignalService.sendCelebrationReminder({
  userEmail: 'user@example.com',
  playerId: 'onesignal_player_id', // Optional
  externalUserId: 'your_user_id', // Optional (use instead of playerId)
  celebrationTitle: 'Wedding Anniversary',
  celebrationDate: new Date('2024-06-15'),
  userName: 'John Doe'
});
```

### Send Birthday Reminder
```javascript
await onesignalService.sendBirthdayReminder({
  userEmail: 'user@example.com',
  playerId: 'onesignal_player_id', // Optional
  externalUserId: 'your_user_id', // Optional (use instead of playerId)
  birthdayDate: new Date('1990-01-15'),
  userName: 'John Doe',
  message: 'Custom birthday message (optional)'
});
```

### Register/Update Player
```javascript
await onesignalService.createOrUpdatePlayer({
  userId: 'your_user_id', // Your database user ID (external_id)
  identifier: 'user@example.com', // Email or device token
  deviceType: 11, // 0=iOS, 1=Android, 11=Email, 14=SMS
  tags: {
    department: 'Engineering',
    role: 'developer'
  }
});
```

### Get Notification Status
```javascript
const notification = await onesignalService.getNotification('notification_id');
console.log('Notification status:', notification);
```

### Cancel Notification
```javascript
await onesignalService.cancelNotification('notification_id');
```

## User Model Updates

The User model now includes:
- `oneSignalPlayerId`: Stores the OneSignal player ID for push notifications
- `celebrations`: Array of celebration IDs (references to Celebration model)
- `dob_alerts`: Array of DOB alert IDs (references to DobAlert model)

### Update User with OneSignal Player ID
```bash
PATCH /v1/users/:userId
Authorization: Bearer <token>

{
  "oneSignalPlayerId": "onesignal_player_id_here"
}
```

## Automated Notifications

### Setting Up Cron Jobs
You can set up cron jobs to automatically send notifications for upcoming celebrations and birthdays:

```javascript
// Example: Check daily for upcoming celebrations
import cron from 'node-cron';
import Celebration from './models/celebration.model.js';
import User from './models/user.model.js';
import onesignalService from './services/onesignal.service.js';

// Run every day at 9 AM
cron.schedule('0 9 * * *', async () => {
  const today = new Date();
  const upcomingDate = new Date(today);
  upcomingDate.setDate(today.getDate() + 7); // 7 days ahead

  const celebrations = await Celebration.find({
    date: { $gte: today, $lte: upcomingDate },
    isActive: true,
    notificationSent: false
  }).populate('userId');

  for (const celebration of celebrations) {
    const user = celebration.userId;
    
    await onesignalService.sendCelebrationReminder({
      userEmail: user.email,
      playerId: user.oneSignalPlayerId,
      celebrationTitle: celebration.title,
      celebrationDate: celebration.date,
      userName: `${user.firstName} ${user.lastName}`
    });

    // Mark notification as sent
    celebration.notificationSent = true;
    celebration.lastNotificationDate = new Date();
    await celebration.save();
  }
});
```

## Frontend Integration

### Web Push Setup
Add the OneSignal SDK to your frontend:

```html
<script src="https://cdn.onesignal.com/sdks/OneSignalSDK.js" async></script>
<script>
  window.OneSignal = window.OneSignal || [];
  OneSignal.push(function() {
    OneSignal.init({
      appId: "YOUR_ONESIGNAL_APP_ID",
    });
  });
</script>
```

### Get Player ID and Update User
```javascript
OneSignal.push(function() {
  OneSignal.getUserId(function(userId) {
    // Update user with OneSignal player ID
    fetch('/v1/users/' + currentUserId, {
      method: 'PATCH',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer ' + token
      },
      body: JSON.stringify({
        oneSignalPlayerId: userId
      })
    });
  });
});
```

## Testing

### Test Push Notification
```bash
curl -X POST https://onesignal.com/api/v1/notifications \
  -H "Content-Type: application/json" \
  -H "Authorization: Basic YOUR_REST_API_KEY" \
  -d '{
    "app_id": "YOUR_APP_ID",
    "include_player_ids": ["player_id"],
    "headings": {"en": "Test Notification"},
    "contents": {"en": "This is a test"}
  }'
```

## Best Practices

1. **Store Player IDs**: Always update the user's `oneSignalPlayerId` when they log in from a new device
2. **Handle Failures**: Use try-catch blocks when sending notifications as they may fail
3. **Rate Limiting**: Be mindful of OneSignal's API rate limits
4. **Batch Notifications**: Use segments for sending to multiple users
5. **Test Before Production**: Always test notifications in development environment first

## Troubleshooting

### Common Issues:

**Notifications not sending:**
- Check OneSignal credentials in `.env`
- Verify player IDs are valid
- Check OneSignal dashboard for delivery reports

**Email not working:**
- Ensure OneSignal email is configured in dashboard
- Verify email addresses are valid
- Check spam folders

**Player ID not found:**
- User needs to grant notification permissions
- OneSignal SDK must be properly initialized on frontend

## Resources

- [OneSignal Documentation](https://documentation.onesignal.com/)
- [OneSignal REST API Reference](https://documentation.onesignal.com/reference)
- [OneSignal Web Push Guide](https://documentation.onesignal.com/docs/web-push-quickstart)

