# Sample Login Credentials

This document provides sample credentials for testing the Bridge Bond application after running `npm run seed`.

## 🔐 Authentication Credentials

### 1. Super Admin (Global Access)

| Field | Value |
|-------|-------|
| **Email** | superadmin@bridge-bond.com |
| **Password** | Super@123 |
| **Role** | superadmin |
| **Access Level** | Full system access across all organizations |
| **Job Title** | Super Administrator |
| **Department** | N/A (Global) |
| **Organization** | N/A (Global) |

---

## 🏢 Organization 1: TechCorp Solutions

**Domain:** techcorp.com

### Organization Admins

#### Admin 1 - CEO
| Field | Value |
|-------|-------|
| **Email** | *Randomly generated: firstname.lastname@techcorp.com* |
| **Password** | Admin@123 |
| **Role** | org_admin |
| **Organization** | TechCorp Solutions |
| **Department** | Operations |
| **Job Title** | Chief Executive Officer |
| **Access Level** | Full access to TechCorp Solutions |

#### Admin 2 - COO
| Field | Value |
|-------|-------|
| **Email** | *Randomly generated: firstname.lastname@techcorp.com* |
| **Password** | Admin@123 |
| **Role** | org_admin |
| **Organization** | TechCorp Solutions |
| **Department** | Operations |
| **Job Title** | Chief Operating Officer |
| **Access Level** | Full access to TechCorp Solutions |

### Regular Users (Sample 2 of 10)

#### User 1
| Field | Value |
|-------|-------|
| **Email** | *Randomly generated: firstname123.lastname@techcorp.com* |
| **Password** | User@123 |
| **Role** | user |
| **Organization** | TechCorp Solutions |
| **Department** | Engineering (or randomly assigned) |
| **Job Title** | Software Engineer (or randomly assigned) |
| **Access Level** | Department-level access |

#### User 2
| Field | Value |
|-------|-------|
| **Email** | *Randomly generated: firstname456.lastname@techcorp.com* |
| **Password** | User@123 |
| **Role** | user |
| **Organization** | TechCorp Solutions |
| **Department** | Product (or randomly assigned) |
| **Job Title** | Product Manager (or randomly assigned) |
| **Access Level** | Department-level access |

### Departments in TechCorp Solutions
1. Engineering
2. Product
3. Design
4. Marketing
5. Sales
6. Human Resources
7. Finance
8. Operations
9. Customer Support
10. Legal

---

## 🏢 Organization 2: Innovate Digital

**Domain:** innovate-digital.com

### Organization Admins

#### Admin 1 - CEO
| Field | Value |
|-------|-------|
| **Email** | *Randomly generated: firstname.lastname@innovate-digital.com* |
| **Password** | Admin@123 |
| **Role** | org_admin |
| **Organization** | Innovate Digital |
| **Department** | Operations |
| **Job Title** | Chief Executive Officer |
| **Access Level** | Full access to Innovate Digital |

#### Admin 2 - COO
| Field | Value |
|-------|-------|
| **Email** | *Randomly generated: firstname.lastname@innovate-digital.com* |
| **Password** | Admin@123 |
| **Role** | org_admin |
| **Organization** | Innovate Digital |
| **Department** | Operations |
| **Job Title** | Chief Operating Officer |
| **Access Level** | Full access to Innovate Digital |

### Regular Users (Sample 2 of 10)

#### User 1
| Field | Value |
|-------|-------|
| **Email** | *Randomly generated: firstname789.lastname@innovate-digital.com* |
| **Password** | User@123 |
| **Role** | user |
| **Organization** | Innovate Digital |
| **Department** | Design (or randomly assigned) |
| **Job Title** | UI/UX Designer (or randomly assigned) |
| **Access Level** | Department-level access |

#### User 2
| Field | Value |
|-------|-------|
| **Email** | *Randomly generated: firstname012.lastname@innovate-digital.com* |
| **Password** | User@123 |
| **Role** | user |
| **Organization** | Innovate Digital |
| **Department** | Marketing (or randomly assigned) |
| **Job Title** | Marketing Manager (or randomly assigned) |
| **Access Level** | Department-level access |

