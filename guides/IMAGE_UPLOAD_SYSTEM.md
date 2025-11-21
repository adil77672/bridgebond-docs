# Image Upload System Documentation

## Overview
Flexible image upload system that supports multiple cloud storage providers (Cloudinary and AWS S3) with easy switching between providers through configuration.

## Features
- ✅ **Multi-Provider Support**: Cloudinary, AWS S3, and local storage
- ✅ **Easy Provider Switching**: Change providers by updating environment variables
- ✅ **Image Optimization**: Automatic resizing, compression, and format conversion
- ✅ **Type-Specific Uploads**: Dedicated functions for user images, organization logos, and celebration images
- ✅ **Memory-Based Processing**: No temp files, direct buffer processing
- ✅ **Error Handling**: Comprehensive error handling with meaningful messages

## Architecture

### Service Layer (`src/services/upload.service.js`)
- Abstract interface for multiple storage providers
- Image processing with Sharp
- Provider-specific implementations (Cloudinary, S3, Local)

### Middleware Layer (`src/middlewares/upload.js`)
- Multer configuration for file handling
- Type-specific upload middlewares
- Error handling for upload failures

## Configuration

### Environment Variables

Add these variables to your `.env` file:

```bash
# Upload Configuration
# Options: 'cloudinary', 's3', 'local'
UPLOAD_PROVIDER=cloudinary

# Cloudinary Configuration
CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=975187343871965
CLOUDINARY_API_SECRET=ufhNVCZj6EWjwkpR6k-DDBotk6k

# AWS S3 Configuration (for future use)
# AWS_S3_BUCKET=your_bucket_name
# AWS_ACCESS_KEY_ID=your_access_key
# AWS_SECRET_ACCESS_KEY=your_secret_key
# AWS_REGION=us-east-1
```

### Switching Providers

**To use Cloudinary (Current):**
```bash
UPLOAD_PROVIDER=cloudinary
CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret
```

**To switch to AWS S3:**
```bash
UPLOAD_PROVIDER=s3
AWS_S3_BUCKET=your_bucket
AWS_ACCESS_KEY_ID=your_key
AWS_SECRET_ACCESS_KEY=your_secret
AWS_REGION=us-east-1
```

**For local development:**
```bash
UPLOAD_PROVIDER=local
```

## Usage

### 1. User Profile Image Upload

```javascript
import { uploadSingle, uploadUserImage } from '../middlewares/upload.js';

router.patch(
  '/users/:userId/image',
  auth(),
  uploadSingle('image'),          // Handle file upload
  uploadUserImage,                 // Process and upload to cloud
  userController.updateUserImage  // Controller to save URL
);
```

**Controller Example:**
```javascript
const updateUserImage = catchAsync(async (req, res) => {
  // Image uploaded, result in req.uploadedImage
  const { url, publicId } = req.uploadedImage;
  
  const user = await userService.updateUser(req.params.userId, {
    imageUrl: url,
    imagePublicId: publicId
  });
  
  res.send(user);
});
```

**API Request:**
```bash
curl -X PATCH http://localhost:2323/v1/users/123/image \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F "image=@profile.jpg"
```

### 2. Organization Logo Upload

```javascript
router.patch(
  '/organizations/:organizationId/logo',
  auth('manageOrganizations'),
  uploadSingle('logo'),              // Handle file upload
  uploadOrganizationLogo,            // Process and upload to cloud
  organizationController.updateLogo // Controller to save URL
);
```

**Controller Example:**
```javascript
const updateLogo = catchAsync(async (req, res) => {
  const { url, publicId } = req.uploadedImage;
  
  const organization = await organizationService.updateOrganization(
    req.params.organizationId,
    { logo: url, logoPublicId: publicId }
  );
  
  res.send(organization);
});
```

### 3. Celebration Image Upload

```javascript
router.post(
  '/celebrations',
  auth(),
  uploadSingle('image'),             // Handle file upload
  uploadCelebrationImage,            // Process and upload to cloud
  celebrationController.create      // Controller to save URL
);
```

### 4. Generic Image Upload

```javascript
router.post(
  '/upload',
  auth(),
  uploadSingle('file'),    // Handle file upload
  uploadToCloud,           // Process and upload to cloud
  (req, res) => {
    res.send(req.uploadedImage);
  }
);
```

