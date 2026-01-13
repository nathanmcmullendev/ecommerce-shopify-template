# Bulk Update Product Categories

Update all products' Shopify Standard Taxonomy categories efficiently using Bulk Operations API.

## Quick Method (For < 50 products)

Use batch productUpdate mutations in parallel:

```graphql
mutation productUpdate($input: ProductInput!) {
  productUpdate(input: $input) {
    product { id title category { fullName } }
    userErrors { field message }
  }
}
```

## Efficient Method (For 50+ products)

### Step 1: Get all products needing category update

```graphql
query getProductsWithoutCategory {
  products(first: 250, query: "category:NULL") {
    nodes {
      id
      title
      productType
    }
    pageInfo {
      hasNextPage
      endCursor
    }
  }
}
```

### Step 2: Create JSONL file

Each line = one product update:
```jsonl
{"input":{"id":"gid://shopify/Product/123","category":"gid://shopify/TaxonomyCategory/hg-3-4-2-2"}}
{"input":{"id":"gid://shopify/Product/456","category":"gid://shopify/TaxonomyCategory/hg-3-4-2-2"}}
```

### Step 3: Stage upload

```graphql
mutation {
  stagedUploadsCreate(input: [{
    resource: BULK_MUTATION_VARIABLES,
    filename: "category-updates.jsonl",
    mimeType: "text/jsonl",
    httpMethod: POST
  }]) {
    stagedTargets {
      url
      resourceUrl
      parameters { name value }
    }
    userErrors { message }
  }
}
```

### Step 4: Upload JSONL to staged URL

POST multipart form with all parameters + file.

### Step 5: Run bulk mutation

```graphql
mutation {
  bulkOperationRunMutation(
    mutation: "mutation($input: ProductInput!) { productUpdate(input: $input) { product { id } userErrors { message } } }",
    stagedUploadPath: "tmp/XXXXX/category-updates.jsonl"
  ) {
    bulkOperation { id status }
    userErrors { message }
  }
}
```

### Step 6: Poll for completion

```graphql
query {
  currentBulkOperation {
    id
    status
    objectCount
    url
    errorCode
  }
}
```

---

## Category Mapping by productType

| productType | Category ID | Category Name |
|-------------|-------------|---------------|
| Art Print | `hg-3-4-2-2` | Prints |
| Painting | `hg-3-4-2-4` | Paintings |
| Poster | `hg-3-4-2-1` | Posters |
| Canvas | `hg-3-4-2-3` | Visual Artwork |
| Sculpture | `hg-3-4-3` | Sculptures & Statues |
| Tapestry | `hg-3-4-1` | Decorative Tapestries |

---

## Claude Workflow

When asked to update categories for all products:

1. **Query products** - Get all products with null category
2. **Map categories** - Match productType to taxonomy category
3. **Execute updates** - Use parallel productUpdate for <50, Bulk API for 50+
4. **Verify** - Query products to confirm categories set

---

## Example: Update All "Art Print" Products

```javascript
// 1. Get products
const { products } = await query(`
  query {
    products(first: 250, query: "product_type:'Art Print' AND category:NULL") {
      nodes { id }
    }
  }
`);

// 2. Build updates
const updates = products.nodes.map(p => ({
  input: {
    id: p.id,
    category: "gid://shopify/TaxonomyCategory/hg-3-4-2-2"
  }
}));

// 3. Execute (parallel for small batches)
for (const update of updates) {
  await mutation(`
    mutation($input: ProductInput!) {
      productUpdate(input: $input) {
        product { id category { name } }
      }
    }
  `, update);
}
```
