# Merge.dev Implementation Guide

## Overview

This guide explains how Merge.dev integration works in Bridge Bond, based on the official Merge.dev documentation and best practices.

**Reference Documentation:**
- [Merge.dev Get Started](https://docs.merge.dev/get-started/introduction/)
- [Merge.dev API Reference](https://docs.merge.dev/api/hris/overview)
- [Merge.dev SDK](https://github.com/merge-api/merge-node-client)

---

## Architecture

### Merge.dev as Super Integration

**Merge.dev is a super integration that covers ALL HRIS platforms:**

```
┌─────────────────────────────────────────────────────────┐
│              Bridge Bond Application                     │
│                                                           │
│  ┌──────────────────────────────────────────────────┐   │
│  │     hrIntegration.service.js                      │   │
│  │     (Existing Integration Service)                │   │
│  └──────────────┬────────────────────────────────────┘   │
│                 │                                         │
│                 ▼                                         │
│  ┌──────────────────────────────────────────────────┐   │
│  │     MergeService                                  │   │
│  │     (High-level Service Layer)                    │   │
│  └──────────────┬────────────────────────────────────┘   │
│                 │                                         │
│                 ▼                                         │
│  ┌──────────────────────────────────────────────────┐   │
│  │     MergeClient (Official SDK Wrapper)            │   │
│  │     @mergeapi/merge-node-client                   │   │
│  └──────────────┬────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────┐
│              Merge.dev Platform                          │
│  (Super Integration - Covers ALL HRIS Platforms)         │
│                                                           │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐ │
│  │BambooHR │  │   ADP    │  │ Workday  │  │  Gusto   │ │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘ │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐ │
│  │Rippling  │  │Paylocity │  │  Namely  │  │ 40+ more │ │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘ │
└─────────────────────────────────────────────────────────┘
```

### Organization-Based Platform Connections

**Key Points:**
- **Single Merge.dev Account**: Shared API key (stored in environment)
- **Organization Account Tokens**: Each organization has their own token
- **Multiple Platform Connections**: Each organization can connect multiple HRIS platforms through Merge.dev
- **Separate Management**: Platform connections are managed separately per organization in Merge.dev dashboard

**Example:**
- **Org A**: Connects BambooHR + ADP via Merge.dev
- **Org B**: Connects Workday only via Merge.dev
- **Org C**: Connects Gusto + Rippling via Merge.dev

---

## Setup Process

### 1. Create Merge.dev Account

1. Sign up at [merge.dev](https://www.merge.dev)
2. Get your API key from [API Keys](https://app.merge.dev/keys)
3. Store API key in environment: `MERGE_API_KEY`

### 2. Connect Organizations

For each organization:

1. **Create Account Token** in Merge.dev dashboard
2. **Connect HRIS Platforms** in Merge.dev:
   - Go to Linked Accounts
   - Connect desired platforms (BambooHR, ADP, Workday, etc.)
   - Each platform connection is separate
3. **Store Account Token** in Bridge Bond integration

### 3. Connect Bridge Bond to Merge.dev

```http
POST /v1/hr-integrations/organizations/{organizationId}/connect
{
  "platform": "merge",
  "credentials": {
    "apiKey": "your-merge-api-key",
    "accountToken": "org-specific-account-token"
  }
}
```

---

## Using the Official SDK

### Installation

```bash
npm install --save @mergeapi/merge-node-client
```

### Basic Usage

```javascript
import { MergeClient } from '@mergeapi/merge-node-client';

const merge = new MergeClient({
  apiKey: 'YOUR_API_KEY',
  accountToken: 'YOUR_ACCOUNT_TOKEN',
});

// Fetch employees
const employees = await merge.hris.employees.list();

// Pagination
let response = await merge.hris.employees.list({ pageSize: 100 });
while (response.next) {
  response = await merge.hris.employees.list({
    cursor: response.next,
    pageSize: 100,
  });
}
```

### Categories

The SDK supports multiple categories:

```javascript
merge.hris.employees.list()      // HRIS
merge.ats.candidates.list()      // ATS
merge.accounting.contacts.list()  // Accounting
merge.ticketing.tickets.create() // Ticketing
merge.crm.contacts.list()        // CRM
merge.filestorage.files.list()   // File Storage
```

---

## Data Model

### Merge.dev Common Model

Merge.dev normalizes data across all platforms using a Common Model:

```javascript
{
  id: "employee-id",
  employee_number: "EMP001",
  first_name: "John",
  last_name: "Doe",
  display_name: "John Doe",
  email: "john.doe@example.com",
  date_of_birth: "1990-01-15",
  employments: [
    {
      job_title: "Software Engineer",
      department: "Engineering",
      start_date: "2024-01-15",
      end_date: null,
    }
  ],
  home_location: { ... },
  work_location: { ... },
  phone_numbers: [ ... ],
  custom_fields: { ... }
}
```

### Field Mapping

Our `MergeMapper` transforms Merge.dev format to BridgeBond format:

```javascript
import { MergeMapper } from './integrations/merge/index.js';

const { mappedData, additionalFields } = MergeMapper.mapEmployeeToBridgeBond(
  mergeEmployee,
  fieldMappings
);
```

---

## Webhooks

### Setup

1. Go to Merge.dev Dashboard → Webhooks
2. Add webhook URL: `https://your-domain.com/v1/merge/webhook`
3. Set webhook secret
4. Select events:
   - `LinkedAccount.linked`
   - `CommonModel.synced`
   - `ChangedData`
   - `DeletedData`

### Webhook Payload

```json
{
  "hook": {
    "id": "hook-id",
    "event": "CommonModel.synced",
    "target": "https://your-domain.com/v1/merge/webhook"
  },
  "linked_account": {
    "id": "account-id",
    "integration": "BambooHR",
    "integration_slug": "bamboohr",
    "category": "hris",
    "status": "COMPLETE",
    "account_token": "YOUR_ACCOUNT_TOKEN"
  },
  "data": {
    "model": "Employee",
    "id": "employee-id",
    "fields": { ... }
  }
}
```

### Signature Verification

```javascript
import crypto from 'crypto';

const signature = req.headers['x-merge-webhook-signature'];
const rawBody = JSON.stringify(webhookData);

// Calculate HMAC-SHA256
const hmac = crypto.createHmac('sha256', webhookSecret);
hmac.update(rawBody, 'utf8');
const digest = hmac.digest();

// Base64url encode
const b64Encoded = digest.toString('base64')
  .replace(/\+/g, '-')
  .replace(/\//g, '_')
  .replace(/=/g, '');

// Timing-safe comparison
const isValid = crypto.timingSafeEqual(
  Buffer.from(b64Encoded),
  Buffer.from(signature)
);
```

---

## Pagination

Merge.dev uses cursor-based pagination:

```javascript
// First page
let response = await merge.hris.employees.list({ pageSize: 100 });

// Next page
if (response.next) {
  response = await merge.hris.employees.list({
    cursor: response.next,
    pageSize: 100,
  });
}

// Previous page
if (response.previous) {
  response = await merge.hris.employees.list({
    cursor: response.previous,
    pageSize: 100,
  });
}
```

---

## Error Handling

### Common Errors

| Status | Error | Solution |
|--------|-------|----------|
| 401 | Unauthorized | Check API key and account token |
| 403 | Forbidden | Check account permissions |
| 404 | Not Found | Resource doesn't exist |
| 429 | Rate Limited | Wait and retry |
| 500+ | Server Error | Contact Merge.dev support |

### Error Handling in Code

```javascript
try {
  const employees = await merge.hris.employees.list();
} catch (error) {
  if (error.status === 401) {
    // Authentication failed
  } else if (error.status === 429) {
    // Rate limited - retry with backoff
  } else {
    // Other error
  }
}
```

---

## Best Practices

### 1. Use Official SDK

✅ **Use**: `@mergeapi/merge-node-client`  
❌ **Don't**: Build custom HTTP client

### 2. Handle Pagination

✅ **Do**: Use cursor-based pagination  
❌ **Don't**: Assume all data fits in one request

### 3. Verify Webhooks

✅ **Do**: Always verify webhook signatures  
❌ **Don't**: Trust webhooks without verification

### 4. Error Handling

✅ **Do**: Handle all error cases  
❌ **Don't**: Ignore error responses

### 5. Rate Limiting

✅ **Do**: Respect rate limits  
❌ **Don't**: Make excessive requests

---

## Testing

### Using Test Account Token

Merge.dev provides `TEST_ACCOUNT_TOKEN` for testing:

```javascript
const merge = new MergeClient({
  apiKey: 'YOUR_API_KEY',
  accountToken: 'TEST_ACCOUNT_TOKEN', // Returns dummy data
});
```

### Sandbox Environment

1. Connect to Merge.dev Mock Sandbox
2. Test with dummy data
3. Verify field mappings
4. Test webhooks

---

## Resources

- **Documentation**: https://docs.merge.dev/get-started/introduction/
- **API Reference**: https://docs.merge.dev/api/hris/overview
- **SDK GitHub**: https://github.com/merge-api/merge-node-client
- **Dashboard**: https://app.merge.dev
- **Support**: support@merge.dev

---

## Next Steps

1. ✅ Install SDK: `npm install --save @mergeapi/merge-node-client`
2. ✅ Set up Merge.dev account
3. ✅ Get API key and account tokens
4. ✅ Connect organizations via API
5. ✅ Configure field mappings
6. ✅ Set up webhooks
7. ✅ Test integration
8. ✅ Monitor and maintain

---

**Last Updated**: 2024  
**Status**: Production Ready

