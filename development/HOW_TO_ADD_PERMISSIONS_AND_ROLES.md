# 🎯 How to Add Permissions and Roles

## 📝 Table of Contents
1. [Adding a New Permission](#adding-a-new-permission)
2. [Adding a New Role](#adding-a-new-role)
3. [Examples](#examples)
4. [Best Practices](#best-practices)

---

## 🔹 Adding a New Permission

### Step 1: Add Permission to `roles.js`

**File:** `src/config/roles.js`

Add the permission to the appropriate role(s):

```javascript
const allRoles = {
  user: [
    'getProfile',
    'updateProfile',
    'viewReports',  // ✅ NEW PERMISSION
  ],
  
  org_admin: [
    // ... existing permissions
    'viewReports',      // ✅ NEW PERMISSION
    'exportReports',    // ✅ NEW PERMISSION (admin only)
  ],
  
  superadmin: [
    // ... existing permissions
    'viewReports',      // ✅ NEW PERMISSION
    'exportReports',    // ✅ NEW PERMISSION
    'deleteReports',    // ✅ NEW PERMISSION (superadmin only)
  ],
};
```

### Step 2: Use Permission in Routes

**File:** `src/routes/v1/report.route.js` (example)

```javascript
import express from 'express';
import auth from '../../middlewares/auth.js';
import permission from '../../middlewares/permission.js';
import reportController from '../../controllers/report.controller.js';

const router = express.Router();

// All users can view reports
router.get('/', 
  auth(), 
  permission('viewReports'),  // ✅ Use the new permission
  reportController.getReports
);

// Only admins can export
router.get('/export', 
  auth(), 
  permission('exportReports'),  // ✅ Use the new permission
  reportController.exportReports
);

// Only superadmin can delete
router.delete('/:reportId', 
  auth(), 
  permission('deleteReports'),  // ✅ Use the new permission
  reportController.deleteReport
);

export default router;
```

### Step 3: Use Permission in Controllers (Optional)

**File:** `src/controllers/report.controller.js`

```javascript
import { hasPermission } from '../middlewares/permission.js';

const getReports = catchAsync(async (req, res) => {
  // Check permission programmatically if needed
  if (hasPermission(req.user, 'exportReports')) {
    // Show export button in response
    data.canExport = true;
  }
  
  res.send(data);
});
```

### ✅ That's it! Your new permission is ready.

---

## 🔹 Adding a New Role

### Step 1: Add Role to `roles.js`

**File:** `src/config/roles.js`

```javascript
const allRoles = {
  user: [
    // ... existing permissions
  ],
  
  org_admin: [
    // ... existing permissions
  ],
  
  superadmin: [
    // ... existing permissions
  ],
  
  // ✅ NEW ROLE
  moderator: [
    // Give it the permissions it needs
    'getProfile',
    'updateProfile',
    'getOrganization',
    'getDepartments',
    'getOrgUsers',
    'getQuestions',
    'manageQuestions',  // Moderators can manage questions
    'viewReports',
    'moderateContent',  // New permission for moderators
  ],
};
```

### Step 2: Update Role Definition

**File:** `src/config/roles.js`

Make sure the role is exported:

```javascript
const roles = Object.keys(allRoles);  // ✅ Automatically includes new role
const roleRights = new Map(Object.entries(allRoles));

export { roles, roleRights };
```

### Step 3: Add Role to User Model (if not already flexible)

**File:** `src/models/user.model.js`

Update the role enum:

```javascript
role: {
  type: String,
  enum: ['user', 'org_admin', 'superadmin', 'moderator'],  // ✅ Add new role
  default: 'user',
},
```

### Step 4: Update Validation (if needed)

**File:** `src/validations/user.validation.js`

```javascript
role: Joi.string()
  .valid('user', 'org_admin', 'superadmin', 'moderator')  // ✅ Add new role
  .default('user'),
```

### Step 5: Use Role in Routes

```javascript
import { requireRole } from '../../middlewares/permission.js';

// Only moderators and admins can access
router.post('/moderate', 
  auth(), 
  requireRole('moderator', 'org_admin', 'superadmin'),
  controller.moderate
);
```

### ✅ Your new role is ready!

---

## 📚 Examples

### Example 1: Add "View Analytics" Permission

**Goal:** Users can view analytics, but only admins can export them.

#### Step 1: Add to `roles.js`
```javascript
user: [
  // ... existing
  'viewAnalytics',  // ✅ NEW
],

org_admin: [
  // ... existing
  'viewAnalytics',   // ✅ NEW
  'exportAnalytics', // ✅ NEW (admin only)
],

superadmin: [
  // ... existing
  'viewAnalytics',   // ✅ NEW
  'exportAnalytics', // ✅ NEW
],
```

#### Step 2: Create route
```javascript
// src/routes/v1/analytics.route.js
router.get('/', 
  auth(), 
  permission('viewAnalytics'), 
  analyticsController.getAnalytics
);

router.get('/export', 
  auth(), 
  permission('exportAnalytics'), 
  analyticsController.exportAnalytics
);
```

---

### Example 2: Add "Billing Admin" Role

**Goal:** Create a new role that can only manage billing, nothing else.

#### Step 1: Add to `roles.js`
```javascript
const allRoles = {
  // ... existing roles
  
  billing_admin: [
    'getProfile',
    'updateProfile',
    'viewBilling',
    'manageBilling',
    'viewInvoices',
    'exportInvoices',
    'managePaymentMethods',
  ],
};
```

#### Step 2: Update user model
```javascript
// src/models/user.model.js
role: {
  type: String,
  enum: ['user', 'org_admin', 'superadmin', 'billing_admin'],
  default: 'user',
},
```

#### Step 3: Create billing routes
```javascript
// src/routes/v1/billing.route.js
router.get('/invoices', 
  auth(), 
  permission('viewInvoices'), 
  billingController.getInvoices
);

router.post('/payment-methods', 
  auth(), 
  permission('managePaymentMethods'), 
  billingController.addPaymentMethod
);
```

---

### Example 3: Add Permission Check in Controller

```javascript
// src/controllers/user.controller.js
import { hasPermission, hasAnyPermission } from '../middlewares/permission.js';

const getUsers = catchAsync(async (req, res) => {
  // Check if user can export
  const canExport = hasPermission(req.user, 'exportUsers');
  
  // Check if user has ANY admin permission
  const isAdmin = hasAnyPermission(
    req.user, 
    'manageUsers', 
    'manageOrgUsers', 
    'manageOrganizations'
  );
  
  const users = await userService.queryUsers(filter);
  
  res.send({
    users,
    canExport,
    isAdmin,
  });
});
```

---

## ✅ Best Practices

### 1. **Permission Naming Convention**
```javascript
// ✅ GOOD: Clear, descriptive, verb-based
'getUsers'
'manageUsers'
'updateOrganization'
'deleteReports'
'viewAnalytics'

// ❌ BAD: Vague, unclear
'users'
'access'
'stuff'
'manage'
```

### 2. **Permission Granularity**
```javascript
// ✅ GOOD: Specific permissions
'viewReports'
'createReports'
'updateReports'
'deleteReports'
'exportReports'

// ❌ BAD: Too broad
'manageReports'  // Does this include view? delete? export?
```

### 3. **Role Hierarchy**
```javascript
// ✅ GOOD: Clear hierarchy
user         → Basic permissions
org_admin    → All USER permissions + org management
superadmin   → All ORG_ADMIN permissions + system-wide access

// Don't randomly mix permissions across roles
```

### 4. **Permission Groups**
```javascript
// Group related permissions together in roles.js
user: [
  // Profile
  'getProfile',
  'updateProfile',
  
  // Organization
  'getOrganization',
  'getDepartments',
  
  // Questions
  'getQuestions',
  'answerQuestions',
],
```

### 5. **Use Middleware Correctly**
```javascript
// ✅ GOOD: Auth first, then permission
router.get('/users', auth(), permission('getUsers'), controller);

// ✅ GOOD: Multiple permissions (user needs ALL)
router.post('/users', auth(), permission('manageUsers', 'getOrganizations'), controller);

// ✅ GOOD: Any permission (user needs ONE)
router.post('/users', auth(), permissionAny('manageUsers', 'manageOrgUsers'), controller);

// ❌ BAD: Permission without auth
router.get('/users', permission('getUsers'), controller);
```

### 6. **Document Your Permissions**
```javascript
// ✅ GOOD: Add comments explaining what each permission does
const allRoles = {
  user: [
    'viewReports',    // Can view all reports in their organization
    'createReports',  // Can create new reports
    // Cannot delete or export (admin only)
  ],
};
```

---

## 🔍 Quick Checklist

When adding a **new permission**:
- [ ] Add to appropriate roles in `src/config/roles.js`
- [ ] Use in routes with `permission('newPermission')`
- [ ] Test with different roles
- [ ] Update documentation if needed

When adding a **new role**:
- [ ] Add to `src/config/roles.js` with permissions
- [ ] Update `src/models/user.model.js` enum
- [ ] Update `src/validations/user.validation.js`
- [ ] Test role creation and permissions
- [ ] Update Swagger/API documentation
- [ ] Update this guide with examples

---

## 🚀 Testing Your Changes

```javascript
// Test in your terminal
node -e "
import('./src/config/roles.js').then(({ roles, roleRights }) => {
  console.log('Available Roles:', roles);
  console.log('Moderator Permissions:', roleRights.get('moderator'));
});
"
```

---

## 📖 Related Documentation

- `PERMISSION_SYSTEM_SUMMARY.md` - Overview of the permission system
- `src/middlewares/permission.js` - Permission middleware implementation
- `src/config/roles.js` - All roles and permissions

---

**Last Updated:** $(date)

