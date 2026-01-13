# Collection Management

Create and manage product collections - both manual and smart (rule-based).

## Quick Reference

| Task | Method |
|------|--------|
| List collections | `mcp__shopify__listCollections` |
| Create smart collection | `collectionCreate` with rules |
| Create manual collection | `collectionCreate` + add products |
| Add products | `collectionAddProducts` |
| Update collection | `collectionUpdate` |
| Delete collection | `collectionDelete` |

---

## Collection Types

| Type | Description | Best For |
|------|-------------|----------|
| **Smart** | Auto-populates based on rules | Categories, tags, sale items |
| **Manual** | Hand-picked products | Featured, curated sets |

---

## Create Smart Collection

```graphql
mutation collectionCreate($input: CollectionInput!) {
  collectionCreate(input: $input) {
    collection { id title productsCount }
    userErrors { field message }
  }
}
```

### By Product Type
```json
{
  "input": {
    "title": "Art Prints",
    "handle": "art-prints",
    "ruleSet": {
      "appliedDisjunctively": false,
      "rules": [
        { "column": "TYPE", "relation": "EQUALS", "condition": "Art Print" }
      ]
    }
  }
}
```

### By Tag
```json
{
  "input": {
    "title": "Landscape Art",
    "handle": "landscape",
    "ruleSet": {
      "appliedDisjunctively": false,
      "rules": [
        { "column": "TAG", "relation": "EQUALS", "condition": "landscape" }
      ]
    }
  }
}
```

### By Vendor
```json
{
  "input": {
    "title": "Winslow Homer Collection",
    "handle": "winslow-homer",
    "ruleSet": {
      "appliedDisjunctively": false,
      "rules": [
        { "column": "VENDOR", "relation": "EQUALS", "condition": "Winslow Homer" }
      ]
    }
  }
}
```

### By Price (Under $50)
```json
{
  "input": {
    "title": "Under $50",
    "handle": "under-50",
    "ruleSet": {
      "appliedDisjunctively": false,
      "rules": [
        { "column": "VARIANT_PRICE", "relation": "LESS_THAN", "condition": "50" }
      ]
    }
  }
}
```

---

## Rule Reference

| Column | Description |
|--------|-------------|
| `TITLE` | Product title |
| `TYPE` | Product type |
| `VENDOR` | Vendor name |
| `TAG` | Product tag |
| `VARIANT_PRICE` | Variant price |

| Relation | Usage |
|----------|-------|
| `EQUALS` | Exact match |
| `NOT_EQUALS` | Exclude |
| `CONTAINS` | Partial match |
| `GREATER_THAN` | Numeric > |
| `LESS_THAN` | Numeric < |

---

## Add Products to Manual Collection

```graphql
mutation collectionAddProducts($id: ID!, $productIds: [ID!]!) {
  collectionAddProducts(id: $id, productIds: $productIds) {
    collection { id productsCount }
    userErrors { field message }
  }
}
```

---

## Sort Orders

- `MANUAL` - Custom order
- `BEST_SELLING` - By sales
- `ALPHA_ASC` - A-Z
- `PRICE_ASC` - Low to high
- `PRICE_DESC` - High to low
- `CREATED` - Newest first

---

## Scopes Required

- `read_products` - View collections
- `write_products` - Create, update, delete collections
