---
name: fireberry-app-builder
description: Use when building Fireberry platform apps, components, or plugins - scaffolding with CLI, using @fireberry/sdk for iframe communication, @fireberry/ds design system, component types (record/side-menu/global-menu), local debugging, deployment, or integrating fireberry-api-client in SDK mode
---

# Fireberry App Builder

Build and deploy custom apps that run inside the Fireberry CRM platform as iframe-based components.

## Architecture

Apps are **client-side only** (React + Vite), hosted on Fireberry servers, rendered in iframes. Communication with the platform happens via `@fireberry/sdk` messaging. The `fireberry-api-client` library wraps the SDK for easier data operations.

## Quick Reference

| Command | Purpose |
|---------|---------|
| `npm i -g @fireberry/cli` | Install CLI |
| `fireberry init [token]` | Authenticate |
| `fireberry create <name>` | Scaffold app |
| `fireberry create-component [name] [type]` | Add component |
| `fireberry debug <id> localhost:<port>` | Live debug |
| `fireberry debug <id> --stop` | Stop debug |
| `fireberry push` | Deploy to servers |
| `fireberry install` | Install on account |
| `fireberry delete` | Remove app permanently |

## Component Types

| Type | Location | Record Context | Use Case |
|------|----------|----------------|----------|
| `record` | Record toolbar | `record.id`, `record.type` | Record-specific views, custom forms |
| `side-menu` | Side nav panel | No | Quick tools, utilities, search |
| `global-menu` | Top nav bar | No | Dashboards, admin panels, reports |

All types get `user.id`, `user.fullName`, `user.organizationId`.

## Scaffolding & Project Structure

```bash
fireberry create my-app
# Prompts: component name, type, settings
```

```
my-app/
├── manifest.yml          # App & component config
├── component-1/
│   ├── src/
│   ├── dist/             # Build output
│   └── package.json
└── component-2/
    └── ...
```

### manifest.yml

```yaml
app:
  id: "auto-generated-uuid"
  name: "my-app"
  description: "App description"

components:
  - type: record
    title: "My Component"
    id: "auto-generated-uuid"
    path: component-name/dist
    settings:
      iconName: "icon-name"
      iconColor: "#3498db"
      objectType: 1        # Which record type (1=Account, 2=Contact...)
      height: "M"          # S, M, L

  - type: side-menu
    title: "Side Panel"
    id: "uuid"
    path: side-panel/dist
    settings:
      icon: "lightning"
      width: "S"

  - type: global-menu
    title: "Dashboard"
    id: "uuid"
    path: dashboard/dist
    settings:
      displayName: "Dashboard"
```

## SDK Setup (`@fireberry/sdk`)

```typescript
import FireberryClientSDK from "@fireberry/sdk/client";

const client = new FireberryClientSDK();
await client.initializeContext();

// Access context
console.log(client.context.user.id);
console.log(client.context.user.fullName);
console.log(client.context.record.id);   // Record components only
console.log(client.context.record.type); // Record components only
```

### TypeScript Generics

```typescript
const client = new FireberryClientSDK<MyResponseType, MySettingsType>();
```

### SDK CRUD Operations

```typescript
// Query
const results = await client.api.query(objectType, {
  fields: "accountid,accountname,statuscode",
  query: '(statuscode = 1)',
  page_size: 20,
  page_number: 1,
});

// Create
const result = await client.api.create(objectType, { accountname: "New" });

// Update
const result = await client.api.update(objectType, recordId, { accountname: "Updated" });

// Delete
const result = await client.api.delete(objectType, recordId);
```

### Response Format

```typescript
interface ResponseData {
  success: boolean;
  data: any;
  error?: { status: number; statusText: string; data: { Message?: string } };
  requestId: string;
}
```

### Settings API (Cross-Component Storage)

```typescript
// Read settings (organization-scoped, shared across app components)
const settings = await client.app.settings.get();

// Write settings - REPLACES entire value, does NOT merge!
const existing = await client.app.settings.get();
await client.app.settings.set({ ...existing, newKey: "value" });
```

### System Methods

```typescript
// Badge notification
await client.system.badge.show({ number: 5, badgeType: "info" }); // success|warning|error|info
await client.system.badge.hide();

// Callbar (telephony integration)
await client.system.callbar.show({
  callInfo: { number: 1234567890, status: "Talking" }, // Talking|Ringing|Missed|Dialing
  objectConfig: [{ objectType: "1", order: 1, fields: [{ name: "accountname" }] }],
  placemment: "bottom-end", // bottom-start|bottom-end (note: typo is in the API)
});
await client.system.callbar.hide();
```

### Cleanup

```typescript
// Call before component unmount
client.destroy();
```

## Using fireberry-api-client with SDK (Recommended)

Wraps the SDK with QueryBuilder, caching, batch operations, and more. See the `fireberry` skill for full API client reference.

```typescript
import FireberryClientSDK from "@fireberry/sdk/client";
import { FireberryClient } from "fireberry-api-client";

const sdk = new FireberryClientSDK();
await sdk.initializeContext();

// SDK-only mode (CRUD only, no metadata)
const client = new FireberryClient({ sdk });

// Hybrid mode (recommended - SDK for CRUD, API for metadata)
const client = new FireberryClient({ sdk, apiKey: "your-key" });

// Use fluent QueryBuilder
const accounts = await client
  .queryBuilder()
  .objectType(1)
  .select("accountid", "accountname")
  .where("statuscode").equals("1")
  .execute();

// CRUD, batch, streaming - all work through SDK transparently
await client.records.create(1, { accountname: "New Account" });
```

