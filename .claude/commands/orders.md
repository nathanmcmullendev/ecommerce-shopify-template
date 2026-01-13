# Order Management

Manage orders, fulfillments, refunds, and customer communications.

## Quick Reference

| Task | Method | MCP Tool |
|------|--------|----------|
| List orders | Query with filters | `mcp__shopify__listOrders` |
| Get order details | Query by ID | `mcp__shopify__getOrder` |
| Fulfill order | `fulfillmentCreateV2` | `mcp__shopify__createFulfillment` |
| Refund order | `refundCreate` | `mcp__shopify__createRefund` |
| Cancel order | `orderCancel` | GraphQL |
| Add note | `orderUpdate` | GraphQL |
| Update tracking | `fulfillmentTrackingInfoUpdateV2` | GraphQL |

---

## List Orders

### By Status
```
mcp__shopify__listOrders
  status: "open" | "closed" | "cancelled" | "any"
  limit: 10
```

### By Financial Status
```
mcp__shopify__listOrders
  financialStatus: "paid" | "pending" | "refunded" | "partially_refunded"
```

### By Fulfillment Status
```
mcp__shopify__listOrders
  fulfillmentStatus: "shipped" | "partial" | "unshipped" | "any"
```

### GraphQL Query (Advanced Filters)
```graphql
query getOrders($query: String, $first: Int) {
  orders(first: $first, query: $query) {
    nodes {
      id
      name
      createdAt
      displayFinancialStatus
      displayFulfillmentStatus
      totalPriceSet { shopMoney { amount currencyCode } }
      customer { firstName lastName email }
      shippingAddress { city province country }
      lineItems(first: 10) {
        nodes { title quantity }
      }
    }
  }
}
```

**Query Examples:**
- `"fulfillment_status:unshipped"` - Needs shipping
- `"financial_status:paid"` - Paid orders
- `"created_at:>2026-01-01"` - Orders after date
- `"tag:rush"` - Orders with specific tag

---

## Get Order Details

```
mcp__shopify__getOrder
  id: "gid://shopify/Order/123456789"
```

### Full Order Query
```graphql
query getOrder($id: ID!) {
  order(id: $id) {
    id
    name
    email
    phone
    createdAt
    displayFinancialStatus
    displayFulfillmentStatus
    note
    tags
    totalPriceSet { shopMoney { amount currencyCode } }
    subtotalPriceSet { shopMoney { amount } }
    totalShippingPriceSet { shopMoney { amount } }
    totalTaxSet { shopMoney { amount } }

    customer {
      id
      firstName
      lastName
      email
      phone
      ordersCount
    }

    shippingAddress {
      firstName
      lastName
      address1
      address2
      city
      province
      zip
      country
      phone
    }

    billingAddress {
      firstName
      lastName
      address1
      city
      province
      zip
      country
    }

    lineItems(first: 50) {
      nodes {
        id
        title
        quantity
        sku
        originalUnitPriceSet { shopMoney { amount } }
        variant { id title }
        product { id }
      }
    }

    fulfillments {
      id
      status
      trackingInfo { number url company }
      createdAt
    }

    transactions {
      id
      kind
      status
      amountSet { shopMoney { amount } }
      gateway
    }

    refunds {
      id
      createdAt
      totalRefundedSet { shopMoney { amount } }
    }
  }
}
```

---

## Fulfill Order

> ⚠️ **IMPORTANT:** The `mcp__shopify__createFulfillment` tool has ID format issues.
> **Use `executeGraphQL` with `fulfillmentCreateV2` mutation instead.**
> See [fulfillment.md](fulfillment.md) for complete workflow.

### Correct Fulfillment Workflow

**Step 1:** Get FulfillmentOrder IDs (NOT Order IDs)
```
mcp__shopify__listFulfillmentOrders
  status: "open"
```

**Step 2:** Create fulfillment using GraphQL (see fulfillment.md)

### GraphQL Method
```graphql
mutation fulfillOrder($fulfillment: FulfillmentV2Input!) {
  fulfillmentCreateV2(fulfillment: $fulfillment) {
    fulfillment {
      id
      status
      trackingInfo { number url company }
    }
    userErrors { field message }
  }
}
```

**Variables:**
```json
{
  "fulfillment": {
    "lineItemsByFulfillmentOrder": [
      {
        "fulfillmentOrderId": "gid://shopify/FulfillmentOrder/123",
        "fulfillmentOrderLineItems": [
          { "id": "gid://shopify/FulfillmentOrderLineItem/456", "quantity": 1 }
        ]
      }
    ],
    "trackingInfo": {
      "number": "1Z999AA10123456784",
      "url": "https://ups.com/track",
      "company": "UPS"
    },
    "notifyCustomer": true
  }
}
```

### Common Carriers
| Carrier | Company Name |
|---------|-------------|
| UPS | "UPS" |
| FedEx | "FedEx" |
| USPS | "USPS" |
| DHL | "DHL Express" |

---

## Update Tracking

```graphql
mutation updateTracking($fulfillmentId: ID!, $trackingInfo: FulfillmentTrackingInput!) {
  fulfillmentTrackingInfoUpdateV2(fulfillmentId: $fulfillmentId, trackingInfoInput: $trackingInfo) {
    fulfillment {
      id
      trackingInfo { number url company }
    }
    userErrors { field message }
  }
}
```

