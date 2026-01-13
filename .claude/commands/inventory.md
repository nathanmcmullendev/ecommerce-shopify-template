# Inventory Management

Manage stock levels, adjust inventory, and find low stock products.

## Quick Reference

| Task | Method | API Calls |
|------|--------|-----------|
| Set inventory to specific quantity | `inventorySetQuantities` | 1 |
| Adjust inventory (+/-) | `inventoryAdjustQuantities` | 1 |
| Get inventory levels | Query products with variants | 1 |
| Find low stock | Query with inventory filter | 1 |

---

## Set Inventory (Exact Quantity)

Set a variant's stock to a specific number:

```graphql
mutation inventorySetQuantities($input: InventorySetQuantitiesInput!) {
  inventorySetQuantities(input: $input) {
    inventoryAdjustmentGroup {
      reason
      changes { name delta }
    }
    userErrors { field message }
  }
}
```

**Variables:**
```json
{
  "input": {
    "reason": "correction",
    "name": "available",
    "ignoreCompareQuantity": true,
    "quantities": [
      {
        "inventoryItemId": "gid://shopify/InventoryItem/XXXXX",
        "locationId": "gid://shopify/Location/94651252929",
        "quantity": 50
      }
    ]
  }
}
```

**Reason values:** `correction`, `cycle_count_available`, `damaged`, `movement_created`, `movement_updated`, `movement_received`, `movement_canceled`, `other`, `promotion`, `quality_control`, `received`, `reservation_created`, `reservation_deleted`, `reservation_updated`, `restock`, `safety_stock`, `shrinkage`

---

## Adjust Inventory (Add/Subtract)

Add or subtract from current stock:

```graphql
mutation inventoryAdjustQuantities($input: InventoryAdjustQuantitiesInput!) {
  inventoryAdjustQuantities(input: $input) {
    inventoryAdjustmentGroup {
      reason
      changes { name delta }
    }
    userErrors { field message }
  }
}
```

**Variables (subtract 5):**
```json
{
  "input": {
    "reason": "correction",
    "name": "available",
    "changes": [
      {
        "inventoryItemId": "gid://shopify/InventoryItem/XXXXX",
        "locationId": "gid://shopify/Location/94651252929",
        "delta": -5
      }
    ]
  }
}
```

**Variables (add 10):**
```json
{
  "input": {
    "reason": "received",
    "name": "available",
    "changes": [
      {
        "inventoryItemId": "gid://shopify/InventoryItem/XXXXX",
        "locationId": "gid://shopify/Location/94651252929",
        "delta": 10
      }
    ]
  }
}
```

---

## Get Inventory Levels

Query products with inventory data:

```graphql
query getInventory {
  products(first: 100) {
    nodes {
      title
      totalInventory
      variants(first: 100) {
        nodes {
          id
          title
          sku
          inventoryQuantity
          inventoryItem {
            id
            tracked
          }
        }
      }
    }
  }
}
```

---

## Find Low Stock Products

Products with inventory below threshold:

```graphql
query lowStock {
  products(first: 100) {
    nodes {
      title
      variants(first: 100) {
        nodes {
          title
          sku
          inventoryQuantity
          inventoryItem { id }
        }
      }
    }
  }
}
```

Then filter in code:
```javascript
const threshold = 10;
const lowStock = products.nodes.flatMap(p =>
  p.variants.nodes
    .filter(v => v.inventoryQuantity <= threshold)
    .map(v => ({
      product: p.title,
      variant: v.title,
      sku: v.sku,
      quantity: v.inventoryQuantity,
      inventoryItemId: v.inventoryItem.id
    }))
);
```

---

## Bulk Set Inventory (Multiple Variants)

Set inventory for multiple variants in one call:

```json
{
  "input": {
    "reason": "correction",
    "name": "available",
    "ignoreCompareQuantity": true,
    "quantities": [
      {
        "inventoryItemId": "gid://shopify/InventoryItem/111",
        "locationId": "gid://shopify/Location/94651252929",
        "quantity": 100
      },
      {
        "inventoryItemId": "gid://shopify/InventoryItem/222",
        "locationId": "gid://shopify/Location/94651252929",
        "quantity": 50
      },
      {
        "inventoryItemId": "gid://shopify/InventoryItem/333",
        "locationId": "gid://shopify/Location/94651252929",
        "quantity": 25
      }
    ]
  }
}
```

---

## Enable Inventory Tracking

If `tracked: false`, enable tracking first:

```graphql
mutation inventoryItemUpdate($id: ID!, $input: InventoryItemInput!) {
  inventoryItemUpdate(id: $id, input: $input) {
    inventoryItem { id tracked }
    userErrors { field message }
  }
}
```

**Variables:**
```json
{
  "id": "gid://shopify/InventoryItem/XXXXX",
  "input": {
    "tracked": true
  }
}
```

---

## Get Location ID

```graphql
query { locations(first: 5) { nodes { id name isActive } } }
```

Or use MCP: `mcp__shopify__listLocations`

**demo-exec-2026 store:**
```
gid://shopify/Location/94651252929
```

---

## Inventory Workflow Examples

### Restock After Shipment Received
```javascript
const items = [
  { inventoryItemId: "gid://shopify/InventoryItem/111", delta: 50 },
  { inventoryItemId: "gid://shopify/InventoryItem/222", delta: 30 }
];

await executeGraphQL(ADJUST_MUTATION, {
  input: {
    reason: "received",
    name: "available",
    changes: items.map(i => ({
      ...i,
      locationId: "gid://shopify/Location/94651252929"
    }))
  }
});
```

### Set All Variants to Same Quantity
```javascript
// Get all inventory item IDs for a product
const product = await executeGraphQL(GET_PRODUCT_QUERY, { id: productId });
const inventoryItemIds = product.variants.nodes.map(v => v.inventoryItem.id);

// Set all to 100
await executeGraphQL(SET_MUTATION, {
  input: {
    reason: "correction",
    name: "available",
    ignoreCompareQuantity: true,
    quantities: inventoryItemIds.map(id => ({
      inventoryItemId: id,
      locationId: "gid://shopify/Location/94651252929",
      quantity: 100
    }))
  }
});
```

### Low Stock Alert Report
```javascript
const threshold = 5;
const { products } = await executeGraphQL(GET_INVENTORY_QUERY);

const alerts = [];
for (const product of products.nodes) {
  for (const variant of product.variants.nodes) {
    if (variant.inventoryQuantity <= threshold) {
      alerts.push({
        product: product.title,
        variant: variant.title,
        sku: variant.sku,
        stock: variant.inventoryQuantity
      });
    }
  }
}

console.table(alerts);
```

---

## Multi-Location Inventory

For stores with multiple locations, specify each location:

```json
{
  "input": {
    "reason": "movement_created",
    "name": "available",
    "changes": [
      {
        "inventoryItemId": "gid://shopify/InventoryItem/XXXXX",
        "locationId": "gid://shopify/Location/WAREHOUSE_1",
        "delta": -10
      },
      {
        "inventoryItemId": "gid://shopify/InventoryItem/XXXXX",
        "locationId": "gid://shopify/Location/WAREHOUSE_2",
        "delta": 10
      }
    ]
  }
}
```

---

## Constraints

- Must have inventory tracking enabled (`tracked: true`)
- Quantity cannot go negative (unless overselling allowed)
- `ignoreCompareQuantity: true` required for set operations
- Max items per mutation: ~100 (use batching for more)
