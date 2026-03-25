# Monday.com GraphQL API Reference

## Authentication & Setup

Token stored in `~/.project-management/.env` as `MONDAY_API_TOKEN`.

First use — prompt user for token, then create directory and `.env` file:

```bash
mkdir -p ~/.project-management && echo "MONDAY_API_TOKEN=<token>" > ~/.project-management/.env
```

Load the token into the shell environment:

```bash
source ~/.project-management/.env
```

**API endpoint:** `POST https://api.monday.com/v2`

**Required headers:**
- `Authorization: {token}`
- `Content-Type: application/json`
- `API-Version: 2024-10`

**Curl template:**

```bash
curl -s -X POST https://api.monday.com/v2 \
  -H "Authorization: $MONDAY_API_TOKEN" \
  -H "Content-Type: application/json" \
  -H "API-Version: 2024-10" \
  -d '{"query": "..."}'
```

---

## Board IDs

- **Tasks board:** `9116155603`
- **Projects board:** `7109422573`

---

## Column Discovery

```graphql
query {
  boards(ids: [9116155603]) {
    columns { id title type settings_str }
    groups { id title }
  }
}
```

Run on first use and cache column IDs. For subitems, fetch the subitems board ID from the first created subitem's `board { id }` response.

---

## Create Item

```graphql
mutation {
  create_item(
    board_id: 9116155603
    item_name: "בניית תשתית מערכת"
    column_values: "{\"person\":{\"personsAndTeams\":[{\"id\":USER_ID,\"kind\":\"person\"}]},\"status\":{\"label\":\"Need To Start\"},\"date4\":{\"date\":\"2026-04-01\"},\"connect_boards\":{\"item_ids\":[PROJECT_ITEM_ID]}}"
  ) {
    id name
    column_values { id value text }
  }
}
```

---

## Create Subitem

```graphql
mutation {
  create_subitem(
    parent_item_id: PARENT_ID
    item_name: "בניית אובייקט לקוחות"
    column_values: "{\"person\":{\"personsAndTeams\":[{\"id\":USER_ID,\"kind\":\"person\"}]},\"status\":{\"label\":\"Need To Start\"}}"
  ) {
    id name
    board { id }
    column_values { id value text }
  }
}
```

**Note:** Subitem column IDs differ from parent board columns — they belong to an auto-generated subitems board.

---

## Create Update with Checklist

```graphql
mutation {
  create_update(
    item_id: ITEM_ID
    body: "<p><b>שדות נדרשים:</b></p><ul class='checklist'><li class='checklist_task' dir='rtl'>שם לקוח</li><li class='checklist_task' dir='rtl'>מספר טלפון</li></ul><p><b>מקרי קצה:</b></p><ul class='checklist'><li class='checklist_task' dir='rtl'>לקוח ללא מספר ת.ז.</li></ul>"
  ) {
    id body
  }
}
```

**Notes:**
- Use `dir='rtl'` for Hebrew checklist items
- The checklist class names (`checklist`, `checklist_task`) are unofficial but functional
- HTML body supports: `<p>`, `<b>`, `<strong>`, `<i>`, `<em>`, `<br>`, `<ul>/<li>`, `<ol>/<li>`

---

## Change Multiple Column Values

```graphql
mutation {
  change_multiple_column_values(
    item_id: ITEM_ID
    board_id: 9116155603
    column_values: "{...}"
    create_labels_if_missing: true
  ) {
    id
    column_values { id value text }
  }
}
```

**Note:** `column_values` is a JSON-encoded string. Required for board_relation columns.

---

## Query Items

```graphql
query {
  items_page_by_column_values(
    board_id: 9116155603
    limit: 50
    columns: [
      { column_id: "status", column_values: ["Need To Start", "Working on it"] }
    ]
  ) {
    cursor
    items {
      id name
      column_values { id value text }
      subitems { id name column_values { id value text } }
    }
  }
}
```

**Notes:**
- Multiple `columns` entries = AND logic
- Multiple values within one `column_values` = OR logic
- When `cursor` is null, no more pages

---

## Search Projects

```graphql
query {
  items_page_by_column_values(
    board_id: 7109422573
    limit: 10
    columns: [
      { column_id: "name", column_values: ["PROJECT_NAME"] }
    ]
  ) {
    items { id name }
  }
}
```

---

## Get Users

```graphql
query {
  users(kind: non_guests) {
    id name email
  }
}
```

---

## Column Value Formats (Quick Reference)

| Column Type | Format | Example / Notes |
|-------------|--------|-----------------|
| People | `{"personsAndTeams": [{"id": N, "kind": "person"}]}` | Replaces all assigned people |
| Status | `{"label": "Done"}` or `{"index": 1}` | Prefer label for readability |
| Date | `{"date": "2026-04-15"}` | ISO format |
| Board Relation | `{"item_ids": [12345]}` | Must use change_multiple_column_values |
| Numbers | `"42.5"` | String value. null to clear (not "") |
| Text | `"free text"` | Empty string to clear |

---

## Error Handling

- **Rate limits:** monday.com uses complexity-based rate limiting. If 429 received, wait and retry.
- Always check response for `errors` array before processing `data`.
- **Common errors:** invalid column ID, item not found, insufficient permissions.