**Variables:**
```json
{
  "fulfillmentId": "gid://shopify/Fulfillment/123",
  "trackingInfo": {
    "number": "NEW_TRACKING_NUMBER",
    "url": "https://carrier.com/track",
    "company": "FedEx"
  }
}
```

---

## Refund Order

### Full Refund
```
mcp__shopify__createRefund
  orderId: "gid://shopify/Order/123"
  note: "Customer request"
  notify: true
```

### Partial Refund (Specific Items)
```
mcp__shopify__createRefund
  orderId: "gid://shopify/Order/123"
  lineItems: [
    { "lineItemId": "gid://shopify/LineItem/456", "quantity": 1, "restockType": "RETURN" }
  ]
  note: "Damaged item"
  notify: true
```

### Refund Shipping
```
mcp__shopify__createRefund
  orderId: "gid://shopify/Order/123"
  shipping: { "amount": "9.99", "fullRefund": false }
```

### GraphQL Method
```graphql
mutation refundCreate($input: RefundInput!) {
  refundCreate(input: $input) {
    refund {
      id
      totalRefundedSet { shopMoney { amount } }
    }
    userErrors { field message }
  }
}
```

**Restock Types:**
- `NO_RESTOCK` - Don't return to inventory
- `RETURN` - Item returned, add back to inventory
- `CANCEL` - Order cancelled before shipping

---

## Cancel Order

```graphql
mutation orderCancel($orderId: ID!, $reason: OrderCancelReason!, $refund: Boolean!, $restock: Boolean!) {
  orderCancel(orderId: $orderId, reason: $reason, refund: $refund, restock: $restock) {
    orderCancelUserErrors { field message }
    job { id done }
  }
}
```

**Variables:**
```json
{
  "orderId": "gid://shopify/Order/123",
  "reason": "CUSTOMER",
  "refund": true,
  "restock": true
}
```

**Cancel Reasons:**
- `CUSTOMER` - Customer changed/cancelled order
- `FRAUD` - Fraudulent order
- `INVENTORY` - Items not available
- `DECLINED` - Payment declined
- `OTHER` - Other reason

---

## Add Order Note

```graphql
mutation orderUpdate($input: OrderInput!) {
  orderUpdate(input: $input) {
    order { id note }
    userErrors { field message }
  }
}
```

**Variables:**
```json
{
  "input": {
    "id": "gid://shopify/Order/123",
    "note": "Customer requested gift wrapping"
  }
}
```

---

## Add Order Tags

```graphql
mutation tagsAdd($id: ID!, $tags: [String!]!) {
  tagsAdd(id: $id, tags: $tags) {
    node { ... on Order { id tags } }
    userErrors { field message }
  }
}
```

**Variables:**
```json
{
  "id": "gid://shopify/Order/123",
  "tags": ["rush", "gift", "vip"]
}
```

---

## Common Workflows

### Process Unshipped Orders
```javascript
// 1. Get unshipped orders
const orders = await listOrders({ fulfillmentStatus: "unshipped", status: "open" });

// 2. For each order, create fulfillment
for (const order of orders) {
  await createFulfillment({
    orderId: order.id,
    lineItems: order.lineItems.map(li => ({ id: li.id, quantity: li.quantity })),
    trackingInfo: { number: "TRACKING123", company: "USPS" },
    notifyCustomer: true
  });
}
```

### Daily Order Summary
```javascript
const today = new Date().toISOString().split('T')[0];
const orders = await executeGraphQL(`
  query {
    orders(first: 100, query: "created_at:>=${today}") {
      nodes {
        name
        displayFinancialStatus
        displayFulfillmentStatus
        totalPriceSet { shopMoney { amount } }
      }
    }
  }
`);

const summary = {
  total: orders.nodes.length,
  revenue: orders.nodes.reduce((sum, o) => sum + parseFloat(o.totalPriceSet.shopMoney.amount), 0),
  unfulfilled: orders.nodes.filter(o => o.displayFulfillmentStatus === 'UNFULFILLED').length,
  paid: orders.nodes.filter(o => o.displayFinancialStatus === 'PAID').length
};
```

### Handle Refund Request
```javascript
// 1. Get order details
const order = await getOrder(orderId);

// 2. Calculate refund amount
const itemToRefund = order.lineItems.nodes.find(li => li.sku === 'DAMAGED-SKU');

// 3. Create partial refund
await createRefund({
  orderId: order.id,
  lineItems: [{ lineItemId: itemToRefund.id, quantity: 1, restockType: "RETURN" }],
  note: "Damaged in shipping",
  notify: true
});
```

---

## Order Statuses Reference

### Financial Status
| Status | Meaning |
|--------|---------|
| `PENDING` | Payment not yet captured |
| `AUTHORIZED` | Payment authorized, not captured |
| `PAID` | Payment received |
| `PARTIALLY_PAID` | Partial payment received |
| `PARTIALLY_REFUNDED` | Some items refunded |
| `REFUNDED` | Fully refunded |
| `VOIDED` | Payment voided |

### Fulfillment Status
| Status | Meaning |
|--------|---------|
| `UNFULFILLED` | No items shipped |
| `PARTIALLY_FULFILLED` | Some items shipped |
| `FULFILLED` | All items shipped |
| `RESTOCKED` | Items returned to inventory |

---

## Scopes Required

- `read_orders` - View orders
- `write_orders` - Update, fulfill, refund orders
- `read_fulfillments` - View fulfillment status
- `write_fulfillments` - Create fulfillments

