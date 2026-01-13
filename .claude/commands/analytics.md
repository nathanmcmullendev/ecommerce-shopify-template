# Analytics & Reports

Generate sales reports, product performance, and customer insights.

---

> ⚠️ **Note:** MCP analytics tools (`getSalesReport`, etc.) use deprecated API fields.
> **Use the GraphQL queries below for reliable analytics.**

---

## Quick Reference

| Report | Method |
|--------|--------|
| Sales by date | Order query with date filter |
| Top products | Products query with BEST_SELLING sort |
| Top customers | Customers query + client-side sort by amountSpent |
| Revenue summary | Order aggregation |
| Inventory report | Product inventory query |

---

## Sales Report (Working Method)

Query orders directly and aggregate:

```graphql
query salesReport($query: String!, $first: Int!) {
  orders(first: $first, query: $query) {
    nodes {
      id
      name
      createdAt
      displayFinancialStatus
      displayFulfillmentStatus
      totalPriceSet { shopMoney { amount currencyCode } }
      subtotalPriceSet { shopMoney { amount } }
      totalTaxSet { shopMoney { amount } }
      totalShippingPriceSet { shopMoney { amount } }
      totalDiscountsSet { shopMoney { amount } }
      totalRefundedSet { shopMoney { amount } }
      lineItems(first: 50) {
        nodes {
          title
          quantity
          originalTotalSet { shopMoney { amount } }
        }
      }
    }
    pageInfo { hasNextPage endCursor }
  }
}
```

**Variables (Last 30 Days):**
```json
{
  "query": "created_at:>=2026-01-01 financial_status:paid",
  "first": 100
}
```

**Query Examples:**
- Last 7 days: `"created_at:>=2026-01-07 created_at:<=2026-01-13"`
- This month: `"created_at:>=2026-01-01"`
- Paid only: `"financial_status:paid"`
- Fulfilled: `"fulfillment_status:fulfilled"`

---

## Daily Sales Summary

```javascript
async function getDailySales(startDate, endDate) {
  const query = `
    query getDailySales($query: String!) {
      orders(first: 250, query: $query) {
        nodes {
          createdAt
          totalPriceSet { shopMoney { amount } }
          displayFinancialStatus
        }
      }
    }
  `;

  const result = await executeGraphQL(query, {
    query: `created_at:>=${startDate} created_at:<=${endDate} financial_status:paid`
  });

  // Group by day
  const byDay = {};
  for (const order of result.data.orders.nodes) {
    const day = order.createdAt.split('T')[0];
    if (!byDay[day]) byDay[day] = { count: 0, revenue: 0 };
    byDay[day].count++;
    byDay[day].revenue += parseFloat(order.totalPriceSet.shopMoney.amount);
  }

  return byDay;
}
```

---

## Top Products (by Sales)

```
mcp__shopify__listProducts
  sortKey: "BEST_SELLING"
  limit: 50
```

Or via GraphQL:
```graphql
query topProducts {
  products(first: 50, sortKey: BEST_SELLING) {
    nodes {
      id
      title
      totalInventory
      totalVariants
      priceRangeV2 {
        minVariantPrice { amount currencyCode }
        maxVariantPrice { amount currencyCode }
      }
    }
  }
}
```

---

## Top Customers

> Note: CustomerSortKeys doesn't include TOTAL_SPENT. Query all and sort client-side.

```graphql
query topCustomers {
  customers(first: 100) {
    nodes {
      id
      firstName
      lastName
      email
      numberOfOrders
      amountSpent { amount currencyCode }
      createdAt
      tags
    }
  }
}
```

**Sort client-side by amount spent:**
```javascript
const sorted = result.data.customers.nodes
  .sort((a, b) => parseFloat(b.amountSpent.amount) - parseFloat(a.amountSpent.amount))
  .slice(0, 50);
```

---

## Customer Analytics

### New vs Returning Customers
```graphql
query customerBreakdown {
  # New customers (1 order)
  newCustomers: customers(first: 1, query: "orders_count:1") {
    pageInfo { hasNextPage }
  }

  # Returning customers (2+ orders)
  returning: customers(first: 1, query: "orders_count:>=2") {
    pageInfo { hasNextPage }
  }

  # High value (spent $500+)
  highValue: customers(first: 1, query: "total_spent:>=500") {
    pageInfo { hasNextPage }
  }
}
```