### Departments in Innovate Digital
1. Engineering
2. Product
3. Design
4. Marketing
5. Sales
6. Human Resources
7. Finance
8. Operations
9. Customer Support
10. Legal

---

## 📋 All Organizations & Domains

| # | Organization Name | Domain | Admin Count | User Count | Dept Count |
|---|-------------------|--------|-------------|------------|------------|
| 1 | TechCorp Solutions | techcorp.com | 2 | 10 | 10 |
| 2 | Innovate Digital | innovate-digital.com | 2 | 10 | 10 |
| 3 | FutureSoft Inc | futuresoft.io | 2 | 10 | 10 |
| 4 | DataStream Analytics | datastream.tech | 2 | 10 | 10 |
| 5 | CloudFirst Technologies | cloudfirst.net | 2 | 10 | 10 |
| 6 | NextGen Systems | nextgensystems.com | 2 | 10 | 10 |
| 7 | SmartBridge Enterprises | smartbridge.io | 2 | 10 | 10 |
| 8 | Quantum Labs | quantumlabs.tech | 2 | 10 | 10 |
| 9 | Digital Horizon | digitalhorizon.com | 2 | 10 | 10 |
| 10 | Synergy Innovations | synergyinnovations.net | 2 | 10 | 10 |

---

## 🎯 Quick Test Scenarios

### Testing Super Admin
```
Login: superadmin@bridge-bond.com
Password: Super@123
Expected: Access to all organizations and departments
```

### Testing Org Admin
```
Login: Any admin from organization (check console output after seeding)
Password: Admin@123
Expected: Full access to their organization only
```

### Testing Regular User
```
Login: Any user from organization (check console output after seeding)
Password: User@123
Expected: Access to their organization and assigned department
```

---

## 📝 How to Get Exact Credentials

Since emails are randomly generated, after running `npm run seed`, the script outputs the actual credentials to the console:

```bash
npm run seed
```

Look for this section in the output:

```
Sample Login Credentials:
=================================
Super Admin (Global Access):
  Email: superadmin@bridge-bond.com
  Password: Super@123
  Role: superadmin

Org Admin Examples (2 per organization):
  1. Email: john.smith@techcorp.com
     Password: Admin@123
     Organization: TechCorp Solutions
     Job Title: Chief Executive Officer
  2. Email: jane.doe@techcorp.com
     Password: Admin@123
     Organization: TechCorp Solutions
     Job Title: Chief Operating Officer
  ...

Regular User Examples (10 per organization):
  1. Email: michael456.johnson@techcorp.com
     Password: User@123
     Job Title: Software Engineer
  ...
```

---

## 🔍 Finding Users in Database

You can also query the database directly:

### Super Admin
```javascript
db.users.findOne({ role: 'superadmin' })
```

### Org Admins for Specific Organization
```javascript
db.users.find({ 
  role: 'org_admin',
  'organizationMemberships.organizationId': ObjectId('...')
})
```

### Regular Users in Department
```javascript
db.users.find({ 
  role: 'user',
  'departmentMemberships.departmentId': ObjectId('...')
})
```

---

## 🛡️ Security Notes

⚠️ **These are TEST credentials only!**

- Never use these passwords in production
- Change all default passwords before deploying
- The seed script should only be run in development/testing environments
- All passwords meet minimum requirements (8+ chars, letter + number)

---

## 📞 Common Job Titles by Department

**Engineering:**
- Software Engineer
- Senior Developer
- DevOps Engineer
- Full Stack Developer
- Technical Lead

**Product:**
- Product Manager
- Business Analyst

**Design:**
- UI/UX Designer

**Marketing:**
- Marketing Manager

**Sales:**
- Sales Representative

**Human Resources:**
- HR Manager

**Finance:**
- Financial Analyst

**Operations:**
- Operations Manager
- Chief Executive Officer
- Chief Operating Officer

**Customer Support:**
- Customer Success Manager

**Legal:**
- Legal positions

---

**Last Updated:** October 28, 2025  
**Script Version:** 1.0.0

