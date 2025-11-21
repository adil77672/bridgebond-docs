# Endpoint Analysis - What to Keep or Remove?

## 📊 Current Endpoint Count: 58 Total

Let me break down **EVERY endpoint** by module with its **specific purpose** so you can decide what to remove:

---

## 1️⃣ Auth Module (6 endpoints) - **CORE, KEEP ALL**

| Endpoint | Method | Purpose | Keep/Remove? |
|----------|--------|---------|--------------|
| `/auth/register` | POST | Create new user account + auto-send OTP | ✅ **ESSENTIAL** |
| `/auth/login` | POST | Step 1: Login → Get organizations list | ✅ **ESSENTIAL** |
| `/auth/select-organization` | POST | Step 2: Choose org → Get access tokens | ✅ **ESSENTIAL** |
| `/auth/logout` | POST | Logout (stateless, client-side) | ⚠️ Could remove (not needed for stateless) |
| `/auth/refresh-tokens` | POST | Refresh expired access token | ✅ **ESSENTIAL** |
| `/auth/change-password` | POST | Change password (authenticated) | ✅ **USEFUL** |

**Recommendation:** Keep all except maybe `/logout` (stateless anyway)

---

## 2️⃣ OTP Module (5 endpoints) - **KEEP ALL**

| Endpoint | Method | Purpose | Keep/Remove? |
|----------|--------|---------|--------------|
| `/otp/email-verification/send` | POST | Send OTP to verify email | ✅ **ESSENTIAL** |
| `/otp/email-verification/verify` | POST | Verify email with OTP code | ✅ **ESSENTIAL** |
| `/otp/password-reset/send` | POST | Send OTP for forgot password | ✅ **ESSENTIAL** |
| `/otp/password-reset/verify` | POST | Reset password with OTP | ✅ **ESSENTIAL** |
| `/otp/resend` | POST | Resend expired OTP | ✅ **ESSENTIAL** |

**Recommendation:** **KEEP ALL** - Core security features

---

## 3️⃣ User Module (5 endpoints) - **KEEP ALL**

| Endpoint | Method | Purpose | Keep/Remove? |
|----------|--------|---------|--------------|
| `/users` | POST | Create new user (admin only) | ✅ **ESSENTIAL** |
| `/users` | GET | List all users (org-filtered) | ✅ **ESSENTIAL** |
| `/users/:id` | GET | Get single user details | ✅ **ESSENTIAL** |
| `/users/:id` | PATCH | Update user details | ✅ **ESSENTIAL** |
| `/users/:id` | DELETE | Delete user (soft delete) | ✅ **ESSENTIAL** |

**Recommendation:** **KEEP ALL** - Basic CRUD

---

## 4️⃣ Organization Module (8 endpoints) - **SOME REDUNDANCY**

| Endpoint | Method | Purpose | Keep/Remove? |
|----------|--------|---------|--------------|
| `/organizations` | POST | Create organization (superadmin) | ✅ **ESSENTIAL** |
| `/organizations` | GET | List organizations (org-filtered) | ✅ **ESSENTIAL** |
| `/organizations/:id` | GET | Get single organization | ✅ **ESSENTIAL** |
| `/organizations/:id` | PATCH | Update organization | ✅ **ESSENTIAL** |
| `/organizations/:id` | DELETE | Delete organization (superadmin) | ✅ **ESSENTIAL** |
| `/organizations/:id/logo` | PATCH | Upload organization logo | ⚠️ **OPTIONAL** - Could merge with PATCH |
| `/organizations/:id/users` | GET | Get users in organization | ❌ **REDUNDANT** - Use `/users?organizationId=X` |
| `/organizations/:id/departments` | GET | Get departments in org | ❌ **REDUNDANT** - Use `/departments?organizationId=X` |

**Recommendation:** 
- ❌ **REMOVE** `/organizations/:id/users` (use `/users` with filter)
- ❌ **REMOVE** `/organizations/:id/departments` (use `/departments` with filter)
- ⚠️ **OPTIONAL:** Keep logo endpoint or merge with main PATCH

