# Metafields Management

Store custom data on products, collections, customers, and orders.

## Quick Reference

| Task | Method |
|------|--------|
| Set metafield | `metafieldsSet` |
| Get metafields | Query on resource |
| Delete metafield | `metafieldDelete` |
| Create definition | `metafieldDefinitionCreate` |

---

## Metafield Types

| Type | Description | Example |
|------|-------------|---------|
| `single_line_text_field` | Short text | "Artist Name" |
| `multi_line_text_field` | Long text | Description |
| `number_integer` | Whole number | 1920 |
| `number_decimal` | Decimal | 24.5 |
| `boolean` | True/false | true |
| `date` | Date | "2026-01-15" |
| `date_time` | Date + time | "2026-01-15T10:00:00Z" |
| `url` | URL | "https://..." |
| `json` | JSON object | {"key": "value"} |
| `color` | Hex color | "#FF5733" |
| `rich_text_field` | HTML content | Rich text |
| `file_reference` | File/image | File GID |
| `product_reference` | Product link | Product GID |

---

## Set Metafield

```graphql
mutation metafieldsSet($metafields: [MetafieldsSetInput!]!) {
  metafieldsSet(metafields: $metafields) {
    metafields {
      id
      namespace
      key
      value
    }
    userErrors { field message }
  }
}
```

### On Product
```json
{
  "metafields": [
    {
      "ownerId": "gid://shopify/Product/123",
      "namespace": "custom",
      "key": "artist",
      "value": "Winslow Homer",
      "type": "single_line_text_field"
    }
  ]
}
```

### On Customer
```json
{
  "metafields": [
    {
      "ownerId": "gid://shopify/Customer/123",
      "namespace": "custom",
      "key": "vip_level",
      "value": "gold",
      "type": "single_line_text_field"
    }
  ]
}
```

### Multiple Metafields
```json
{
  "metafields": [
    {
      "ownerId": "gid://shopify/Product/123",
      "namespace": "art",
      "key": "medium",
      "value": "Oil on canvas",
      "type": "single_line_text_field"
    },
    {
      "ownerId": "gid://shopify/Product/123",
      "namespace": "art",
      "key": "year",
      "value": "1890",
      "type": "number_integer"
    },
    {
      "ownerId": "gid://shopify/Product/123",
      "namespace": "art",
      "key": "dimensions",
      "value": "30 x 40 inches",
      "type": "single_line_text_field"
    }
  ]
}
```

---

## Get Metafields

### On Product
```graphql
query getProductMetafields($id: ID!) {
  product(id: $id) {
    metafields(first: 20) {
      nodes {
        namespace
        key
        value
        type
      }
    }
  }
}
```

### Specific Metafield
```graphql
query getMetafield($id: ID!, $namespace: String!, $key: String!) {
  product(id: $id) {
    metafield(namespace: $namespace, key: $key) {
      value
      type
    }
  }
}
```

---

## Delete Metafield

```graphql
mutation metafieldDelete($input: MetafieldDeleteInput!) {
  metafieldDelete(input: $input) {
    deletedId
    userErrors { field message }
  }
}
```

**Variables:**
```json
{
  "input": {
    "id": "gid://shopify/Metafield/123"
  }
}
```

---

## MCP Tool

```
mcp__shopify__setMetafield
  ownerId: "gid://shopify/Product/123"
  namespace: "custom"
  key: "artist"
  value: "Winslow Homer"
  type: "single_line_text_field"
```

---

## Common Use Cases

### Product SEO
```json
{
  "metafields": [
    {
      "ownerId": "gid://shopify/Product/123",
      "namespace": "seo",
      "key": "meta_title",
      "value": "High Cliff, Coast of Maine - Winslow Homer Art Print",
      "type": "single_line_text_field"
    },
    {
      "ownerId": "gid://shopify/Product/123",
      "namespace": "seo",
      "key": "meta_description",
      "value": "Museum-quality print of Winslow Homer's iconic seascape...",
      "type": "multi_line_text_field"
    }
  ]
}
```

### Art Details
```json
{
  "metafields": [
    { "namespace": "art", "key": "artist", "value": "Winslow Homer", "type": "single_line_text_field" },
    { "namespace": "art", "key": "year", "value": "1894", "type": "number_integer" },
    { "namespace": "art", "key": "medium", "value": "Oil on canvas", "type": "single_line_text_field" },
    { "namespace": "art", "key": "museum", "value": "Smithsonian American Art Museum", "type": "single_line_text_field" }
  ]
}
```

### Customer Preferences
```json
{
  "metafields": [
    { "ownerId": "gid://shopify/Customer/123", "namespace": "preferences", "key": "frame_style", "value": "black", "type": "single_line_text_field" },
    { "ownerId": "gid://shopify/Customer/123", "namespace": "preferences", "key": "newsletter", "value": "true", "type": "boolean" }
  ]
}
```

---

## Namespace Conventions

| Namespace | Purpose |
|-----------|---------|
| `custom` | General custom fields |
| `seo` | SEO metadata |
| `art` | Artwork details |
| `shipping` | Shipping info |
| `preferences` | Customer preferences |
| `internal` | Internal notes |

---

## Scopes Required

- `read_metafields` - View metafields (included with resource scope)
- `write_metafields` - Create, update, delete metafields
