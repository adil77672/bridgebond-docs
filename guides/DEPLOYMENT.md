# Deployment Guide

## Vercel Deployment

### Quick Deploy
```bash
# 1. Install Vercel CLI
npm install -g vercel

# 2. Login
vercel login

# 3. Deploy
vercel --prod
```

### Environment Variables

Required in Vercel Dashboard:

```env
NODE_ENV=production
IS_LIVE=true
MONGODB_URL_LIVE=mongodb+srv://...
JWT_SECRET=your-secret-min-32-chars
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USERNAME=your-email@gmail.com
SMTP_PASSWORD=your-app-password
CLOUDINARY_CLOUD_NAME=your-cloud-name
CLOUDINARY_API_KEY=your-key
CLOUDINARY_API_SECRET=your-secret
ONESIGNAL_APP_ID=your-app-id
ONESIGNAL_REST_API_KEY=your-key
```

### Important Vercel Considerations

1. **Serverless Functions** - No persistent filesystem
2. **Timeout Limits** - 10s (Hobby), 60s (Pro)
3. **Cron Jobs** - Use Vercel Cron or external service
4. **File Uploads** - Use Cloudinary/S3, not local storage

### Test Deployment
```bash
curl https://your-app.vercel.app/
curl https://your-app.vercel.app/v1/docs
```

## Auto-Deploy

Push to `main` branch triggers automatic deployment.

## Troubleshooting

| Error | Solution |
|-------|----------|
| Module not found | Move to dependencies (not devDependencies) |
| Function timeout | Upgrade to Pro or optimize queries |
| MongoDB connection | Whitelist `0.0.0.0/0` in MongoDB Atlas |

## Resources

- [Vercel Docs](https://vercel.com/docs)
- [MongoDB Atlas](https://www.mongodb.com/cloud/atlas)
- [Cloudinary](https://cloudinary.com)

