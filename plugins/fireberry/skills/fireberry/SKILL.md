---
name: fireberry
description: Use when integrating with Fireberry CRM API - querying records, CRUD operations, batch processing, metadata introspection, field creation, schema generation, or building apps with fireberry-api-client
---

# Fireberry API Client Integration Skill

Expert guidance for using the fireberry-api-client TypeScript library in Node.js and browser environments.

## Quick Start

### Installation
```bash
npm install fireberry-api-client
```

For SDK mode (browser only):
```bash
npm install @fireberry/sdk fireberry-api-client
```

### Setup Patterns

**API Mode (Node.js - Recommended)**
```typescript
import { FireberryClient } from 'fireberry-api-client';

const client = new FireberryClient({
  apiKey: process.env.FIREBERRY_TOKEN, // or FIREBERRY_API_KEY
});
```

**SDK Mode (Browser Only)**
```typescript
import { FireberryClientSDK } from '@fireberry/sdk';
import { FireberryClient } from 'fireberry-api-client';

const sdk = new FireberryClientSDK();
await sdk.init();

const client = new FireberryClient({ sdk });
```

**Hybrid Mode (SDK + API)**
```typescript
const client = new FireberryClient({
  sdk, // CRUD operations via SDK
  apiKey: process.env.FIREBERRY_TOKEN, // Enables metadata
});
```

## Core Operations

### CRUD Pattern

```typescript
// Create
const account = await client.records.create('1', {
  accountname: 'Acme Corp',
  email: 'contact@acme.com',
});

// Read
const account = await client.records.get('1', accountId);

// Update
const updated = await client.records.update('1', accountId, {
  email: 'new@acme.com',
});

// Delete
await client.records.delete('1', accountId);

// Upsert
const result = await client.records.upsert('1', data, {
  matchField: 'email',
});
```

### Query Pattern

**Use QueryBuilder for all queries - it handles escaping and prevents injection:**

```typescript
// QueryBuilder (fluent API)
const accounts = await client.queryBuilder()
  .objectType('1')
  .select('accountname', 'email', 'revenue')
  .where('statuscode').equals('1')
  .and()
  .where('revenue').greaterThan(100000)
  .sortBy('revenue', 'desc')
  .limit(100)
  .execute();

// Query by ID (auto-maps field name per object type)
const account = await client.queryBuilder()
  .objectType(1)
  .whereId('abc123')
  .first(); // Returns single record or null

// Date helpers
const recent = await client.queryBuilder()
  .objectType(1)
  .whereDate('createdon').daysAgo(7)
  .execute();

// Auto-pagination (default)
const result = await client.query({
  objectType: '1',
  fields: '*',
  query: '(statuscode = 1)',
});
```

### Batch Operations Pattern

```typescript
// Batch create (auto-chunks to 20 records per batch)
const accounts = Array.from({ length: 100 }, (_, i) => ({
  accountname: `Account ${i}`,
}));

const result = await client.batch.create('1', accounts, {
  onProgress: (current, total) => {
    console.log(`${current}/${total}`);
  },
});

// Batch update
const updates = [
  { id: 'id-1', record: { statuscode: '2' } },
  { id: 'id-2', record: { statuscode: '2' } },
];
await client.batch.update('1', updates);

// Batch delete
await client.batch.delete('1', ['id-1', 'id-2', 'id-3']);
```

## Metadata Operations

**⚠️ Metadata requires API key (not available in SDK-only mode)**

```typescript
// Get all objects
const { objects } = await client.metadata.getObjects();

// Get fields for object type
const { fields } = await client.metadata.getFields('1', {
  includeLookupRelations: true, // default: true
});

// Get dropdown values
const { values } = await client.metadata.getFieldValues('1', 'statuscode');
```

## Schema & ERD Generation

**Generate TypeScript interfaces from Fireberry metadata:**

```typescript
import { schemaBuilder, erdBuilder } from 'fireberry-api-client/utils';

// Generate TypeScript schema
const schema = await schemaBuilder(client)
  .include([1, 2, 3]) // Account, Contact, Lead
  .withComments(true)
  .withFieldTypes(true)
  .withLookupInfo(true)
  .generate();

// Save to file
import { writeFileSync } from 'fs';
writeFileSync('fireberry-schema.ts', schema.typescript);

// Generate ERD diagram (Mermaid)
const erd = await erdBuilder(client)
  .include([1, 2, 3])
  .settings({
    includeFields: true,
    showFieldTypes: true,
    maxFieldsPerEntity: 15,
  })
  .generate();

writeFileSync('fireberry-erd.md', erd.mermaid);
```

## Error Handling Pattern

```typescript
import { FireberryError, FireberryErrorCode } from 'fireberry-api-client';

try {
  const result = await client.records.get('1', id);
} catch (error) {
  if (error instanceof FireberryError) {
    switch (error.code) {
      case FireberryErrorCode.RATE_LIMITED:
        // Auto-retry is enabled by default
        break;
      case FireberryErrorCode.NOT_FOUND:
        return null; // Handle missing record
      case FireberryErrorCode.AUTHENTICATION_FAILED:
        throw new Error('Invalid API key');
      case FireberryErrorCode.NETWORK_ERROR:
        // Network or timeout error
        break;
    }
  }
}
```

