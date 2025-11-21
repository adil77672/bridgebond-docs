# Database Seeding Quick Start Guide

## 🚀 Quick Start

Populate your database with complete hierarchical test data in one command:

```bash
npm run seed
```

## 📊 What Gets Created

| Entity | Count | Details |
|--------|-------|---------|
| **Super Admin** | 1 | Global system administrator |
| **Organizations** | 10 | Complete company structures |
| **Org Admins** | 20 | 2 per organization (CEO + COO) |
| **Departments** | 100 | 10 per organization |
| **Regular Users** | 100 | 10 per organization |
| **OTPs** | ~15-20 | Sample verification codes |
| **Tokens** | ~30-40 | Active session tokens |
| **TOTAL USERS** | **121** | Across all organizations |

## 🔑 Login Credentials

### Super Admin (Full System Access)
```
Email:    superadmin@bridge-bond.com
Password: Super@123
Role:     superadmin
```

### Organization Admins (Per Organization)
```
Email:    firstname.lastname@domain.com
Password: Admin@123
Role:     org_admin
Examples:
  - john.smith@techcorp.com
  - jane.doe@innovate-digital.com
```

### Regular Users (Per Organization & Department)
```
Email:    firstname123.lastname@domain.com
Password: User@123
Role:     user
Examples:
  - john456.smith@techcorp.com
  - jane789.doe@innovate-digital.com
```

## 🏢 Sample Organizations

1. TechCorp Solutions (`techcorp.com`)
2. Innovate Digital (`innovate-digital.com`)
3. FutureSoft Inc (`futuresoft.io`)
4. DataStream Analytics (`datastream.tech`)
5. CloudFirst Technologies (`cloudfirst.net`)
6. NextGen Systems (`nextgensystems.com`)
7. SmartBridge Enterprises (`smartbridge.io`)
8. Quantum Labs (`quantumlabs.tech`)
9. Digital Horizon (`digitalhorizon.com`)
10. Synergy Innovations (`synergyinnovations.net`)

## 📁 Departments (10 per Organization)

- Engineering
- Product
- Design
- Marketing
- Sales
- Human Resources
- Finance
- Operations
- Customer Support
- Legal

## 👥 User Hierarchy

```
┌─────────────────────────────────────────┐
│         Super Admin (1)                  │
│   superadmin@bridge-bond.com            │
│   Full system access                    │
└─────────────────────────────────────────┘
                 │
    ┌────────────┴────────────┐
    │                         │
┌───▼──────────────┐    ┌────▼─────────────┐
│  Organization 1   │    │ Organization 2-10│
│  techcorp.com     │    │  (similar)       │
├───────────────────┤    └──────────────────┘
│ Org Admins (2)    │
│ ├─ CEO            │
│ └─ COO            │
├───────────────────┤
│ Departments (10)  │
│ ├─ Engineering    │
│ ├─ Product        │
│ ├─ Design         │
│ └─ ...            │
├───────────────────┤
│ Users (10)        │
│ Assigned to       │
│ departments       │
└───────────────────┘
```

## ⚠️ Important Notes

### Before Running:
1. ✅ Ensure MongoDB is running
2. ✅ Configure `.env` file properly
3. ✅ Backup any existing data (script clears all data!)

### Safe Usage:
- ✅ Development environments
- ✅ Testing environments
- ✅ Local development
- ❌ **NEVER in production!**

## 📖 Full Documentation

For detailed information, see [src/scripts/README.md](src/scripts/README.md)

## 🔧 Customization

Edit quantities in `src/scripts/seedDatabase.js`:

```javascript
// Line ~394: Number of organizations
await createOrganizations(10);

// Line ~400: Departments per organization
await createDepartments(organizations, 10);

// Line ~406: Users per organization
await createUsers(organizations, allDepartments, 10);
```

## 🐛 Troubleshooting

| Issue | Solution |
|-------|----------|
| Connection error | Check MongoDB is running |
| Env var error | Verify `.env` configuration |
| Duplicate key | Script may have failed midway, clear DB manually |
| Timeout | Reduce counts in script |

## 💡 Use Cases

### API Testing
Use the generated users to test:
- Authentication flows (verified/unverified users)
- Authorization (different roles: superadmin, org_admin, user)
- Multi-tenant features (10 isolated organizations)
- Department-based access control

### Development
- Quickly reset database to known state
- Test UI with realistic data
- Develop features requiring hierarchical data
- Demo the application with sample data

### Integration Testing
- Test cross-organization isolation
- Verify role-based permissions
- Test department memberships
- Validate data relationships

---

**Last Updated:** October 28, 2025
**Script Version:** 1.0.0

