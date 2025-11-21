# Quick Login Guide

## 🚀 Super Quick Start

### 1. Seed the Database (First Time Only)

```bash
npm run seed
```

### 2. Login with Super Admin

**Use these credentials - they're FIXED and always work:**

```json
{
  "email": "superadmin@bridge-bond.com",
  "password": "Super@123"
}
```

### 3. cURL Example

```bash
curl -X POST http://localhost:2323/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "superadmin@bridge-bond.com",
    "password": "Super@123"
  }'
```

---

## 🔍 Need Other User Emails?

The seed script generates **random** emails like `john.smith@techcorp.com`, `jane.wilson@innovate-digital.com`, etc.

### Get Real Emails - Option 1: Check Script

```bash
npm run check-users
```

This shows:
- ✅ All actual emails in the database
- ✅ Password verification
- ✅ Sample credentials ready to copy/paste

### Get Real Emails - Option 2: Check Seed Output

When you ran `npm run seed`, the console showed actual emails:

```
Org Admin Examples (2 per organization):
  1. Email: john.smith@techcorp.com          ← COPY THIS
     Password: Admin@123
     Organization: TechCorp Solutions
```

---

## 🔑 Default Passwords

| User Type | Password |
|-----------|----------|
| Super Admin | `Super@123` |
| Org Admins | `Admin@123` |
| Regular Users | `User@123` |

---

## ❌ Common Mistake

**DON'T use placeholder emails like:**
- ❌ `firstname.lastname@techcorp.com`
- ❌ `firstname.lastname@domain.com`

**DO use actual generated emails like:**
- ✅ `superadmin@bridge-bond.com` (super admin - always works!)
- ✅ `john.smith@techcorp.com` (from seed output)
- ✅ `jane.wilson@innovate-digital.com` (from seed output)

---

## 📖 Swagger Documentation

Visit: `http://localhost:2323/v1/docs`

The Swagger docs now include:
- ✅ Super admin credentials in examples
- ✅ Notes about getting real emails
- ✅ Instructions to run check script
- ✅ All required fields clearly marked

---

## 🛠️ Quick Commands

```bash
# Seed database with test data
npm run seed

# Check what users exist and their emails
npm run check-users

# Start development server
npm run dev

# View API docs
# Open: http://localhost:2323/v1/docs
```

---

## 🎯 Test Flow

### Step 1: Login
```bash
curl -X POST http://localhost:2323/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "superadmin@bridge-bond.com",
    "password": "Super@123"
  }'
```

### Step 2: Copy the Access Token
```json
{
  "user": { ... },
  "accessToken": "eyJhbGciOiJIUzI1NiIs...",  ← COPY THIS
  "refreshToken": "eyJhbGciOiJIUzI1NiIs..."
}
```

### Step 3: Use in Authenticated Requests
```bash
curl -X GET http://localhost:2323/v1/users \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..."
```

---

## 📚 More Help

- **Full API Guide:** [API_TESTING_GUIDE.md](API_TESTING_GUIDE.md)
- **Seed Details:** [SEED_QUICK_START.md](SEED_QUICK_START.md)
- **All Credentials:** [CREDENTIALS_LIST.txt](CREDENTIALS_LIST.txt)
- **Project README:** [README.md](README.md)

---

## 🆘 Still Can't Login?

1. **Make sure you seeded the database:**
   ```bash
   npm run seed
   ```

2. **Check if super admin exists:**
   ```bash
   npm run check-users
   ```

3. **Make sure server is running:**
   ```bash
   npm run dev
   ```

4. **Try the exact credentials:**
   - Email: `superadmin@bridge-bond.com`
   - Password: `Super@123`

5. **Check for typos:**
   - Email is all lowercase
   - Password is case-sensitive (`Super@123` not `super@123`)

---

**Last Updated:** October 28, 2025

