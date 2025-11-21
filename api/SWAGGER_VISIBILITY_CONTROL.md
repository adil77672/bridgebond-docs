# Swagger Module Visibility Control

This guide explains how to control which modules are visible in the Swagger API documentation.

## Overview

The Swagger documentation can be configured to hide or show specific modules. This is useful when you want to:
- Hide internal/admin modules from client-facing documentation
- Show only specific modules to different audiences
- Control API exposure based on environment

## Configuration Methods

### Method 1: Edit Configuration File (Recommended)

Edit the file: `src/config/swaggerVisibility.js`

Change the `defaultVisibility` object to set which modules are visible:

```javascript
const defaultVisibility = {
  'Auth': true,              // Visible
  'OTP': true,               // Visible
  'Users': true,             // Visible
  'UserSettings': false,     // Hidden
  'Organizations': true,     // Visible
  'Departments': true,       // Visible
  'Questions': false,        // Hidden
  'Profile': true,           // Visible
  'Devices': false,          // Hidden
  'IncomingEvents': false,   // Hidden
  'Audit Logs': true,       // Visible
  'HR Integrations': false,  // Hidden
  'Notes': false,           // Hidden
  'Notifications': false,    // Hidden
  'Payments': false,        // Hidden
  'Plans': false,           // Hidden
  'Webhooks': false,        // Hidden
  'Dashboard': true,        // Visible
  'Features': false,        // Hidden
};
```

**To show a module:** Set to `true`  
**To hide a module:** Set to `false`

### Method 2: Environment Variable

Set the `SWAGGER_HIDDEN_MODULES` environment variable with a comma-separated list of modules to hide:

```bash
# Hide specific modules
SWAGGER_HIDDEN_MODULES=UserSettings,Questions,Devices,IncomingEvents,HR Integrations,Notes,Notifications,Payments,Plans,Webhooks,Features

# Or in .env file
SWAGGER_HIDDEN_MODULES=UserSettings,Questions,Devices
```

**Note:** When using environment variable, all modules NOT listed will be visible by default.

## Available Modules

| Module Name | Tag Name | Route Prefix | Default Status |
|------------|----------|--------------|----------------|
| Authentication | Auth | `/auth` | ✅ Visible |
| OTP | OTP | `/otp` | ✅ Visible |
| Users | Users | `/users` | ✅ Visible |
| User Settings | UserSettings | `/user-settings` | ❌ Hidden |
| Organizations | Organizations | `/organizations` | ✅ Visible |
| Departments | Departments | `/departments` | ✅ Visible |
| Questions | Questions | `/questions` | ❌ Hidden |
| Profile | Profile | `/profile` | ✅ Visible |
| Devices | Devices | `/devices` | ❌ Hidden |
| Incoming Events | IncomingEvents | `/incoming-events` | ❌ Hidden |
| Audit Logs | Audit Logs | `/audit-logs` | ✅ Visible |
| HR Integrations | HR Integrations | `/hr-integrations` | ❌ Hidden |
| Notes | Notes | `/notes` | ❌ Hidden |
| Notifications | Notifications | `/notifications` | ❌ Hidden |
| Payments | Payments | `/payments` | ❌ Hidden |
| Plans | Plans | `/plans` | ❌ Hidden |
| Webhooks | Webhooks | `/webhooks` | ❌ Hidden |
| Dashboard | Dashboard | `/dashboard` | ✅ Visible |
| Features | Features | `/features` | ❌ Hidden |

## How It Works

1. **Tag Filtering**: The Swagger tags are filtered based on visibility configuration
2. **Path Filtering**: All API paths under hidden modules are removed from the documentation
3. **Dynamic**: Changes take effect when the server restarts

## Examples

### Example 1: Show All Modules

Edit `src/config/swaggerVisibility.js`:

```javascript
const defaultVisibility = {
  'Auth': true,
  'OTP': true,
  'Users': true,
  'UserSettings': true,      // Changed to true
  'Organizations': true,
  'Departments': true,
  'Questions': true,          // Changed to true
  'Profile': true,
  'Devices': true,            // Changed to true
  'IncomingEvents': true,     // Changed to true
  'Audit Logs': true,
  'HR Integrations': true,    // Changed to true
  'Notes': true,              // Changed to true
  'Notifications': true,       // Changed to true
  'Payments': true,           // Changed to true
  'Plans': true,              // Changed to true
  'Webhooks': true,           // Changed to true
  'Dashboard': true,
  'Features': true,           // Changed to true
};
```

### Example 2: Hide Additional Modules

Edit `src/config/swaggerVisibility.js`:

```javascript
const defaultVisibility = {
  'Auth': true,
  'OTP': true,
  'Users': false,              // Hide Users module
  'UserSettings': false,
  'Organizations': true,
  'Departments': false,       // Hide Departments module
  'Questions': false,
  'Profile': true,
  'Devices': false,
  'IncomingEvents': false,
  'Audit Logs': false,        // Hide Audit Logs module
  'HR Integrations': false,
  'Notes': false,
  'Notifications': false,
  'Payments': false,
  'Plans': false,
  'Webhooks': false,
  'Dashboard': true,
  'Features': false,
};
```

### Example 3: Using Environment Variable

In your `.env` file:

```bash
# Hide only Questions and Payments
SWAGGER_HIDDEN_MODULES=Questions,Payments
```

Or when starting the server:

```bash
SWAGGER_HIDDEN_MODULES=Questions,Payments npm start
```

## Verification

After making changes:

1. Restart your server
2. Visit `/v1/docs` in your browser
3. Check the Swagger UI - hidden modules should not appear in the tags list
4. Check the server logs - you should see a message like:
   ```
   Swagger specs generated: 50 total paths, 35 visible paths, 15 hidden paths
   Hidden path prefixes: /user-settings, /questions, /devices, ...
   ```

## Current Default Configuration

By default, the following modules are **HIDDEN**:
- UserSettings
- Questions
- Devices
- IncomingEvents
- HR Integrations
- Notes
- Notifications
- Payments
- Plans
- Webhooks
- Features

The following modules are **VISIBLE**:
- Auth
- OTP
- Users
- Organizations
- Departments
- Profile
- Audit Logs
- Dashboard

## Troubleshooting

### Modules still showing after changes

1. Make sure you restarted the server after making changes
2. Check that the module name matches exactly (case-sensitive)
3. Verify the configuration file syntax is correct

### Environment variable not working

1. Make sure the environment variable is set before starting the server
2. Check that module names match exactly (including spaces, e.g., "HR Integrations")
3. Verify the `.env` file is being loaded correctly

### Need to see all modules temporarily

Set the environment variable to an empty value:

```bash
SWAGGER_HIDDEN_MODULES=
```

This will show all modules.

## Technical Details

- Configuration file: `src/config/swaggerVisibility.js`
- Swagger definition: `src/docs/swaggerDef.js`
- Route handler: `src/routes/v1/docs.route.js`
- Filtering happens at both tag and path levels
- Changes require server restart to take effect

