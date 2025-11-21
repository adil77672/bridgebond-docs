# Cloudinary Image Upload - Quick Start Guide

## 🚀 Setup (5 Minutes)

### Step 1: Add Environment Variables

Add these to your `.env` file:

```bash
# Upload Provider Configuration
UPLOAD_PROVIDER=cloudinary

# Cloudinary Credentials
CLOUDINARY_CLOUD_NAME=your_cloud_name_here
CLOUDINARY_API_KEY=975187343871965
CLOUDINARY_API_SECRET=ufhNVCZj6EWjwkpR6k-DDBotk6k
```

**⚠️ Important**: Replace `your_cloud_name_here` with your actual Cloudinary cloud name.

### Step 2: Get Your Cloud Name

1. Go to https://cloudinary.com/console
2. Find your **Cloud Name** in the dashboard
3. Copy it to your `.env` file

### Step 3: Verify Installation

The cloudinary package is already installed! You're ready to go.

---

## 📝 Quick Usage Examples

### Example 1: Upload User Profile Image

**Route:**
```javascript
// src/routes/v1/user.route.js
import { uploadSingle, uploadUserImage } from '../../middlewares/upload.js';

router.patch(
  '/:userId/image',
  auth(),
  uploadSingle('image'),    // 1. Accept file upload
  uploadUserImage,          // 2. Process and upload to Cloudinary
  userController.updateUserImage  // 3. Save URL to database
);
```

**Controller:**
```javascript
// src/controllers/user.controller.js
const updateUserImage = catchAsync(async (req, res) => {
  const { url, publicId } = req.uploadedImage;  // Cloudinary upload result
  
  const user = await userService.updateUser(req.params.userId, {
    imageUrl: url,
    imagePublicId: publicId
  });
  
  res.send(user);
});
```

**Test:**
```bash
curl -X PATCH http://localhost:2323/v1/users/123/image \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F "image=@profile.jpg"
```

**Response:**
```json
{
  "url": "https://res.cloudinary.com/your-cloud/image/upload/v123/bridge-bond/users/abc.jpg",
  "publicId": "bridge-bond/users/abc",
  "width": 500,
  "height": 500,
  "format": "jpg",
  "provider": "cloudinary"
}
```

---

### Example 2: Upload Organization Logo

**Route:**
```javascript
router.patch(
  '/:organizationId/logo',
  auth('manageOrganizations'),
  uploadSingle('logo'),
  uploadOrganizationLogo,
  organizationController.updateLogo
);
```

**Test:**
```bash
curl -X PATCH http://localhost:2323/v1/organizations/org123/logo \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F "logo=@company-logo.png"
```

---

### Example 3: Direct Service Usage

```javascript
import uploadService from '../services/upload.service.js';

// Upload user image
const result = await uploadService.uploadUserImage(buffer, userId);

// Upload organization logo
const result = await uploadService.uploadOrganizationLogo(buffer, orgId);

// Upload celebration image
const result = await uploadService.uploadCelebrationImage(buffer);

// Generic upload with custom options
const result = await uploadService.uploadImage(buffer, {
  folder: 'custom/path',
  processing: {
    width: 800,
    height: 600,
    quality: 85
  }
});

// Delete image
await uploadService.deleteImage(publicId, 'cloudinary');
```

---

## 🎨 Image Specifications

| Type | Size | Quality | Crop | Format |
|------|------|---------|------|--------|
| **User Profile** | 500x500 | 90% | Face-centered | JPEG |
| **Organization Logo** | 400x400 | 90% | Contain | JPEG/PNG |
| **Celebration** | 1200x1200 | 85% | Inside | JPEG |

---

## 🔄 Switching to AWS S3 Later

When you're ready to switch to AWS S3, just update your `.env`:

```bash
# Change provider
UPLOAD_PROVIDER=s3

# Add S3 credentials
AWS_S3_BUCKET=your-bucket
AWS_ACCESS_KEY_ID=your-key
AWS_SECRET_ACCESS_KEY=your-secret
AWS_REGION=us-east-1
```

No code changes needed! The service layer handles the switch automatically.

---

## ✅ What's Already Done

- ✅ Cloudinary package installed
- ✅ Configuration system ready
- ✅ Upload service created
- ✅ Middleware configured
- ✅ Image optimization with Sharp
- ✅ Error handling
- ✅ Multiple provider support

---

## 🛠️ Available Middleware

```javascript
import { 
  uploadSingle,              // Single file upload
  uploadMultiple,            // Multiple files upload
  uploadUserImage,           // Process user profile image
  uploadOrganizationLogo,    // Process org logo
  uploadCelebrationImage,    // Process celebration image
  uploadToCloud,             // Generic cloud upload
  handleUploadError          // Error handler
} from '../middlewares/upload.js';
```

---

## 🔍 Accessing Upload Results

After upload middleware runs, access the result in `req.uploadedImage`:

```javascript
{
  url: "https://res.cloudinary.com/...",  // Public URL
  publicId: "bridge-bond/users/abc123",    // For deletion
  width: 500,                              // Image width
  height: 500,                             // Image height
  format: "jpg",                           // File format
  provider: "cloudinary"                   // Upload provider
}
```

---

## 🚨 Common Errors & Solutions

### Error: "Config validation error: CLOUDINARY_CLOUD_NAME"
**Solution:** Add `CLOUDINARY_CLOUD_NAME` to your `.env` file

### Error: "Invalid API credentials"
**Solution:** Verify your API key and secret in Cloudinary console

### Error: "Only image files are allowed"
**Solution:** Ensure you're uploading image files (JPEG, PNG, etc.)

### Error: "File size too large"
**Solution:** Image must be under 10MB

---

## 📚 Full Documentation

For complete documentation, see: `IMAGE_UPLOAD_SYSTEM.md`

---

## 🎯 Key Features

- **Multi-Provider**: Cloudinary, AWS S3, Local
- **Auto-Optimization**: Automatic resizing and compression
- **Face Detection**: Smart cropping for profile images
- **CDN Delivery**: Global content delivery network
- **Format Conversion**: Auto WebP for modern browsers
- **Security**: File type and size validation
- **Performance**: Memory-based processing, no temp files

---

## 📦 What's Stored in Database

Store these fields in your user/organization models:

```javascript
{
  imageUrl: String,      // The public URL
  imagePublicId: String  // For deletion (important!)
}
```

---

## 🔐 Security Features

- ✅ Authentication required
- ✅ File type validation (images only)
- ✅ File size limits (10MB)
- ✅ No temp files on disk
- ✅ Environment-based credentials
- ✅ No hardcoded secrets

---

## 🎉 You're Ready!

1. Add cloud name to `.env`
2. Use the middleware in your routes
3. Store the URL in your database
4. Done! ✨

For help, see `IMAGE_UPLOAD_SYSTEM.md` for detailed documentation.

