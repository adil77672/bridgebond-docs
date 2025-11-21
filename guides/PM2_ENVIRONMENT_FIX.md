# PM2 Environment Configuration Fix

## Issue

The backend was failing with:
```
Error: Config validation error: "NODE_ENV" must be one of [production, development, test]
```

This happened because `ecosystem.config.json` was using `NODE_ENV: "staging"`, but the config validation only allows `production`, `development`, or `test`.

## Solution

The `ecosystem.config.json` has been updated to use:
- **Staging**: `NODE_ENV: "development"` with `IS_LIVE: "false"`
- **Production**: `NODE_ENV: "production"` with `IS_LIVE: "true"`

The `IS_LIVE` flag is used to differentiate between staging and production databases.

## Updated Configuration

### Backend Staging
```json
{
  "name": "backend-staging",
  "env": {
    "NODE_ENV": "development",
    "IS_LIVE": "false",
    "PORT": 2323,
    "HOST": "0.0.0.0"
  }
}
```

### Backend Production
```json
{
  "name": "backend-prod",
  "env": {
    "NODE_ENV": "production",
    "IS_LIVE": "true",
    "PORT": 2324,
    "HOST": "0.0.0.0"
  }
}
```

## How It Works

The application uses `IS_LIVE` to determine which database to use:

- **Staging** (`IS_LIVE: false`): Uses `MONGODB_URL_STAGING`
- **Production** (`IS_LIVE: true`): Uses `MONGODB_URL_LIVE`

This is defined in `src/config/config.js`:

```javascript
url: envVars.MONGODB_URI 
  ? envVars.MONGODB_URI
  : (envVars.IS_LIVE
      ? envVars.MONGODB_URL_LIVE
      : envVars.MONGODB_URL_STAGING + (envVars.NODE_ENV === 'test' ? '-test' : '')),
```

## Steps to Fix on Server

1. **Pull the updated `ecosystem.config.json`**:
   ```bash
   cd ~/bridge-bond/staging/bridge-bond-api
   git pull origin staging
   ```

2. **Restart the PM2 process**:
   ```bash
   pm2 restart backend-staging
   ```

3. **Verify it's working**:
   ```bash
   pm2 logs backend-staging --lines 20
   ```

   You should see the server starting successfully without the validation error.

## Environment Variables Required

Make sure your `.env` file has:

```env
# For Staging
NODE_ENV=development
IS_LIVE=false
MONGODB_URL_STAGING=mongodb://username:password@host:27017/bridge-bond-staging
MONGODB_URL_LIVE=mongodb://username:password@host:27017/bridge-bond-live

# For Production
NODE_ENV=production
IS_LIVE=true
MONGODB_URL_STAGING=mongodb://username:password@host:27017/bridge-bond-staging
MONGODB_URL_LIVE=mongodb://username:password@host:27017/bridge-bond-live
```

**Note:** Even in production, you should have both `MONGODB_URL_STAGING` and `MONGODB_URL_LIVE` defined. The `IS_LIVE` flag determines which one is used.

## Quick Fix Command

If you need to quickly fix this on the server without pulling:

```bash
cd ~/bridge-bond/staging/bridge-bond-api
nano ecosystem.config.json
```

Change:
```json
"NODE_ENV": "staging",
```

To:
```json
"NODE_ENV": "development",
"IS_LIVE": "false",
```

Then restart:
```bash
pm2 restart backend-staging
```

## Verification

After the fix, check:

```bash
# Check PM2 status
pm2 status

# Check logs (should not show validation error)
pm2 logs backend-staging --lines 30

# Test the API
curl http://localhost:2323/health
```

## Related Files

- `ecosystem.config.json` - PM2 configuration
- `src/config/config.js` - Environment validation and database selection
- `.env` - Environment variables (not committed to git)

