# Environment Variables for Vercel Deployment

## 📋 Copy-Paste Template for Vercel

Add these to **Vercel Dashboard** → **Your Project** → **Settings** → **Environment Variables**

---

## ✅ Required Variables

```env
NODE_ENV=production
IS_LIVE=true
MONGODB_URL_STAGING=mongodb+srv://username:password@cluster.mongodb.net/bridge-bond-staging
MONGODB_URL_LIVE=mongodb+srv://username:password@cluster.mongodb.net/bridge-bond-live
JWT_SECRET=your-super-secret-jwt-key-minimum-32-characters-long
JWT_ACCESS_EXPIRATION_MINUTES=2880
JWT_REFRESH_EXPIRATION_DAYS=5
JWT_RESET_PASSWORD_EXPIRATION_MINUTES=10
JWT_VERIFY_EMAIL_EXPIRATION_MINUTES=10
```

---

## 🔧 Optional (But Recommended)

### Push Notifications
```env
# Provider: onesignal, fcm, or none (default: onesignal)
NOTIFICATION_PROVIDER=onesignal

# OneSignal Configuration
ONESIGNAL_APP_ID=your-onesignal-app-id
ONESIGNAL_REST_API_KEY=your-onesignal-rest-api-key
ONESIGNAL_USER_AUTH_KEY=your-onesignal-user-auth-key

# Firebase Cloud Messaging (alternative to OneSignal)
FCM_SERVER_KEY=your-fcm-server-key
```

### Cloudinary (Image Upload)
```env
UPLOAD_PROVIDER=cloudinary
CLOUDINARY_CLOUD_NAME=your-cloudinary-cloud-name
CLOUDINARY_API_KEY=your-cloudinary-api-key
CLOUDINARY_API_SECRET=your-cloudinary-api-secret
```

### AWS S3 (Alternative to Cloudinary)
```env
UPLOAD_PROVIDER=s3
AWS_S3_BUCKET=your-bucket-name
AWS_ACCESS_KEY_ID=your-access-key-id
AWS_SECRET_ACCESS_KEY=your-secret-access-key
AWS_REGION=us-east-1
```

### Base URL (Auto-set by Vercel)
```env
BASE_URL=https://your-app.vercel.app
```

---

## 📝 Variable Descriptions

| Variable | Description | Example |
|----------|-------------|---------|
| `NODE_ENV` | Environment mode | `production` |
| `IS_LIVE` | Use live database | `true` or `false` |
| `MONGODB_URL_STAGING` | Staging MongoDB connection | `mongodb+srv://...` |
| `MONGODB_URL_LIVE` | Production MongoDB connection | `mongodb+srv://...` |
| `JWT_SECRET` | Secret key for JWT tokens (min 32 chars) | `super-secret-key-123456789...` |
| `JWT_ACCESS_EXPIRATION_MINUTES` | Access token expiry (minutes) | `2880` (2 days) |
| `JWT_REFRESH_EXPIRATION_DAYS` | Refresh token expiry (days) | `5` |
| `SMTP_HOST` | Email server host | `smtp.gmail.com` |
| `SMTP_PORT` | Email server port | `587` |
| `SMTP_SERVICE` | Email service provider | `gmail` |
| `SMTP_USERNAME` | Email account username | `your-email@gmail.com` |
| `SMTP_PASSWORD` | Email account password/app password | App-specific password |
| `EMAIL_FROM` | Sender email address | `noreply@yourdomain.com` |
| `NOTIFICATION_PROVIDER` | Push notification provider | `onesignal`, `fcm`, or `none` |
| `ONESIGNAL_APP_ID` | OneSignal application ID | From OneSignal dashboard |
| `ONESIGNAL_REST_API_KEY` | OneSignal REST API key | From OneSignal dashboard |
| `ONESIGNAL_USER_AUTH_KEY` | OneSignal user auth key | From OneSignal dashboard |
| `FCM_SERVER_KEY` | Firebase Cloud Messaging key | From Firebase console |
| `UPLOAD_PROVIDER` | File upload provider | `cloudinary`, `s3`, or `local` |
| `CLOUDINARY_CLOUD_NAME` | Cloudinary account name | From Cloudinary dashboard |
| `CLOUDINARY_API_KEY` | Cloudinary API key | From Cloudinary dashboard |
| `CLOUDINARY_API_SECRET` | Cloudinary API secret | From Cloudinary dashboard |
| `AWS_S3_BUCKET` | AWS S3 bucket name | `my-bucket` |
| `AWS_ACCESS_KEY_ID` | AWS access key | From AWS IAM |
| `AWS_SECRET_ACCESS_KEY` | AWS secret key | From AWS IAM |
| `AWS_REGION` | AWS region | `us-east-1` |
| `BASE_URL` | Application base URL | Auto-set by Vercel |

---

## 🔐 Security Best Practices

### 1. **JWT_SECRET**
- Use minimum 32 characters
- Use random, complex string
- Generate with: `openssl rand -base64 32`

### 2. **SMTP_PASSWORD**
- For Gmail, use **App-Specific Password**:
  1. Go to Google Account → Security
  2. Enable 2FA
  3. Generate App Password
  4. Use that password (not your regular Gmail password)

### 3. **MongoDB Connection**
- Use MongoDB Atlas with IP whitelist
- For Vercel, whitelist `0.0.0.0/0` (all IPs)
- Or use specific Vercel IP ranges

### 4. **Never Commit Secrets**
- Keep `.env` in `.gitignore`
- Use Vercel Dashboard for production secrets
- Use different secrets for staging/production

---

## 🚀 Quick Setup Commands

### Generate JWT Secret
```bash
openssl rand -base64 32
```

### Test MongoDB Connection
```bash
mongosh "mongodb+srv://username:password@cluster.mongodb.net/database"
```

### Test SMTP Connection
```bash
telnet smtp.gmail.com 587
```

---

## 📚 Additional Resources

- [MongoDB Atlas Setup](https://www.mongodb.com/docs/atlas/getting-started/)
- [Gmail App Passwords](https://support.google.com/accounts/answer/185833)
- [Cloudinary Setup](https://cloudinary.com/documentation/node_integration)
- [OneSignal Setup](https://documentation.onesignal.com/docs/accounts-and-keys)
- [Vercel Environment Variables](https://vercel.com/docs/concepts/projects/environment-variables)

---

## ✅ Checklist

- [ ] All required variables added to Vercel
- [ ] JWT_SECRET is strong (min 32 chars)
- [ ] MongoDB Atlas configured with correct IP whitelist
- [ ] Gmail App-Specific Password created
- [ ] Cloudinary account created and configured
- [ ] OneSignal app created and configured
- [ ] Test each service connection
- [ ] Deploy and verify

---

**Last Updated:** October 29, 2024