### Customer Segments
```graphql
query customerSegments {
  # VIP customers (high spenders)
  vip: customers(first: 50, query: "total_spent:>=1000") {
    nodes { firstName lastName email amountSpent { amount } numberOfOrders }
  }

  # Recent customers
  recent: customers(first: 50, query: "created_at:>=2026-01-01", sortKey: CREATED_AT, reverse: true) {
    nodes { firstName lastName email createdAt numberOfOrders }
  }
}
```

---

## Inventory Report

```graphql
query inventoryReport {
  products(first: 250) {
    nodes {
      title
      status
      totalInventory
      variants(first: 100) {
        nodes {
          title
          sku
          inventoryQuantity
          price
        }
      }
    }
  }
}
```

### Low Stock Alert
```javascript
const threshold = 10;
const result = await executeGraphQL(INVENTORY_QUERY);

const lowStock = result.data.products.nodes.flatMap(p =>
  p.variants.nodes
    .filter(v => v.inventoryQuantity <= threshold)
    .map(v => ({
      product: p.title,
      variant: v.title,
      sku: v.sku,
      stock: v.inventoryQuantity
    }))
);
```

---

## Revenue Summary

```javascript
async function getRevenueSummary(startDate, endDate) {
  const query = `
    query getOrders($query: String!) {
      orders(first: 250, query: $query) {
        nodes {
          totalPriceSet { shopMoney { amount } }
          subtotalPriceSet { shopMoney { amount } }
          totalTaxSet { shopMoney { amount } }
          totalShippingPriceSet { shopMoney { amount } }
          totalDiscountsSet { shopMoney { amount } }
          totalRefundedSet { shopMoney { amount } }
        }
      }
    }
  `;

  const result = await executeGraphQL(query, {
    query: `created_at:>=${startDate} created_at:<=${endDate}`
  });

  const orders = result.data.orders.nodes;
  return {
    orderCount: orders.length,
    grossRevenue: sum(orders, 'totalPriceSet'),
    subtotal: sum(orders, 'subtotalPriceSet'),
    tax: sum(orders, 'totalTaxSet'),
    shipping: sum(orders, 'totalShippingPriceSet'),
    discounts: sum(orders, 'totalDiscountsSet'),
    refunds: sum(orders, 'totalRefundedSet'),
    netRevenue: sum(orders, 'totalPriceSet') - sum(orders, 'totalRefundedSet')
  };
}

function sum(orders, field) {
  return orders.reduce((sum, o) =>
    sum + parseFloat(o[field]?.shopMoney?.amount || 0), 0
  );
}
```

---

## Export Reports to CSV

```javascript
function toCSV(data, headers) {
  const rows = [headers.join(',')];
  for (const item of data) {
    rows.push(headers.map(h => csvEscape(item[h])).join(','));
  }
  return rows.join('\n');
}

function csvEscape(val) {
  if (val == null) return '';
  val = String(val);
  if (val.includes(',') || val.includes('"') || val.includes('\n')) {
    return '"' + val.replace(/"/g, '""') + '"';
  }
  return val;
}

// Usage
const csv = toCSV(orders, ['name', 'createdAt', 'total', 'status']);
fs.writeFileSync(`reports/orders-${date}.csv`, csv);
```

---

## Scopes Required

- `read_orders` - Order data for sales reports
- `read_products` - Product data and inventory
- `read_customers` - Customer analytics
- `read_reports` - (Legacy) Not needed for direct queries

---

## Why MCP Analytics Tools Don't Work

The built-in MCP analytics tools (`getSalesReport`, etc.) use deprecated API fields:
- `ShopifyPaymentsBalanceTransactionFilter` - Removed in API 2025-01
- These tools were designed for older API versions

**Solution:** Use `executeGraphQL` with the order/customer/product queries above.

---

## Sources

- [Shopify GraphQL Admin API](https://shopify.dev/docs/api/admin-graphql/latest)
- [ShopifyQL syntax reference](https://shopify.dev/docs/api/shopifyql) (deprecated in 2024-07)
- [About API versioning](https://shopify.dev/docs/api/usage/versioning)
