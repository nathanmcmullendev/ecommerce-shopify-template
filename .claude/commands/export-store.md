# Export Store Data

Export all Shopify store data to JSON (full backup) and CSV (Shopify-compatible).

## Output Files

| File | Format | Purpose |
|------|--------|---------|
| `shopify-export-YYYY-MM-DD.json` | JSON | Full backup with nested data |
| `shopify-products-YYYY-MM-DD.csv` | CSV | Shopify-compatible for re-import |

---

## Quick Export Query

Export all products with full data in ONE API call:

```graphql
query exportProducts($cursor: String) {
  products(first: 100, after: $cursor) {
    pageInfo { hasNextPage endCursor }
    nodes {
      id
      title
      handle
      descriptionHtml
      productType
      vendor
      tags
      status
      category { id fullName }
      createdAt
      updatedAt
      options { id name values }
      images(first: 20) {
        nodes { id url altText }
      }
      variants(first: 100) {
        nodes {
          id
          title
          sku
          price
          compareAtPrice
          inventoryQuantity
          inventoryItem { id }
          selectedOptions { name value }
          image { url }
        }
      }
      metafields(first: 20) {
        nodes { namespace key value type }
      }
    }
  }
}
```

---

## CSV Column Structure (Shopify Official)

One row per variant. Product info only on first variant row.

```
Handle,Title,Body (HTML),Vendor,Type,Tags,Published,Option1 Name,Option1 Value,Option2 Name,Option2 Value,Option3 Name,Option3 Value,Variant SKU,Variant Price,Variant Compare At Price,Variant Inventory Qty,Image Src,Image Alt Text,Status
```

### CSV Transformation

```javascript
function productsToCSV(products) {
  const headers = [
    'Handle', 'Title', 'Body (HTML)', 'Vendor', 'Type', 'Tags', 'Published',
    'Option1 Name', 'Option1 Value', 'Option2 Name', 'Option2 Value',
    'Option3 Name', 'Option3 Value', 'Variant SKU', 'Variant Price',
    'Variant Compare At Price', 'Variant Inventory Qty',
    'Image Src', 'Image Alt Text', 'Status'
  ];

  const rows = [headers.join(',')];

  for (const product of products) {
    const variants = product.variants.nodes;
    const images = product.images.nodes;
    const options = product.options || [];

    variants.forEach((variant, idx) => {
      const isFirstVariant = idx === 0;
      const variantImage = variant.image?.url || (isFirstVariant && images[0]?.url) || '';

      const row = [
        product.handle,
        isFirstVariant ? csvEscape(product.title) : '',
        isFirstVariant ? csvEscape(product.descriptionHtml) : '',
        isFirstVariant ? csvEscape(product.vendor) : '',
        isFirstVariant ? csvEscape(product.productType) : '',
        isFirstVariant ? csvEscape(product.tags.join(', ')) : '',
        product.status === 'ACTIVE' ? 'TRUE' : 'FALSE',
        options[0]?.name || '',
        variant.selectedOptions[0]?.value || '',
        options[1]?.name || '',
        variant.selectedOptions[1]?.value || '',
        options[2]?.name || '',
        variant.selectedOptions[2]?.value || '',
        variant.sku || '',
        variant.price,
        variant.compareAtPrice || '',
        variant.inventoryQuantity || 0,
        csvEscape(variantImage),
        isFirstVariant && images[0]?.altText ? csvEscape(images[0].altText) : '',
        product.status
      ];

      rows.push(row.join(','));
    });
  }

  return rows.join('\n');
}

function csvEscape(str) {
  if (!str) return '';
  str = String(str);
  if (str.includes(',') || str.includes('"') || str.includes('\n')) {
    return '"' + str.replace(/"/g, '""') + '"';
  }
  return str;
}
```

---

## Export Collections Query

```graphql
query exportCollections {
  collections(first: 100) {
    nodes {
      id
      title
      handle
      descriptionHtml
      image { url altText }
      ruleSet {
        appliedDisjunctively
        rules { column relation condition }
      }
    }
  }
}
```

---

## Complete Export Script

```javascript
async function exportStore() {
  // 1. Get shop info
  const shopInfo = await mcp__shopify__getShopInfo();

  // 2. Get all products
  const productsResult = await executeGraphQL(EXPORT_PRODUCTS_QUERY);
  const products = productsResult.products.nodes;

  // 3. Get collections
  const collectionsResult = await executeGraphQL(EXPORT_COLLECTIONS_QUERY);
  const collections = collectionsResult.collections.nodes;

  // 4. Create JSON export (full backup)
  const jsonExport = {
    exportDate: new Date().toISOString(),
    shop: shopInfo,
    products: products,
    collections: collections,
    summary: {
      totalProducts: products.length,
      totalVariants: products.reduce((sum, p) => sum + p.variants.nodes.length, 0),
      totalCollections: collections.length
    }
  };

  const dateStr = new Date().toISOString().split('T')[0];
  fs.writeFileSync(`exports/shopify-export-${dateStr}.json`, JSON.stringify(jsonExport, null, 2));

  // 5. Create CSV export (Shopify-compatible)
  const csvContent = productsToCSV(products);
  fs.writeFileSync(`exports/shopify-products-${dateStr}.csv`, csvContent);

  return {
    json: `exports/shopify-export-${dateStr}.json`,
    csv: `exports/shopify-products-${dateStr}.csv`,
    products: products.length,
    variants: jsonExport.summary.totalVariants,
    collections: collections.length
  };
}
```

---

## Claude Workflow

When asked to export store data:

1. **Run product query** - Get all products with variants, images, options
2. **Run collections query** - Get all collections with rules
3. **Get shop info** - Store metadata
4. **Generate JSON** - Full nested data for backup/migration
5. **Generate CSV** - Shopify-compatible for spreadsheet editing or re-import
6. **Report** - File locations and counts

---

## JSON vs CSV Comparison

| Aspect | JSON | CSV |
|--------|------|-----|
| Nested data (variants, options) | Preserved | Flattened (one row per variant) |
| Re-import to Shopify | Via API/import skill | Direct via Admin > Products > Import |
| Spreadsheet editing | Not practical | Easy (Excel, Google Sheets) |
| Images array | All preserved | First image per variant |
| Metafields | Preserved | Not included |
| Collections | Included | Separate file needed |
| File size | Larger | Smaller |

**Recommendation:**
- Use **JSON** for full backup, migration, API import
- Use **CSV** for spreadsheet editing, quick Shopify Admin import

---

## Output File Location

```
C:\xampp\htdocs\ecommerce-shopify-template-demo\exports\
├── shopify-export-YYYY-MM-DD.json    (full backup)
└── shopify-products-YYYY-MM-DD.csv   (Shopify-compatible)
```

---

## Token Efficiency

| Approach | API Calls | Output |
|----------|-----------|--------|
| Single bulk query (100 products) | 1 | Both formats |
| Paginated (250+ products) | N/100 | Both formats |

**Best practice:** One query, two output formats.
