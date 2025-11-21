# Merge.dev Integration Setup Guide

## Overview

This guide explains how to set up and use the Merge.dev integration module for HRIS integrations. Merge.dev provides a unified API for connecting to 50+ HRIS platforms (ADP, BambooHR, Workday, etc.) through a single integration.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Installation](#installation)
3. [Configuration](#configuration)
4. [Usage](#usage)
5. [API Endpoints](#api-endpoints)
6. [Webhooks](#webhooks)
7. [Troubleshooting](#troubleshooting)

---

## Prerequisites

1. **Merge.dev Account**: Sign up at [merge.dev](https://www.merge.dev)
2. **API Key**: Get your Merge.dev API key from the dashboard
3. **Account Token**: Each organization needs an account token (generated in Merge dashboard)

---

## Installation

The Merge.dev integration module uses the **official Merge.dev Node.js SDK**.

### Install SDK

```bash
npm install --save @mergeapi/merge-node-client
```

**SDK Reference:**
- GitHub: https://github.com/merge-api/merge-node-client
- Documentation: https://docs.merge.dev/get-started/introduction/
- NPM: `@mergeapi/merge-node-client`

### Module Structure

```
src/integrations/merge/
├── mergeClient.js      # Low-level API client
├── mergeMapper.js      # Data transformation layer
├── mergeService.js     # High-level service
└── index.js            # Module exports
```

---

## Configuration

### 1. Environment Variables

Add these to your `.env` file:

```env
# Merge.dev Configuration
MERGE_API_KEY=your-merge-api-key-here
MERGE_WEBHOOK_SECRET=your-webhook-secret-here  # Optional, for webhook verification
```

**Get Your API Key:**
1. Sign up at [merge.dev](https://www.merge.dev)
2. Go to [API Keys](https://app.merge.dev/keys)
3. Copy your Production Access Key

### 2. Organization Setup

Each organization that wants to use Merge.dev needs:

1. **Account Token**: Generated in Merge.dev dashboard for each organization
   - Go to Merge.dev Dashboard → Linked Accounts
   - Create a new linked account for the organization
   - Copy the account token (found at bottom right of linked account page)

2. **Platform Connection**: Connect HRIS platforms in Merge.dev dashboard
   - For each organization, connect desired platforms (BambooHR, ADP, Workday, etc.)
   - Each platform connection is separate per organization
   - Example: Org A can connect BambooHR + ADP, Org B can connect Workday only

**Important:** Platform connections are managed in Merge.dev dashboard, not in Bridge Bond. Bridge Bond only connects to Merge.dev using the account token.

---

## Usage

### Connecting an Organization to Merge.dev

#### Option 1: Via API (Recommended)

```http
POST /v1/hr-integrations/organizations/{organizationId}/connect
Authorization: Bearer <your-token>
Content-Type: application/json

{
  "platform": "merge",
  "credentials": {
    "apiKey": "your-merge-api-key",
    "accountToken": "org-specific-account-token"
  }
}
```

#### Option 2: Using Merge Credentials in Existing Platform

You can also use Merge credentials with existing platform types:

```http
POST /v1/hr-integrations/organizations/{organizationId}/connect
Authorization: Bearer <your-token>
Content-Type: application/json

{
  "platform": "bamboohr",  // or "adp", "workday"
  "credentials": {
    "mergeApiKey": "your-merge-api-key",
    "mergeAccountToken": "org-specific-account-token"
  }
}
```

The system will automatically detect Merge credentials and use the Merge.dev integration.

### Syncing Employees

Once connected, sync employees:

```http
POST /v1/hr-integrations/integrations/{integrationId}/sync
Authorization: Bearer <your-token>
```

### Field Mapping

Map Merge.dev fields to BridgeBond fields:

```http
GET /v1/hr-integrations/integrations/{integrationId}/field-mappings
Authorization: Bearer <your-token>
```

```http
PATCH /v1/hr-integrations/integrations/{integrationId}/field-mappings
Authorization: Bearer <your-token>
Content-Type: application/json

{
  "fieldMappings": {
    "email": "email",
    "firstName": "first_name",
    "lastName": "last_name",
    "jobTitle": "employments.job_title",
    "department": "employments.department",
    "dateOfHire": "employments.start_date"
  }
}
```

---

## API Endpoints

### Merge.dev Integration Endpoints

All standard HR integration endpoints work with Merge.dev:

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/v1/hr-integrations/organizations/:orgId/connect` | POST | Connect to Merge.dev |
| `/v1/hr-integrations/integrations/:id/sync` | POST | Sync employees |
| `/v1/hr-integrations/integrations/:id/field-mappings` | GET/PATCH | Manage field mappings |
| `/v1/hr-integrations/integrations/:id` | GET/DELETE | Get/Delete integration |

### Merge.dev Webhook Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/v1/merge/webhook` | GET | Verify webhook endpoint |
| `/v1/merge/webhook` | POST | Receive webhook events |

---

## Webhooks

### Setting Up Webhooks in Merge.dev

1. Go to Merge.dev Dashboard → Advanced Configuration → Webhooks
2. Click **+Webhook**
3. Add webhook URL: `https://your-domain.com/v1/merge/webhook`
4. Set webhook secret (recommended, for signature verification)
5. Select events to subscribe to:
   - `LinkedAccount.linked` - New account connected
   - `CommonModel.synced` - Data synced (recommended)
   - `ChangedData` - Data changed (recommended)
   - `DeletedData` - Data deleted
   - `AsyncPassthrough.completed` - Async operation completed

**Reference:** https://docs.merge.dev/api/merge-api-basics/webhooks/merge-webhooks-sent-to-you

### Webhook Payload Format

According to Merge.dev documentation, webhook payloads have this structure:

```json
{
  "hook": {
    "id": "cb1fe0a7-c2a1-4bd6-8cf5-57e70c7f1d53",
    "event": "CommonModel.synced",
    "target": "https://your-domain.com/v1/merge/webhook"
  },
  "linked_account": {
    "id": "a3602c03-aba7-4d9d-a349-dbc338504092",
    "integration": "BambooHR",
    "integration_slug": "bamboohr",
    "category": "hris",
    "end_user_organization_name": "Acme Corp",
    "end_user_email_address": "admin@acme.com",
    "status": "COMPLETE",
    "account_token": "YOUR_ACCOUNT_TOKEN"
  },
  "data": {
    "model": "Employee",
    "id": "employee-123",
    "fields": {
      "first_name": "John",
      "last_name": "Doe",
      "email": "john.doe@example.com"
    }
  }
}
```

**Reference:** https://docs.merge.dev/api/merge-api-basics/webhooks/merge-webhooks-sent-to-you

### Webhook Security

If you set `MERGE_WEBHOOK_SECRET`, the webhook handler will verify the signature using HMAC-SHA256.

**Verification Process:**
1. Calculate HMAC-SHA256 of the raw request body using your webhook secret
2. Base64url encode the digest
3. Compare with `X-Merge-Webhook-Signature` header using timing-safe comparison

**Setup:**
1. Get webhook signature key from Merge.dev Dashboard → Webhooks → Security
2. Set `MERGE_WEBHOOK_SECRET` environment variable
3. The same key must be set in Merge.dev dashboard

**Reference:** https://docs.merge.dev/api/merge-api-basics/webhooks/merge-webhooks-sent-to-you#security

---

## Code Examples

### Using MergeService (Recommended)

```javascript
import { MergeService } from './integrations/merge/index.js';

// Test connection
const credentials = {
  apiKey: process.env.MERGE_API_KEY,
  accountToken: 'org-account-token',
};

const isConnected = await MergeService.testConnection(credentials);

// Fetch employees
const employees = await MergeService.fetchEmployees(credentials);

// Fetch available fields
const fields = await MergeService.fetchAvailableFields(credentials);
```

### Using Official SDK Directly

```javascript
import { MergeClient } from '@mergeapi/merge-node-client';

const merge = new MergeClient({
  apiKey: process.env.MERGE_API_KEY,
  accountToken: 'org-account-token',
});

// Fetch employees with pagination
let response = await merge.hris.employees.list({ pageSize: 100 });
console.log(response.results);

// Get next page
if (response.next) {
  response = await merge.hris.employees.list({
    cursor: response.next,
    pageSize: 100,
  });
}

// Fetch single employee
const employee = await merge.hris.employees.retrieve('employee-id', {
  expand: 'employments,home_location,work_location',
});
```

### Data Mapping

```javascript
import { MergeMapper } from './integrations/merge/index.js';

const mergeEmployee = {
  id: 'emp-123',
  first_name: 'John',
  last_name: 'Doe',
  email: 'john@example.com',
  employments: [{
    job_title: 'Engineer',
    department: 'Engineering',
    start_date: '2024-01-15',
  }],
};

const fieldMappings = new Map([
  ['email', 'email'],
  ['firstName', 'first_name'],
  ['jobTitle', 'employments.job_title'],
]);

const { mappedData, additionalFields } = MergeMapper.mapEmployeeToBridgeBond(
  mergeEmployee,
  fieldMappings
);

console.log(mappedData);
// {
//   email: 'john@example.com',
//   firstName: 'John',
//   lastName: 'Doe',
//   jobTitle: 'Engineer',
//   department: 'Engineering',
//   dateOfHire: '2024-01-15',
//   ...
// }
```

---

## Supported Platforms via Merge.dev

Merge.dev supports 50+ HRIS platforms. Some popular ones:

- ✅ ADP (Workforce Now, Vantage)
- ✅ BambooHR
- ✅ Workday
- ✅ Gusto
- ✅ Rippling
- ✅ Paylocity
- ✅ Namely
- ✅ Justworks
- ✅ And 40+ more...

See [Merge.dev Integrations](https://docs.merge.dev/integrations/hris/overview) for the complete list.

---

## Troubleshooting

### Connection Issues

**Problem**: `Merge API authentication failed`

**Solution**:
1. Verify `MERGE_API_KEY` is set correctly
2. Check that `accountToken` is valid for the organization
3. Ensure the account token is active in Merge.dev dashboard

### Sync Issues

**Problem**: Employees not syncing

**Solution**:
1. Check integration connection status
2. Verify field mappings are correct
3. Check Merge.dev logs for API errors
4. Ensure the HRIS platform is connected in Merge.dev dashboard

### Webhook Issues

**Problem**: Webhooks not received

**Solution**:
1. Verify webhook URL is accessible (not behind firewall)
2. Check webhook secret matches in both systems
3. Review Merge.dev webhook logs
4. Ensure webhook endpoint is returning 200 status

### Field Mapping Issues

**Problem**: Fields not mapping correctly

**Solution**:
1. Use `GET /integrations/:id/platform-fields` to see available fields
2. Check Merge.dev field names (they're normalized)
3. Use dot notation for nested fields (e.g., `employments.job_title`)

---

## Migration from Direct Integrations

If you're migrating from direct platform integrations (ADP, BambooHR, Workday) to Merge.dev:

### Why Migrate?

- ✅ **One integration** instead of multiple
- ✅ **Easier maintenance** - Merge.dev handles API changes
- ✅ **More platforms** - Access to 50+ platforms
- ✅ **Better reliability** - Built-in retries and error handling
- ✅ **Unified data model** - Consistent format across platforms

### Migration Steps

1. **Set up Merge.dev account** and get API key
2. **Keep existing integrations running** during migration
3. **Create Merge.dev integration** for one organization (test)
4. **Connect platforms in Merge.dev** dashboard for that organization
5. **Run parallel syncs** to compare data
6. **Validate results** match between direct and Merge.dev integrations
7. **Switch over** once validated
8. **Repeat for other organizations**
9. **Remove old direct integrations** after successful migration

### Migration Checklist

- [ ] Set up Merge.dev account
- [ ] Connect HRIS platforms in Merge.dev dashboard
- [ ] Get account tokens for each organization
- [ ] Create Merge.dev integrations via API
- [ ] Configure field mappings
- [ ] Run test sync
- [ ] Compare results with existing integration
- [ ] Update frontend to use Merge.dev
- [ ] Monitor for 1-2 weeks
- [ ] Remove old direct integrations

---

## Best Practices

1. **Use Account Tokens**: Each organization should have its own account token
2. **Field Mapping**: Always configure field mappings for accurate data sync
3. **Error Handling**: Monitor sync logs and handle errors gracefully
4. **Rate Limiting**: Merge.dev handles rate limiting, but be aware of limits
5. **Webhooks**: Use webhooks for real-time updates instead of polling
6. **Testing**: Test in sandbox environment before production

---

## Support

- **Merge.dev Documentation**: [docs.merge.dev](https://docs.merge.dev)
- **Merge.dev Support**: [support@merge.dev](mailto:support@merge.dev)
- **Bridge Bond Issues**: Create an issue in the repository

---

## Next Steps

1. Review the [Merge.dev Evaluation Document](./MERGE_DEV_EVALUATION.md)
2. Set up Merge.dev account and get API key
3. Test connection with one organization
4. Configure field mappings
5. Run initial sync
6. Set up webhooks for real-time updates

---

**Last Updated**: 2024
**Status**: Ready for Testing

