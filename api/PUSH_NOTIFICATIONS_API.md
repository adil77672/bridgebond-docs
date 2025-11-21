# Push Notification Service API

## Overview

The `pushNotificationService` provides methods to send custom push notifications using the OneSignal NodeJS SDK.

## Import

```javascript
import { pushNotificationService } from '../services/index.js';
```

## Methods

### 1. Send to Specific Players

Send notification to specific device(s) by Player ID.

```javascript
const notificationId = await pushNotificationService.sendToPlayers({
  playerIds: ['player-id-1', 'player-id-2'],
  heading: 'Welcome!',
  content: 'Thanks for joining Bridge Bond',
  data: {
    type: 'welcome',
    userId: user.id,
  },
  url: 'https://yourapp.com/welcome',
});
```

**Parameters:**
- `playerIds` (Array<string>) - OneSignal player/device IDs
- `heading` (string) - Notification title
- `content` (string) - Notification message
- `data` (Object) - Optional custom data
- `url` (string) - Optional URL to open

**Returns:** `Promise<string>` - Notification ID

---

### 2. Send to All Subscribed Users

Broadcast to everyone subscribed.

```javascript
const notificationId = await pushNotificationService.sendToAll({
  heading: 'New Feature Released!',
  content: 'Check out our latest update',
  url: 'https://yourapp.com/updates',
});
```

**Parameters:**
- `heading` (string) - Notification title
- `content` (string) - Notification message
- `data` (Object) - Optional custom data
- `url` (string) - Optional URL to open

**Returns:** `Promise<string>` - Notification ID

---

### 3. Send to Segments

Send to specific user segments.

```javascript
const notificationId = await pushNotificationService.sendToSegments({
  segments: ['Active Users', 'Premium Members'],
  heading: 'Exclusive Offer',
  content: 'Get 20% off today only!',
  data: {
    type: 'promotion',
    discount: 20,
  },
});
```

**Parameters:**
- `segments` (Array<string>) - Segment names from OneSignal dashboard
- `heading` (string) - Notification title
- `content` (string) - Notification message
- `data` (Object) - Optional custom data
- `url` (string) - Optional URL to open

**Returns:** `Promise<string>` - Notification ID

---

### 4. Send Scheduled Notification

Schedule a notification for later.

```javascript
const sendTime = new Date();
sendTime.setHours(sendTime.getHours() + 24); // 24 hours from now

const notificationId = await pushNotificationService.sendScheduled({
  playerIds: ['player-id-1'],
  heading: 'Reminder',
  content: 'Your appointment is tomorrow',
  sendAfter: sendTime,
  data: {
    type: 'reminder',
    appointmentId: '123',
  },
});
```

**Parameters:**
- `playerIds` (Array<string>) - Optional player IDs (or use segments)
- `segments` (Array<string>) - Optional segments (or use playerIds)
- `heading` (string) - Notification title
- `content` (string) - Notification message
- `sendAfter` (Date|string) - When to send (Date object or ISO string)
- `data` (Object) - Optional custom data
- `url` (string) - Optional URL to open

**Returns:** `Promise<string>` - Notification ID

---

### 5. View Notification

Get notification details and statistics.

```javascript
const notificationData = await pushNotificationService.viewNotification(notificationId);

console.log('Sent:', notificationData.successful);
console.log('Failed:', notificationData.failed);
console.log('Converted:', notificationData.converted);
console.log('Remaining:', notificationData.remaining);
```

**Parameters:**
- `notificationId` (string) - OneSignal notification ID

**Returns:** `Promise<Object>` - Notification data with statistics

**Response Example:**
```json
{
  "id": "notification-id",
  "successful": 150,
  "failed": 5,
  "converted": 23,
  "remaining": 0,
  "queued_at": 1234567890,
  "send_after": 1234567890,
  "completed_at": 1234567900,
  "headings": { "en": "Welcome!" },
  "contents": { "en": "Thanks for joining" },
  "url": "https://yourapp.com",
  "data": { "type": "welcome" }
}
```

---

### 6. Cancel Scheduled Notification

Cancel a notification that hasn't been sent yet.

```javascript
const success = await pushNotificationService.cancelNotification(notificationId);

if (success) {
  console.log('Notification cancelled');
}
```

**Parameters:**
- `notificationId` (string) - OneSignal notification ID

**Returns:** `Promise<boolean>` - Success status

---

## Usage Examples

### Example 1: Birthday Notification

```javascript
import { pushNotificationService } from '../services/index.js';

const sendBirthdayNotification = async (user) => {
  if (!user.oneSignalPlayerIds || user.oneSignalPlayerIds.length === 0) {
    return;
  }

  await pushNotificationService.sendToPlayers({
    playerIds: user.oneSignalPlayerIds,
    heading: '🎂 Happy Birthday!',
    content: `${user.firstName}, we wish you an amazing birthday!`,
    data: {
      type: 'birthday',
      userId: user.id,
    },
    url: 'https://yourapp.com/celebrations',
  });
};
```

### Example 2: System Announcement