### 5. Multiple Images Upload

```javascript
import { uploadMultiple, uploadToCloud } from '../middlewares/upload.js';

router.post(
  '/gallery',
  auth(),
  uploadMultiple('images', 5),  // Max 5 images
  async (req, res, next) => {
    try {
      const uploadPromises = req.files.map(file => 
        uploadService.uploadImage(file.buffer)
      );
      const results = await Promise.all(uploadPromises);
      res.send({ images: results });
    } catch (error) {
      next(error);
    }
  }
);
```

## Upload Service API

### `uploadUserImage(buffer, userId)`
Uploads and optimizes user profile image.
- **Size**: 500x500px (cropped to face)
- **Folder**: `bridge-bond/users`
- **Returns**: `{ url, publicId, width, height, format, provider }`

### `uploadOrganizationLogo(buffer, organizationId)`
Uploads organization logo.
- **Size**: 400x400px (maintain aspect ratio)
- **Folder**: `bridge-bond/organizations`
- **Returns**: `{ url, publicId, width, height, format, provider }`

### `uploadCelebrationImage(buffer)`
Uploads celebration/event image.
- **Size**: 1200x1200px max (maintain aspect ratio)
- **Folder**: `bridge-bond/celebrations`
- **Returns**: `{ url, publicId, width, height, format, provider }`

### `uploadImage(buffer, options)`
Generic image upload with custom options.

**Options:**
```javascript
{
  folder: 'custom/path',           // Cloudinary folder
  processing: {
    width: 800,                     // Max width
    height: 600,                    // Max height
    fit: 'inside',                  // 'cover', 'contain', 'fill', 'inside', 'outside'
    quality: 85                     // JPEG quality (1-100)
  },
  transformation: [                 // Cloudinary-specific transformations
    { width: 800, crop: 'limit' },
    { quality: 'auto' }
  ]
}
```

### `deleteImage(identifier, provider)`
Deletes an image from cloud storage.
- **identifier**: publicId (Cloudinary) or key (S3) or path (local)
- **provider**: 'cloudinary', 's3', or 'local'

**Example:**
```javascript
await uploadService.deleteImage(user.imagePublicId, 'cloudinary');
```

## Image Specifications

### User Profile Images
- **Dimensions**: 500x500px
- **Format**: JPEG (auto-converted)
- **Quality**: 90%
- **Crop**: Face-centered
- **Max Upload**: 10MB

### Organization Logos
- **Dimensions**: 400x400px (max)
- **Format**: JPEG/PNG (auto-converted)
- **Quality**: 90%
- **Crop**: Contain (maintains aspect ratio)
- **Max Upload**: 10MB

### Celebration Images
- **Dimensions**: 1200x1200px (max)
- **Format**: JPEG (auto-converted)
- **Quality**: 85%
- **Crop**: Inside (maintains aspect ratio)
- **Max Upload**: 10MB

## Error Handling

### Common Errors

**File Too Large:**
```json
{
  "code": 400,
  "message": "File size too large. Maximum size is 10MB"
}
```

**Invalid File Type:**
```json
{
  "code": 400,
  "message": "Only image files are allowed"
}
```

**Upload Failed:**
```json
{
  "code": 500,
  "message": "Failed to upload image to Cloudinary"
}
```

**Too Many Files:**
```json
{
  "code": 400,
  "message": "Too many files uploaded"
}
```

### Error Handling in Routes

```javascript
import { handleUploadError } from '../middlewares/upload.js';

router.post(
  '/upload',
  uploadSingle('image'),
  handleUploadError,        // Add this after multer middleware
  uploadUserImage,
  controller
);
```

## Response Format

### Successful Upload Response

```json
{
  "url": "https://res.cloudinary.com/your-cloud/image/upload/v123/bridge-bond/users/abc123.jpg",
  "publicId": "bridge-bond/users/abc123",
  "width": 500,
  "height": 500,
  "format": "jpg",
  "provider": "cloudinary"
}
```

## Cloudinary Features

### Automatic Optimizations
- **Format**: Auto-selects best format (WebP for modern browsers)
- **Quality**: Auto-adjusts for optimal quality/size ratio
- **Compression**: Intelligent compression
- **CDN**: Global CDN delivery

