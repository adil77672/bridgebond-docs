# Merge.dev SDK Update

## Overview

We've updated the Merge.dev integration to use the **official Merge.dev Node.js SDK** (`@mergeapi/merge-node-client`) instead of a custom axios implementation.

## Official SDK

**Package**: `@mergeapi/merge-node-client`  
**GitHub**: https://github.com/merge-api/merge-node-client  
**Documentation**: https://docs.merge.dev/api/hris/overview

## Installation

The SDK has been added to `package.json`. Install dependencies:

```bash
npm install
```

Or install directly:

```bash
npm install --save @mergeapi/merge-node-client
```

## Benefits of Using Official SDK

1. **Official Support**: Maintained by Merge.dev team
2. **Type Safety**: Full TypeScript support
3. **Better Error Handling**: Proper error types and messages
4. **Automatic Updates**: SDK updates with API changes
5. **Pagination Support**: Built-in pagination helpers
6. **Category Support**: Access to all Merge categories (HRIS, ATS, Accounting, CRM, Ticketing, File Storage)

## Usage

The SDK usage remains the same through our wrapper:

```javascript
import { MergeService } from './integrations/merge/index.js';

const credentials = {
  apiKey: process.env.MERGE_API_KEY,
  accountToken: 'org-account-token',
};

// All methods work the same
const employees = await MergeService.fetchEmployees(credentials);
```

## SDK Features

### Categories

The official SDK supports multiple categories:

```javascript
const merge = new MergeClient({
  apiKey: 'YOUR_API_KEY',
  accountToken: 'YOUR_ACCOUNT_TOKEN',
});

// HRIS (Human Resources)
merge.hris.employees.list()
merge.hris.employees.retrieve(id)

// ATS (Applicant Tracking System)
merge.ats.candidates.list()
merge.ats.candidates.retrieve(id)

// Accounting
merge.accounting.contacts.list()

// Ticketing
merge.ticketing.tickets.create()

// CRM
merge.crm.contacts.list()

// File Storage
merge.filestorage.files.list()
```

### Pagination

The SDK handles pagination automatically:

```javascript
let response = await merge.hris.employees.list({ pageSize: 100 });

// Get next page
while (response.next) {
  response = await merge.hris.employees.list({
    cursor: response.next,
    pageSize: 100,
  });
}
```

## Webhook Signature Verification

The webhook handler now uses the official Merge.dev signature verification method:

1. **HMAC-SHA256**: Calculate HMAC of the raw request body
2. **Base64url Encoding**: Encode the digest
3. **Timing-Safe Comparison**: Prevent timing attacks

Reference: https://docs.merge.dev/api/merge-api-basics/webhooks/merge-webhooks-sent-to-you

## Migration Notes

- ✅ **No breaking changes**: Our wrapper maintains the same interface
- ✅ **Backward compatible**: Existing code continues to work
- ✅ **Better error handling**: More descriptive error messages
- ✅ **Type safety**: Full TypeScript support available

## Next Steps

1. Run `npm install` to install the SDK
2. Test existing integrations
3. Consider using SDK directly for new features (bypassing wrapper if needed)

## Resources

- [Merge.dev SDK Documentation](https://docs.merge.dev/api/sdks)
- [Node.js SDK GitHub](https://github.com/merge-api/merge-node-client)
- [Merge.dev API Reference](https://docs.merge.dev/api/hris/overview)
- [Webhook Security Guide](https://docs.merge.dev/api/merge-api-basics/webhooks/merge-webhooks-sent-to-you)