## Common Object Types

| ID | Object | ID Field | Name Field |
|----|--------|----------|------------|
| 1 | Account | accountid | accountname |
| 2 | Contact | contactid | fullname |
| 3 | Lead | leadid | fullname |
| 4 | Opportunity | opportunityid | name |
| 5 | Task | taskid | subject |
| 6 | Event | eventid | subject |
| 7 | Note | noteid | subject |
| 14 | Product | productid | productname |

## Field Types

### System Field Type IDs

Each field has a `systemFieldTypeId` (UUID). Use `FIELD_TYPE_IDS` constants for comparison and `FIELD_TYPE_MAPPINGS` for display names. The metadata API auto-maps these to `field.fieldType`.

| `FIELD_TYPE_IDS.*` | Display Name | Notes |
|--------------------|-------------|-------|
| `TEXT` | Text | maxLength 1-200 |
| `LONG_TEXT` | Long Text | Textarea |
| `NUMERIC` | Number | precision 0-4 |
| `DROPDOWN` | Dropdown | Has picklist values |
| `LOOKUP` | Lookup | relatedObjectType populated with `includeLookupRelations: true` |
| `EMAIL` | Email | |
| `URL` | URL | |
| `TELEPHONE` | Telephone | |
| `DATE` | Date | |
| `DATETIME` | DateTime | |
| `HTML` | HTML | Rich text |

```typescript
import { FIELD_TYPE_IDS, FIELD_TYPE_MAPPINGS } from 'fireberry-api-client';

// Check field type
if (field.systemFieldTypeId === FIELD_TYPE_IDS.LOOKUP) { /* lookup logic */ }

// Get display name from UUID
const typeName = FIELD_TYPE_MAPPINGS[field.systemFieldTypeId]; // "Dropdown"

// metadata.getFields() auto-maps fieldType for you
const { fields } = await client.metadata.getFields('1');
fields.forEach(f => console.log(f.fieldName, f.fieldType)); // "statuscode", "Dropdown"
```

### Field Metadata Properties

`client.metadata.getFields()` returns `FireberryField` objects with: `fieldName`, `label`, `systemFieldTypeId`, `fieldType` (auto-mapped display name), `required?`, `defaultValue?`, `maxLength?` (text), `precision?` (number), `relatedObjectType?` (lookup, when `includeLookupRelations: true`).

### Creating Custom Fields

Use `client.fields.create(objectType, options)`. Field names must start with `pcf_`.

**All types require:** `type`, `fieldName` (must start with `pcf_`), `label`
**Optional for all:** `defaultValue?`, `follow?` (track changes), `autoComplete?` (text/email/url/phone/number only)

| Type | Extra Options |
|------|--------------|
| `text`, `email`, `url` | `maxLength?: number` (1-200) |
| `number` | `precision?: number` (0-4 decimal places) |
| `picklist` | `values: { name: string, value: string }[]` |
| `lookup` | `relatedObjectId: string` |
| `formula` | `formula: string`, `formulaFieldType: 'text'\|'number'\|'date'\|'boolean'`, `formulaPrecision?: number` |
| `summary` | `summaryType: 'avg'\|'count'\|'max'\|'min'\|'sum'`, `relatedObjectId: string`, `summaryField?: string` |
| `textarea`, `phone`, `date`, `datetime`, `html` | No extra options |

```typescript
// Picklist with values
await client.fields.create('1', {
  type: 'picklist',
  fieldName: 'pcf_priority',
  label: 'Priority',
  values: [
    { name: 'Low', value: '1' },
    { name: 'Medium', value: '2' },
    { name: 'High', value: '3' },
  ],
});

// Lookup (relation to another object)
await client.fields.create('2', {
  type: 'lookup',
  fieldName: 'pcf_related_account',
  label: 'Related Account',
  relatedObjectId: '1', // Points to Account
});

// Formula
await client.fields.create('1', {
  type: 'formula',
  fieldName: 'pcf_full_address',
  label: 'Full Address',
  formula: 'CONCAT(city, ", ", country)',
  formulaFieldType: 'text',
});

// Summary (rollup)
await client.fields.create('1', {
  type: 'summary',
  fieldName: 'pcf_total_value',
  label: 'Total Order Value',
  summaryType: 'sum',
  relatedObjectId: '13',
  summaryField: 'totalamount', // Required for sum/avg/min/max
});
```

## Field Mapping Utilities

```typescript
import {
  mapFieldToLabel,
  getObjectIdField,
  getObjectNameField,
} from 'fireberry-api-client/utils';

// Get ID and name fields for object type
const idField = getObjectIdField(1); // 'accountid'
const nameField = getObjectNameField(1); // 'accountname'

// Map ID fields to name fields
const nameField = mapFieldToLabel('accountid', 1); // 'accountname'
const statusField = mapFieldToLabel('statuscode', 1); // 'status'
const lookupName = mapFieldToLabel('primarycontactid', 1); // 'primarycontactname'
```

