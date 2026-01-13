# Import Store Data

Import products, collections, and store data from JSON backup file.

## Prerequisites

- Export file from `export-store.md` skill
- Target store location ID (get via `listLocations`)
- Publicly accessible image URLs (Shopify CDN URLs from export work)

---

## Quick Import (Single Product)

```graphql
mutation productSet($input: ProductSetInput!) {
  productSet(input: $input) {
    product {
      id
      title
      variants(first: 100) { nodes { id title sku } }
    }
    userErrors { field message }
  }
}
```

---

## Full Import Workflow

### Step 1: Get Target Store Location

```graphql
query { locations(first: 1) { nodes { id name } } }
```

Or use MCP: `mcp__shopify__listLocations`

### Step 2: Read Export File

```javascript
const exportData = JSON.parse(fs.readFileSync('exports/shopify-export-YYYY-MM-DD.json'));
const products = exportData.products;
const collections = exportData.collections;
```

### Step 3: Import Products

For each product, transform to `ProductSetInput`:

```javascript
function transformProduct(product, locationId) {
  return {
    title: product.title,
    handle: product.handle,
    descriptionHtml: product.descriptionHtml,
    productType: product.productType,
    vendor: product.vendor,
    tags: product.tags,
    status: product.status,
    category: product.category?.id || null,
    productOptions: product.options.map(opt => ({
      name: opt.name,
      values: opt.values.map(v => ({ name: v }))
    })),
    variants: product.variants.nodes.map(variant => ({
      optionValues: variant.selectedOptions.map(so => ({
        optionName: so.name,
        name: so.value
      })),
      price: variant.price,
      compareAtPrice: variant.compareAtPrice,
      inventoryItem: {
        sku: variant.sku || '',
        tracked: true
      },
      inventoryQuantities: [{
        locationId: locationId,
        name: "available",
        quantity: variant.inventoryQuantity || 0
      }]
    }))
  };
}
```

### Step 4: Import Images

After product creation, add images:

```graphql
mutation productCreateMedia($productId: ID!, $media: [CreateMediaInput!]!) {
  productCreateMedia(productId: $productId, media: $media) {
    media { ... on MediaImage { id status } }
    mediaUserErrors { field message }
  }
}
```

```javascript
const mediaInput = product.images.nodes.map(img => ({
  originalSource: img.url,
  alt: img.altText || product.title,
  mediaContentType: "IMAGE"
}));
```

### Step 5: Import Collections

Smart collections (with rules):

```graphql
mutation collectionCreate($input: CollectionInput!) {
  collectionCreate(input: $input) {
    collection { id title }
    userErrors { field message }
  }
}
```

```javascript
function transformCollection(collection) {
  const input = {
    title: collection.title,
    handle: collection.handle,
    descriptionHtml: collection.descriptionHtml
  };

  if (collection.ruleSet) {
    input.ruleSet = {
      appliedDisjunctively: collection.ruleSet.appliedDisjunctively,
      rules: collection.ruleSet.rules.map(r => ({
        column: r.column,
        relation: r.relation,
        condition: r.condition
      }))
    };
  }

  return input;
}
```

---

## Complete Import Script

```javascript
async function importStore(exportFile, locationId) {
  const exportData = JSON.parse(fs.readFileSync(exportFile));
  const results = { products: [], collections: [], errors: [] };

  // 1. Import products
  for (const product of exportData.products) {
    try {
      const input = transformProduct(product, locationId);
      const result = await executeGraphQL(PRODUCT_SET_MUTATION, { input });

      if (result.productSet.userErrors.length > 0) {
        results.errors.push({ product: product.title, errors: result.productSet.userErrors });
      } else {
        const newProductId = result.productSet.product.id;
        results.products.push({ old: product.id, new: newProductId, title: product.title });

        // 2. Import images
        if (product.images.nodes.length > 0) {
          const mediaInput = product.images.nodes.map(img => ({
            originalSource: img.url,
            alt: img.altText || product.title,
            mediaContentType: "IMAGE"
          }));
          await executeGraphQL(CREATE_MEDIA_MUTATION, {
            productId: newProductId,
            media: mediaInput
          });
        }
      }
    } catch (e) {
      results.errors.push({ product: product.title, error: e.message });
    }
  }

  // 3. Import collections
  for (const collection of exportData.collections) {
    if (collection.handle === 'frontpage') continue; // Skip default

    try {
      const input = transformCollection(collection);
      const result = await executeGraphQL(COLLECTION_CREATE_MUTATION, { input });

      if (result.collectionCreate.userErrors.length > 0) {
        results.errors.push({ collection: collection.title, errors: result.collectionCreate.userErrors });
      } else {
        results.collections.push({ title: collection.title, id: result.collectionCreate.collection.id });
      }
    } catch (e) {
      results.errors.push({ collection: collection.title, error: e.message });
    }
  }

  return results;
}
```

---

## Claude Workflow

When asked to import store data:

1. **Read export file** - Parse the JSON export
2. **Get location ID** - Query target store's location
3. **Import products** - Use parallel `productSet` for <50 products
4. **Import images** - Use `productCreateMedia` after each product
5. **Import collections** - Recreate smart collections with rulesets
6. **Report results** - Show success/failure counts

---

## Handling Duplicates

Check if product exists before import:

```graphql
query checkProduct($handle: String!) {
  productByHandle(handle: $handle) {
    id
    title
  }
}
```

Options:
- **Skip** - Don't import if exists
- **Update** - Use `productUpdate` instead of `productSet`
- **Rename** - Append suffix to handle

---

## Rate Limits

| Store Type | API Limit |
|------------|-----------|
| Development | 50 req/sec |
| Production | Based on plan |

For large imports (100+ products):
- Add 100ms delay between requests
- Use Bulk Operations API for 500+ products

---

## Example: Import to New Store

```javascript
// 1. Get location
const { locations } = await mcp__shopify__listLocations();
const locationId = locations[0].id;

// 2. Run import
const results = await importStore(
  'exports/shopify-export-2026-01-12.json',
  locationId
);

// 3. Report
console.log(`Imported ${results.products.length} products`);
console.log(`Imported ${results.collections.length} collections`);
console.log(`Errors: ${results.errors.length}`);
```

---

## Limitations

- **Images**: Must use publicly accessible URLs (Shopify CDN URLs from same store work)
- **Metafields**: Require separate `metafieldsSet` mutation
- **Reviews**: Require app-specific import (not via Admin API)
- **Customers**: Require `customerCreate` mutations with privacy considerations
- **Orders**: Historical orders typically not re-imported (use for reference only)

---

## File Locations

Export files: `C:\xampp\htdocs\ecommerce-shopify-template-demo\exports\`

Naming: `shopify-export-{YYYY-MM-DD}.json`
