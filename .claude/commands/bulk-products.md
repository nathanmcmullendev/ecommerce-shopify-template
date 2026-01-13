# Bulk Product Creation - Verified Working Methods

Tested 2026-01-12 on demo-exec-2026.myshopify.com. All methods verified working.

## Quick Reference

| Task | Method | API Calls |
|------|--------|-----------|
| Product + variants + options + SKUs + inventory + category | `productSet` | 1 |
| Upload images | `productCreateMedia` | 1 |
| Assign images to variants | `productVariantsBulkUpdate` | 1 |
| Create smart collection | `collectionCreate` with `ruleSet` | 1 |
| Change product status (DRAFT/ACTIVE/ARCHIVED) | `productUpdate` | 1 |
| Bulk status change | Parallel `productUpdate` | N |

**Total for complete product with images:** 3 API calls

**Category reference:** See `.claude/shopify-categories.md`

---

## Step 1: Create Product (productSet)

Creates product, options, variants, SKUs, inventory in ONE call.

```graphql
mutation productSet($input: ProductSetInput!) {
  productSet(input: $input) {
    product {
      id
      title
      productType
      variants(first: 100) {
        nodes { id title price sku }
      }
    }
    userErrors { field message }
  }
}
```

**Variables:**
```json
{
  "input": {
    "title": "Product Name",
    "descriptionHtml": "<p>Description</p>",
    "category": "gid://shopify/TaxonomyCategory/hg-3-4-2-2",
    "productType": "Art Print",
    "vendor": "Vendor Name",
    "tags": ["tag1", "tag2"],
    "status": "ACTIVE",
    "productOptions": [
      {
        "name": "Size",
        "values": [{"name": "8x10"}, {"name": "11x14"}, {"name": "16x20"}]
      },
      {
        "name": "Frame",
        "values": [{"name": "Unframed"}, {"name": "Black Frame"}]
      }
    ],
    "variants": [
      {
        "optionValues": [
          {"optionName": "Size", "name": "8x10"},
          {"optionName": "Frame", "name": "Unframed"}
        ],
        "price": "29.99",
        "inventoryItem": {
          "sku": "SKU-8X10-UF",
          "tracked": true
        },
        "inventoryQuantities": [{
          "locationId": "gid://shopify/Location/94651252929",
          "name": "available",
          "quantity": 100
        }]
      }
    ]
  }
}
```

**Key Fields:**
- `category` = Shopify Standard Taxonomy (for Google Shopping/SEO)
- `productType` = Free text category (for smart collections)
- `status` = "ACTIVE" or "DRAFT"
- `inventoryItem.sku` = SKU per variant
- `inventoryItem.tracked` = Enable inventory tracking
- `inventoryQuantities.locationId` = Get via `listLocations`

**Common Art Categories:**
- `hg-3-4-2-2` = Prints
- `hg-3-4-2-4` = Paintings
- `hg-3-4-2-1` = Posters

---

## Step 2: Upload Images (productCreateMedia)

**IMPORTANT:** Use existing Shopify CDN URLs or publicly accessible URLs.

```graphql
mutation productCreateMedia($productId: ID!, $media: [CreateMediaInput!]!) {
  productCreateMedia(productId: $productId, media: $media) {
    media {
      ... on MediaImage { id status alt }
    }
    mediaUserErrors { field message }
  }
}
```

**Variables:**
```json
{
  "productId": "gid://shopify/Product/XXXXX",
  "media": [
    {
      "originalSource": "https://cdn.shopify.com/s/files/1/XXXX/files/image.jpg",
      "alt": "Image description",
      "mediaContentType": "IMAGE"
    }
  ]
}
```

**Working Image Sources:**
- Shopify CDN URLs (from existing products)
- Cloudinary URLs
- Any publicly accessible HTTPS URL

**Status Values:**
- `UPLOADED` / `PROCESSING` = In progress
- `READY` = Success
- `FAILED` = Bad URL or inaccessible

---

## Step 3: Assign Images to Variants (productVariantsBulkUpdate)

```graphql
mutation productVariantsBulkUpdate($productId: ID!, $variants: [ProductVariantsBulkInput!]!) {
  productVariantsBulkUpdate(productId: $productId, variants: $variants) {
    productVariants {
      id title image { url }
    }
    userErrors { field message }
  }
}
```

**Variables:**
```json
{
  "productId": "gid://shopify/Product/XXXXX",
  "variants": [
    {
      "id": "gid://shopify/ProductVariant/XXXXX",
      "mediaId": "gid://shopify/MediaImage/XXXXX"
    }
  ]
}
```

---

## Smart Collections (Auto-Categorize)

Create collection that auto-adds products by productType:

```graphql
mutation collectionCreate($input: CollectionInput!) {
  collectionCreate(input: $input) {
    collection { id title }
    userErrors { field message }
  }
}
```