## Best Practices

### 1. Always Use QueryBuilder for User Input

```typescript
// ✅ GOOD - QueryBuilder escapes values
const result = await client.queryBuilder()
  .objectType('1')
  .where('accountname').contains(userInput)
  .execute();

// ❌ BAD - Manual query string (injection risk)
const query = `(accountname start-with %${userInput})`;
```

### 2. Use Batch Operations for Multiple Records

```typescript
// ✅ GOOD - Single batch operation
await client.batch.create('1', manyRecords);

// ❌ BAD - Loop with individual creates
for (const record of manyRecords) {
  await client.records.create('1', record);
}
```

### 3. Enable Auto-Pagination with Limits

```typescript
// ✅ GOOD - Auto-pagination with safety limit
const result = await client.query({
  objectType: '1',
  fields: '*',
  autoPage: true,
  limit: 5000, // Prevent runaway queries
});
```

### 4. Use Type-Safe Queries with Generated Types

```typescript
// Generate types first
const schema = await schemaBuilder(client).include([1]).generate();
// Save to fireberry-schema.ts

// Then use in queries
import type { Account } from './fireberry-schema';

const accounts = await client.query<Account>({
  objectType: '1',
  fields: '*',
});

accounts.records.forEach((account: Account) => {
  console.log(account.accountname); // Type-safe
});
```

### 5. Use Hybrid Mode for Browser Apps

```typescript
// ✅ GOOD - Hybrid mode for best performance
const client = new FireberryClient({
  sdk, // Fast iframe messaging for CRUD
  apiKey: process.env.FIREBERRY_TOKEN, // Enables metadata
});
```

### 6. Handle AbortSignal for Long Operations

```typescript
const controller = new AbortController();
setTimeout(() => controller.abort(), 30000);

const result = await client.query({
  objectType: '1',
  fields: '*',
  signal: controller.signal,
});
```

### 7. Cache Configuration

```typescript
// Default: cache enabled with 5-minute TTL
const client = new FireberryClient({
  apiKey: process.env.FIREBERRY_TOKEN,
  enableCache: true, // default
  cacheTTL: 300000, // 5 minutes
});

// Cache auto-invalidates on mutations
await client.records.create('1', data); // Invalidates Account cache
```

## Connection Modes Comparison

| Feature | API Mode | SDK Mode | Hybrid Mode |
|---------|----------|----------|-------------|
| CRUD Operations | ✅ HTTP | ✅ SDK | ✅ SDK |
| Queries | ✅ HTTP | ✅ SDK | ✅ SDK |
| Batch Operations | ✅ HTTP | ✅ SDK* | ✅ SDK* |
| Metadata | ✅ HTTP | ❌ | ✅ HTTP |
| File Upload | ✅ HTTP | ❌ | ✅ HTTP |
| Schema Generator | ✅ HTTP | ❌ | ✅ HTTP |
| ERD Generator | ✅ HTTP | ❌ | ✅ HTTP |

*SDK batch operations use individual calls (no native SDK batch support)

## When I'm Helping You

### If you're setting up a new project:
1. Ask about environment (Node.js vs Browser)
2. Recommend API mode for Node.js, Hybrid for browser
3. Show proper initialization with error handling
4. Set up TypeScript types if requested

### If you're querying data:
1. Always use QueryBuilder for safety
2. Enable auto-pagination by default with reasonable limits
3. Use proper field selection (avoid `*` in production)
4. Show type-safe query patterns

### If you're doing batch operations:
1. Use client.batch.* methods, not loops
2. Show progress tracking for large batches
3. Remind about 20-record chunk size
4. Handle errors per batch

### If you need metadata:
1. Check if API key is available (metadata requires API)
2. Use includeLookupRelations: false for faster responses
3. Cache results when possible
4. Generate TypeScript types for long-term use

### If there's an error:
1. Check error.code for FireberryErrorCode
2. Explain what each error code means
3. Show proper error recovery patterns
4. Suggest configuration changes if needed

## Integration Checklist

When integrating fireberry-api-client, verify:

- [ ] API key stored in environment variable (FIREBERRY_TOKEN)
- [ ] Proper connection mode chosen (API/SDK/Hybrid)
- [ ] Error handling with FireberryError
- [ ] QueryBuilder used for all user input
- [ ] Auto-pagination enabled with limits
- [ ] Batch operations for multiple records
- [ ] Cache configuration appropriate for use case
- [ ] TypeScript types generated (optional but recommended)
- [ ] AbortSignal support for long operations
- [ ] Rate limit handling configured (retryOn429: true)

## Additional Resources

For complete API documentation, refer to:
- `/skill.md` - Comprehensive usage guide
- `/README.md` - Package overview and examples
- `/CLAUDE.md` - Project-specific guidance