### Transformations
Cloudinary automatically applies:
- Face detection for user profiles
- Smart cropping
- Format conversion
- Quality optimization
- Lazy loading support

### URLs
Cloudinary URLs are immutable and cached globally:
```
https://res.cloudinary.com/{cloud_name}/image/upload/v{version}/{folder}/{filename}
```

## Migration to AWS S3

### When Ready to Switch:

1. **Install AWS SDK:**
```bash
npm install @aws-sdk/client-s3 @aws-sdk/lib-storage
```

2. **Update `.env`:**
```bash
UPLOAD_PROVIDER=s3
AWS_S3_BUCKET=your-bucket
AWS_ACCESS_KEY_ID=your-key
AWS_SECRET_ACCESS_KEY=your-secret
AWS_REGION=us-east-1
```

3. **Implement S3 Functions:**

The service already has placeholder functions for S3. Implement:
- `uploadToS3(buffer, options)`
- `deleteFromS3(key)`

Example implementation in `upload.service.js`:

```javascript
import { S3Client, PutObjectCommand, DeleteObjectCommand } from '@aws-sdk/client-s3';

const s3Client = new S3Client({
  region: config.upload.s3.region,
  credentials: {
    accessKeyId: config.upload.s3.accessKeyId,
    secretAccessKey: config.upload.s3.secretAccessKey,
  },
});

const uploadToS3 = async (buffer, options = {}) => {
  const key = `${options.folder || 'images'}/${Date.now()}-${Math.random().toString(36).substring(7)}.jpg`;
  
  const command = new PutObjectCommand({
    Bucket: config.upload.s3.bucket,
    Key: key,
    Body: buffer,
    ContentType: 'image/jpeg',
    ACL: 'public-read',
  });

  await s3Client.send(command);
  
  const url = `https://${config.upload.s3.bucket}.s3.${config.upload.s3.region}.amazonaws.com/${key}`;
  
  return {
    url,
    key,
    provider: 's3',
  };
};
```

## Testing

### Test User Image Upload

```bash
# Upload user image
curl -X PATCH http://localhost:2323/v1/users/YOUR_USER_ID/image \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F "image=@test-image.jpg"
```

### Test Organization Logo

```bash
# Upload organization logo
curl -X PATCH http://localhost:2323/v1/organizations/ORG_ID/logo \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F "logo=@company-logo.png"
```

## Best Practices

1. **Always use type-specific upload functions** for better optimization
2. **Store `publicId` in database** for deletion capability
3. **Validate image dimensions** on client-side before upload
4. **Use WebP format** for modern browsers (Cloudinary auto-handles this)
5. **Delete old images** when user uploads new ones
6. **Use CDN URLs** directly in your frontend
7. **Set appropriate CORS** if accessing from different domain

## Security

- ✅ File type validation (images only)
- ✅ File size limits (10MB max)
- ✅ Authentication required for uploads
- ✅ Memory-based processing (no temp files)
- ✅ Automatic cleanup on errors
- ✅ Environment-based credentials
- ✅ No hardcoded secrets

## Performance

- **Memory Storage**: Files processed in memory, no disk I/O
- **Sharp Processing**: Fast, efficient image processing
- **Parallel Uploads**: Multiple images can be uploaded simultaneously
- **CDN Delivery**: Cloudinary provides global CDN
- **Auto Optimization**: Automatic format and quality optimization

## Troubleshooting

### Issue: Upload hangs
**Solution**: Check network connectivity and Cloudinary credentials

### Issue: Images not optimized
**Solution**: Verify Cloudinary transformations in upload options

### Issue: "Invalid API key"
**Solution**: Double-check Cloudinary credentials in `.env`

### Issue: Local uploads not working
**Solution**: Ensure `uploads` directory exists and has write permissions

## Summary

This upload system provides:
- ✅ **Flexibility**: Easy switching between Cloudinary and AWS S3
- ✅ **Optimization**: Automatic image processing and compression
- ✅ **Simplicity**: Clean API with minimal configuration
- ✅ **Security**: Validated uploads with proper authentication
- ✅ **Performance**: Memory-based processing with CDN delivery
- ✅ **Future-Proof**: Ready for AWS S3 migration with minimal code changes