**Saves:** 2-3 endpoints

---

## 5️⃣ Department Module (7 endpoints) - **MAJOR REDUNDANCY**

| Endpoint | Method | Purpose | Keep/Remove? |
|----------|--------|---------|--------------|
| `/departments` | POST | Create department | ✅ **ESSENTIAL** |
| `/departments` | GET | List departments (org-filtered) | ✅ **ESSENTIAL** |
| `/departments/:id` | GET | Get single department | ✅ **ESSENTIAL** |
| `/departments/:id` | PATCH | Update department | ✅ **ESSENTIAL** |
| `/departments/:id` | DELETE | Delete department | ✅ **ESSENTIAL** |
| `/departments/by-domain/:domain` | GET | Find department by email domain | ❌ **REMOVE** - Use query param |
| `/departments/:id/users` | GET | Get users in department | ❌ **REDUNDANT** - Use `/users?departmentId=X` |

**Recommendation:** 
- ❌ **REMOVE** `/departments/by-domain/:domain` (use `/departments?domain=X`)
- ❌ **REMOVE** `/departments/:id/users` (use `/users` with filter)

**Saves:** 2 endpoints

---

## 6️⃣ Question Module (11 endpoints) - **SOME REDUNDANCY**

| Endpoint | Method | Purpose | Keep/Remove? |
|----------|--------|---------|--------------|
| `/questions` | POST | Create question | ✅ **ESSENTIAL** |
| `/questions` | GET | List questions (returns with myResponses & reactions) | ✅ **ESSENTIAL** |
| `/questions/:id` | GET | Get single question | ✅ **ESSENTIAL** |
| `/questions/:id` | PATCH | Update question | ✅ **ESSENTIAL** |
| `/questions/:id` | DELETE | Delete question | ✅ **ESSENTIAL** |
| `/questions/:id/details` | GET | Get question with FULL details | ❌ **REDUNDANT** - Merge with GET /:id |
| `/questions/:id/respond` | POST | Submit response to question | ✅ **ESSENTIAL** |
| `/questions/:id/responses` | GET | Get all responses for question | ⚠️ **OPTIONAL** - Already in main GET |
| `/questions/:id/react` | POST | Add reaction (like, love, etc.) | ✅ **ESSENTIAL** |
| `/questions/:id/react/:reactionId` | DELETE | Remove reaction | ✅ **ESSENTIAL** |
| `/questions/reactions` | GET | Get all reactions (filtered) | ❌ **REDUNDANT** - Already in question details |

**Recommendation:**
- ❌ **REMOVE** `/questions/:id/details` (merge with `GET /:id`)
- ❌ **REMOVE** `/questions/:id/responses` (already included in main GET)
- ❌ **REMOVE** `/questions/reactions` (already included in question details)

**Saves:** 3 endpoints

---

## 7️⃣ Celebration Module (5 endpoints) - **KEEP ALL**

| Endpoint | Method | Purpose | Keep/Remove? |
|----------|--------|---------|--------------|
| `/celebrations` | POST | Create celebration | ✅ **KEEP** |
| `/celebrations` | GET | List celebrations | ✅ **KEEP** |
| `/celebrations/:id` | GET | Get single celebration | ✅ **KEEP** |
| `/celebrations/:id` | PATCH | Update celebration | ✅ **KEEP** |
| `/celebrations/:id` | DELETE | Delete celebration | ✅ **KEEP** |

**Recommendation:** **KEEP ALL** - Standard CRUD

---

## 8️⃣ DOB Alert Module (5 endpoints) - **KEEP ALL**

| Endpoint | Method | Purpose | Keep/Remove? |
|----------|--------|---------|--------------|
| `/dob-alerts` | POST | Create birthday alert | ✅ **KEEP** |
| `/dob-alerts` | GET | List birthday alerts | ✅ **KEEP** |
| `/dob-alerts/:id` | GET | Get single alert | ✅ **KEEP** |
| `/dob-alerts/:id` | PATCH | Update alert | ✅ **KEEP** |
| `/dob-alerts/:id` | DELETE | Delete alert | ✅ **KEEP** |