**Variables:**
```json
{
  "input": {
    "title": "Art Prints",
    "handle": "art-prints",
    "ruleSet": {
      "appliedDisjunctively": false,
      "rules": [
        {"column": "TYPE", "relation": "EQUALS", "condition": "Art Print"}
      ]
    }
  }
}
```

**Rule Columns:** `TYPE`, `TITLE`, `VENDOR`, `TAG`, `VARIANT_PRICE`
**Relations:** `EQUALS`, `NOT_EQUALS`, `CONTAINS`, `STARTS_WITH`, `GREATER_THAN`, `LESS_THAN`

---

## Product Status (DRAFT / ACTIVE / ARCHIVED)

### Status Values

| Status | Visibility | Use Case |
|--------|------------|----------|
| `DRAFT` | Hidden from storefront | Work in progress, not ready to sell |
| `ACTIVE` | Visible on storefront | Ready to sell |
| `ARCHIVED` | Hidden, searchable in admin | Discontinued, out of season |

### Create Product as Draft

```json
{
  "input": {
    "title": "New Product",
    "status": "DRAFT",
    ...
  }
}
```

### Change Single Product Status

```graphql
mutation productUpdate($input: ProductInput!) {
  productUpdate(input: $input) {
    product { id title status }
    userErrors { field message }
  }
}
```

**Variables:**
```json
{
  "input": {
    "id": "gid://shopify/Product/XXXXX",
    "status": "ACTIVE"
  }
}
```

### Bulk Status Change (Multiple Products)

Use parallel mutations for <50 products:

```javascript
const productIds = ["gid://shopify/Product/123", "gid://shopify/Product/456"];
const newStatus = "ACTIVE"; // or "DRAFT" or "ARCHIVED"

// Execute in parallel
await Promise.all(productIds.map(id =>
  executeGraphQL(PRODUCT_UPDATE_MUTATION, {
    input: { id, status: newStatus }
  })
));
```

### Publish All Draft Products

```graphql
query getDraftProducts {
  products(first: 100, query: "status:draft") {
    nodes { id title }
  }
}
```

Then update each to `ACTIVE`.

### Workflow: Create as Draft → Review → Publish

```javascript
// 1. Create product as DRAFT
const { productSet } = await executeGraphQL(PRODUCT_SET_MUTATION, {
  input: {
    title: "New Art Print",
    status: "DRAFT",  // Hidden until ready
    ...
  }
});

// 2. Add images, verify everything looks good

// 3. Publish when ready
await executeGraphQL(PRODUCT_UPDATE_MUTATION, {
  input: {
    id: productSet.product.id,
    status: "ACTIVE"
  }
});
```

---

## Complete Workflow Example

```javascript
// 1. Get location ID
const { locations } = await executeGraphQL(`
  query { locations(first: 1) { nodes { id } } }
`);
const locationId = locations.nodes[0].id;

// 2. Create product with variants
const { productSet } = await executeGraphQL(PRODUCT_SET_MUTATION, {
  input: {
    title: "My Product",
    productType: "Art Print",  // Category
    productOptions: [...],
    variants: [...]
  }
});
const productId = productSet.product.id;
const variantIds = productSet.product.variants.nodes.map(v => v.id);

// 3. Upload images
const { productCreateMedia } = await executeGraphQL(CREATE_MEDIA_MUTATION, {
  productId,
  media: [
    { originalSource: "https://...", alt: "Image 1", mediaContentType: "IMAGE" }
  ]
});
const mediaIds = productCreateMedia.media.map(m => m.id);

// 4. Assign images to variants
await executeGraphQL(VARIANTS_UPDATE_MUTATION, {
  productId,
  variants: variantIds.map((id, i) => ({
    id,
    mediaId: mediaIds[i % mediaIds.length]
  }))
});
```

---

## Location ID Reference

**demo-exec-2026 store:**
```
gid://shopify/Location/94651252929
```

Query your location:
```graphql
query { locations(first: 5) { nodes { id name isActive } } }
```

Or use MCP: `mcp__shopify__listLocations`

---

## MCP Tool Usage

**Use `executeGraphQL` for all mutations:**
```
mcp__shopify__executeGraphQL
  query: "mutation productSet($input: ProductSetInput!) { ... }"
  variables: { "input": { ... } }
```

**Working MCP tools:**
- `listLocations` - Get location IDs
- `getShopInfo` - Shop details
- `listProducts` - Query products
- `executeGraphQL` - All mutations

**Broken MCP tools (use executeGraphQL instead):**
- `getProduct` - query error
- `listCollections` - query error
- `createProduct` - no variant support

---

## Constraints

- Max 3 options per product
- Max 100 variants default (2048 with Extended Variants preview)
- Image URLs must be publicly accessible HTTPS
- Products need `status: "ACTIVE"` to appear on storefront
- Smart collections auto-update when products match rules