```javascript
const sendAnnouncement = async (title, message) => {
  const notificationId = await pushNotificationService.sendToAll({
    heading: title,
    content: message,
    data: {
      type: 'announcement',
      timestamp: new Date().toISOString(),
    },
  });

  console.log(`Announcement sent: ${notificationId}`);
};

await sendAnnouncement(
  'System Maintenance',
  'Our app will be down for maintenance tonight at 2 AM'
);
```

### Example 3: Scheduled Reminder

```javascript
const scheduleReminder = async (user, event, reminderTime) => {
  const notificationId = await pushNotificationService.sendScheduled({
    playerIds: user.oneSignalPlayerIds,
    heading: `Reminder: ${event.title}`,
    content: `Your event starts in 1 hour`,
    sendAfter: reminderTime,
    data: {
      type: 'event_reminder',
      eventId: event.id,
    },
    url: `https://yourapp.com/events/${event.id}`,
  });

  // Save notification ID to cancel later if needed
  return notificationId;
};
```

### Example 4: Track Notification Performance

```javascript
const trackNotificationPerformance = async (notificationId) => {
  const stats = await pushNotificationService.viewNotification(notificationId);

  const deliveryRate = (stats.successful / (stats.successful + stats.failed)) * 100;
  const conversionRate = (stats.converted / stats.successful) * 100;

  console.log(`Delivery Rate: ${deliveryRate.toFixed(2)}%`);
  console.log(`Conversion Rate: ${conversionRate.toFixed(2)}%`);

  return {
    deliveryRate,
    conversionRate,
    totalSent: stats.successful,
    totalConverted: stats.converted,
  };
};
```

### Example 5: Cancel Scheduled Notifications

```javascript
const cancelEventReminders = async (eventId) => {
  // Assume you stored notification IDs in database
  const scheduledNotifications = await Notification.find({ eventId });

  for (const notification of scheduledNotifications) {
    await pushNotificationService.cancelNotification(notification.onesignalId);
  }

  console.log(`Cancelled ${scheduledNotifications.length} reminders`);
};
```

---

## Integration with Existing Code

### Replace Existing Notification Calls

**Before:**
```javascript
// Old way - using notification.service.js
const notification = new OneSignal.Notification();
notification.app_id = onesignalConfig.appId;
notification.include_player_ids = user.oneSignalPlayerIds;
notification.headings = { en: 'Birthday' };
notification.contents = { en: 'Happy Birthday!' };
await onesignalConfig.client.createNotification(notification);
```

**After:**
```javascript
// New way - using pushNotification.service.js
await pushNotificationService.sendToPlayers({
  playerIds: user.oneSignalPlayerIds,
  heading: 'Birthday',
  content: 'Happy Birthday!',
});
```

---

## Error Handling

```javascript
try {
  const notificationId = await pushNotificationService.sendToPlayers({
    playerIds: user.oneSignalPlayerIds,
    heading: 'Test',
    content: 'Test message',
  });
  
  console.log('Sent:', notificationId);
} catch (error) {
  console.error('Failed to send notification:', error.message);
  // Handle error appropriately
}
```

---

## Configuration

Ensure OneSignal is configured in your `.env`:

```env
NOTIFICATION_PROVIDER=onesignal
ONESIGNAL_APP_ID=97518421-dcf2-446b-a170-f5c07c462bab
ONESIGNAL_REST_API_KEY=your-rest-api-key
ONESIGNAL_USER_AUTH_KEY=your-user-auth-key
```

---

## Best Practices

### 1. **Check Player IDs**
Always verify user has player IDs before sending:
```javascript
if (user.oneSignalPlayerIds && user.oneSignalPlayerIds.length > 0) {
  await pushNotificationService.sendToPlayers({...});
}
```

### 2. **Use Segments for Large Groups**
For broadcasts, use segments instead of individual player IDs:
```javascript
// Good
await pushNotificationService.sendToSegments({
  segments: ['Active Users'],
  heading: 'Update',
  content: 'New version available',
});

// Bad - don't do this
await pushNotificationService.sendToPlayers({
  playerIds: [/* thousands of IDs */],
  ...
});
```

### 3. **Add Custom Data**
Include data for better tracking and handling:
```javascript
await pushNotificationService.sendToPlayers({
  playerIds: user.oneSignalPlayerIds,
  heading: 'New Message',
  content: 'You have a new message',
  data: {
    type: 'message',
    messageId: message.id,
    senderId: sender.id,
  },
});
```

### 4. **Track Performance**
Monitor notification effectiveness:
```javascript
const notificationId = await pushNotificationService.sendToAll({...});

// Check stats after some time
setTimeout(async () => {
  const stats = await pushNotificationService.viewNotification(notificationId);
  console.log('Engagement:', stats.converted / stats.successful);
}, 60000); // 1 minute
```

---

## Resources

- [OneSignal NodeJS SDK](https://github.com/OneSignal/onesignal-node-api)
- [OneSignal API Reference](https://documentation.onesignal.com/reference)
- [Push Notification Best Practices](https://documentation.onesignal.com/docs/sending-notifications)