**Recommendation:** **KEEP ALL** - Standard CRUD

---

## 9️⃣ Audit Log Module (6 endpoints) - **SOME REDUNDANCY**

| Endpoint | Method | Purpose | Keep/Remove? |
|----------|--------|---------|--------------|
| `/audit-logs` | GET | List all audit logs | ✅ **KEEP** |
| `/audit-logs/:id` | GET | Get single audit log | ✅ **KEEP** |
| `/audit-logs/user/:userId` | GET | Get logs for specific user | ❌ **REDUNDANT** - Use `/audit-logs?userId=X` |
| `/audit-logs/user/:userId/login-history` | GET | Get login history for user | ⚠️ **OPTIONAL** - Could use query filter |
| `/audit-logs/resource/:resource/:resourceId` | GET | Get logs for resource | ❌ **REDUNDANT** - Use query params |
| `/audit-logs/failed` | GET | Get failed actions only | ❌ **REDUNDANT** - Use `/audit-logs?success=false` |

**Recommendation:**
- ❌ **REMOVE** `/audit-logs/user/:userId` (use query param)
- ❌ **REMOVE** `/audit-logs/resource/:resource/:resourceId` (use query params)
- ❌ **REMOVE** `/audit-logs/failed` (use query param)
- ⚠️ **OPTIONAL:** Keep login-history as convenience endpoint

**Saves:** 3-4 endpoints

---

## 📉 Removal Summary

### Redundant Endpoints to Remove (Total: 13-14 endpoints)

#### Organizations (2-3 endpoints):
- ❌ `GET /organizations/:id/users`
- ❌ `GET /organizations/:id/departments`
- ⚠️ `PATCH /organizations/:id/logo` (optional - could merge)

#### Departments (2 endpoints):
- ❌ `GET /departments/by-domain/:domain`
- ❌ `GET /departments/:id/users`

#### Questions (3 endpoints):
- ❌ `GET /questions/:id/details`
- ❌ `GET /questions/:id/responses`
- ❌ `GET /questions/reactions`

#### Audit Logs (3-4 endpoints):
- ❌ `GET /audit-logs/user/:userId`
- ❌ `GET /audit-logs/resource/:resource/:resourceId`
- ❌ `GET /audit-logs/failed`
- ⚠️ `GET /audit-logs/user/:userId/login-history` (optional)

#### Auth (1 endpoint):
- ⚠️ `POST /auth/logout` (optional - stateless anyway)

---

## 🎯 After Cleanup

**Current:** 58 endpoints  
**After removal:** ~44-46 endpoints  
**Reduction:** ~20% cleaner API

---

## ✅ Recommended Actions

### Option 1: Conservative Cleanup (Remove 10 endpoints)
Remove only the **clearly redundant** ones:
- Organizations: 2 endpoints (users, departments)
- Departments: 2 endpoints (by-domain, users)
- Questions: 3 endpoints (details, responses, reactions)
- Audit Logs: 3 endpoints (user, resource, failed)

**New total:** 48 endpoints

### Option 2: Aggressive Cleanup (Remove 14 endpoints)
Remove all redundant + optional ones:
- All from Option 1
- Plus: logo endpoint (merge with PATCH)
- Plus: login-history (use query)
- Plus: logout (not needed)

**New total:** 44 endpoints

### Option 3: Keep As-Is
If you want convenience endpoints for frontend developers, keep everything.

---

## ❓ Your Decision

**Which option do you prefer?**

1. **Conservative** - Remove 10 clearly redundant (→ 48 endpoints)
2. **Aggressive** - Remove 14 including optional (→ 44 endpoints)
3. **Keep all** - No changes (58 endpoints)

**Or tell me specific modules to simplify!**

For example:
- "Remove all audit log extras"
- "Simplify questions module only"
- "Remove nested routes (organizations/:id/users, etc.)"