### SDK-Specific Utilities (`fireberry-api-client/sdk`)

```typescript
import { EnhancedSDK, createSDKQueryBuilder } from "fireberry-api-client/sdk";

// Enhanced SDK wrapper
const enhanced = EnhancedSDK.create(sdk);
console.log(enhanced.userId, enhanced.userFullName);
console.log(enhanced.recordId, enhanced.recordType);

// Field utilities
enhanced.getIdField(1);    // "accountid"
enhanced.getNameField(2);  // "fullname"
enhanced.getLabelField("statuscode", 1); // "status"
enhanced.expandFieldsWithLabels(["statuscode", "ownerid"], 1);
// ["statuscode", "status", "ownerid", "ownername"]

// Query factory
const qb = createSDKQueryBuilder(sdk);
const results = await qb(1)
  .select("accountid", "accountname")
  .selectWithLabels("ownerid")
  .where("statuscode").equals("1")
  .execute();
```

## Design System (`@fireberry/ds`)

Storybook: https://design-system.fireberry.com/

```tsx
import { DSThemeContextProvider, Button, Typography, Icon, IconName } from "@fireberry/ds";

function App() {
  return (
    <DSThemeContextProvider isRtl={false}>
      <Typography type="h2">My Component</Typography>
      <Button variant="primary" color="success" onClick={handleClick}>
        Save
      </Button>
      <Icon icon={IconName.Settings} size="20px" />
    </DSThemeContextProvider>
  );
}
```

| Category | Components |
|----------|-----------|
| Buttons | `Button`, `IconButton`, `MultiButton` |
| Form | `Checkbox`, `Toggle`, `RadioButton` |
| Typography | `Typography` (h1-h3, title, subTitle, text, largeText, caption) |
| Icons | `Icon` + `IconName` enum (260+ icons) |
| Display | `Avatar`, `Breadcrumb`, `List`, `ListItem` |
| Layout | `Collapse`, `Stepper` |

**Colors:** `success` (green), `destructive` (red), `warning` (orange), `neutral` (gray), `information` (blue)

**Button variants:** `primary` (filled), `secondary` (outlined+hover), `text` (text only), `outlined` (white+border)

**Theme access:**
```tsx
const { theme, isRtl, palette } = useDSThemeContext();
```

## Local Development

```bash
# Terminal 1: Dev server
cd my-component && npm run dev

# Terminal 2: Connect to Fireberry
fireberry debug <component-id-from-manifest> localhost:5173
```

Changes appear instantly via HMR. Stop debug mode before deploying:
```bash
fireberry debug <component-id> --stop
npm run build
fireberry push
```

## Deployment

```bash
# Build all components
cd component-1 && npm run build && cd ..
cd component-2 && npm run build && cd ..

# Deploy and install
fireberry push      # Upload to Fireberry servers
fireberry install   # First time only - makes app available

# Updates: just push (no re-install needed)
fireberry push
```

## Common Patterns

### Record Component with Context

```tsx
import FireberryClientSDK from "@fireberry/sdk/client";
import { FireberryClient } from "fireberry-api-client";
import { DSThemeContextProvider, Typography } from "@fireberry/ds";
import { useEffect, useState } from "react";

function RecordComponent() {
  const [record, setRecord] = useState<Record<string, unknown> | null>(null);

  useEffect(() => {
    const init = async () => {
      const sdk = new FireberryClientSDK();
      await sdk.initializeContext();
      const client = new FireberryClient({ sdk });

      const { recordId, recordType } = {
        recordId: sdk.context.record.id,
        recordType: sdk.context.record.type,
      };

      const result = await client
        .queryBuilder()
        .objectType(recordType)
        .whereId(recordId)
        .first();

      setRecord(result);
    };
    init();
  }, []);

  return (
    <DSThemeContextProvider isRtl={false}>
      <Typography type="title">
        {record ? String(record.accountname || record.fullname || record.name) : "Loading..."}
      </Typography>
    </DSThemeContextProvider>
  );
}
```

### Settings Persistence Pattern

```typescript
// Read-modify-write (settings.set replaces, doesn't merge!)
const current = await sdk.app.settings.get();
await sdk.app.settings.set({
  ...current,
  lastUpdated: new Date().toISOString(),
  config: { ...current.config, newField: "value" },
});
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Using `settings.set()` without spreading existing | Always read first, spread existing values |
| Accessing `record.id` in side-menu/global-menu | Record context only exists in `record` type components |
| Forgetting `client.destroy()` on unmount | Add cleanup in useEffect return or component unmount |
| Not building before `fireberry push` | Run `npm run build` in each component directory first |
| Debug mode still active when pushing | Run `fireberry debug <id> --stop` before deploying |
| SDK timeout (60s default) not handled | Wrap in try/catch, check `result.success` |
| Using raw SDK for complex queries | Use `fireberry-api-client` wrapper - handles pagination, caching, retries |
