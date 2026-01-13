# Draft Orders

Create and manage draft orders for B2B, phone orders, quotes, and headless checkout.

## Quick Reference

| Task | Method | MCP Tool |
|------|--------|----------|
| List draft orders | Query with pagination | `mcp__shopify__listDraftOrders` |
| Create draft order | Build order manually | `mcp__shopify__createDraftOrder` |
| Complete draft order | Convert to order | GraphQL `draftOrderComplete` |
| Send invoice | Email to customer | GraphQL `draftOrderInvoiceSend` |
| Delete draft order | Remove draft | GraphQL `draftOrderDelete` |

---

## List Draft Orders

```
mcp__shopify__listDraftOrders
  limit: 10
```

### With Search Query
```
mcp__shopify__listDraftOrders
  query: "status:open"
  limit: 20
```

---

## Create Draft Order

### Basic Draft Order
```
mcp__shopify__createDraftOrder
  lineItems: [
    { "variantId": "gid://shopify/ProductVariant/123", "quantity": 2 }
  ]
```

### With Customer
```
mcp__shopify__createDraftOrder
  customerId: "gid://shopify/Customer/456"
  lineItems: [
    { "variantId": "gid://shopify/ProductVariant/123", "quantity": 1 }
  ]
  email: "customer@example.com"
```

### With Custom Line Item (no variant)
```
mcp__shopify__createDraftOrder
  lineItems: [
    {
      "title": "Custom Service",
      "originalUnitPrice": "150.00",
      "quantity": 1
    }
  ]
```

### With Addresses
```
mcp__shopify__createDraftOrder
  lineItems: [
    { "variantId": "gid://shopify/ProductVariant/123", "quantity": 1 }
  ]
  shippingAddress: {
    "address1": "123 Main St",
    "city": "New York",
    "province": "NY",
    "zip": "10001",
    "countryCode": "US"
  }
  billingAddress: {
    "address1": "123 Main St",
    "city": "New York",
    "province": "NY",
    "zip": "10001",
    "countryCode": "US"
  }
```

### With Tags and Note
```
mcp__shopify__createDraftOrder
  lineItems: [
    { "variantId": "gid://shopify/ProductVariant/123", "quantity": 1 }
  ]
  tags: ["wholesale", "priority"]
  note: "Rush order - ship by Friday"
```

---

## GraphQL Methods

### Create Draft Order
```graphql
mutation draftOrderCreate($input: DraftOrderInput!) {
  draftOrderCreate(input: $input) {
    draftOrder {
      id
      name
      invoiceUrl
      status
      totalPrice
      lineItems(first: 50) {
        nodes {
          title
          quantity
          originalUnitPriceSet {
            shopMoney { amount currencyCode }
          }
        }
      }
      customer {
        id
        email
      }
    }
    userErrors {
      field
      message
    }
  }
}
```

**Variables:**
```json
{
  "input": {
    "lineItems": [
      {
        "variantId": "gid://shopify/ProductVariant/123",
        "quantity": 2
      }
    ],
    "customerId": "gid://shopify/Customer/456",
    "email": "customer@example.com",
    "note": "Phone order",
    "tags": ["phone-order"],
    "shippingAddress": {
      "address1": "123 Main St",
      "city": "New York",
      "provinceCode": "NY",
      "zip": "10001",
      "countryCode": "US"
    },
    "billingAddress": {
      "address1": "123 Main St",
      "city": "New York",
      "provinceCode": "NY",
      "zip": "10001",
      "countryCode": "US"
    },
    "appliedDiscount": {
      "value": 10.0,
      "valueType": "PERCENTAGE",
      "title": "Wholesale Discount"
    }
  }
}
```

### Complete Draft Order (Convert to Real Order)
```graphql
mutation draftOrderComplete($id: ID!, $paymentPending: Boolean) {
  draftOrderComplete(id: $id, paymentPending: $paymentPending) {
    draftOrder {
      id
      status
      order {
        id
        name
        displayFinancialStatus
      }
    }
    userErrors {
      field
      message
    }
  }
}
```

**Variables:**
```json
{
  "id": "gid://shopify/DraftOrder/123",
  "paymentPending": false
}
```

- `paymentPending: false` - Mark as paid
- `paymentPending: true` - Mark as payment pending

### Send Invoice
```graphql
mutation draftOrderInvoiceSend($id: ID!, $email: DraftOrderInvoiceInput) {
  draftOrderInvoiceSend(id: $id, email: $email) {
    draftOrder {
      id
      invoiceSentAt
    }
    userErrors {
      field
      message
    }
  }
}
```

**Variables:**
```json
{
  "id": "gid://shopify/DraftOrder/123",
  "email": {
    "to": "customer@example.com",
    "from": "store@example.com",
    "subject": "Your Invoice from Our Store",
    "customMessage": "Thank you for your order! Click below to complete payment."
  }
}
```

### Update Draft Order
```graphql
mutation draftOrderUpdate($id: ID!, $input: DraftOrderInput!) {
  draftOrderUpdate(id: $id, input: $input) {
    draftOrder {
      id
      totalPrice
    }
    userErrors {
      field
      message
    }
  }
}
```

### Delete Draft Order
```graphql
mutation draftOrderDelete($input: DraftOrderDeleteInput!) {
  draftOrderDelete(input: $input) {
    deletedId
    userErrors {
      field
      message
    }
  }
}
```

**Variables:**
```json
{
  "input": {
    "id": "gid://shopify/DraftOrder/123"
  }
}
```

### Calculate Draft Order (Preview Totals)
```graphql
mutation draftOrderCalculate($input: DraftOrderInput!) {
  draftOrderCalculate(input: $input) {
    calculatedDraftOrder {
      lineItems {
        title
        quantity
        discountedTotalSet {
          shopMoney { amount currencyCode }
        }
      }
      totalPriceSet {
        shopMoney { amount currencyCode }
      }
      totalTaxSet {
        shopMoney { amount currencyCode }
      }
      totalShippingPriceSet {
        shopMoney { amount currencyCode }
      }
    }
    userErrors {
      field
      message
    }
  }
}
```

---

## Applying Discounts

### Percentage Discount
```json
{
  "appliedDiscount": {
    "value": 15.0,
    "valueType": "PERCENTAGE",
    "title": "15% Off"
  }
}
```

### Fixed Amount Discount
```json
{
  "appliedDiscount": {
    "value": 25.0,
    "valueType": "FIXED_AMOUNT",
    "title": "$25 Off"
  }
}
```

### Line Item Discount
```json
{
  "lineItems": [
    {
      "variantId": "gid://shopify/ProductVariant/123",
      "quantity": 1,
      "appliedDiscount": {
        "value": 10.0,
        "valueType": "PERCENTAGE",
        "title": "Item Discount"
      }
    }
  ]
}
```

---

## Draft Order Statuses

| Status | Meaning |
|--------|---------|
| `OPEN` | Draft, not converted |
| `INVOICE_SENT` | Invoice emailed to customer |
| `COMPLETED` | Converted to real order |

---

## Common Workflows

### Headless Checkout Flow
```javascript
// 1. Create draft order from cart
const { draftOrderCreate } = await executeGraphQL(DRAFT_ORDER_CREATE, {
  input: {
    lineItems: cartItems.map(item => ({
      variantId: item.variantId,
      quantity: item.quantity
    })),
    email: customerEmail,
    shippingAddress: shippingAddress,
    billingAddress: billingAddress
  }
});

const draftOrderId = draftOrderCreate.draftOrder.id;

// 2. Process payment externally (Stripe, etc.)
const paymentResult = await processStripePayment(amount);

// 3. Complete draft order
if (paymentResult.success) {
  const { draftOrderComplete } = await executeGraphQL(DRAFT_ORDER_COMPLETE, {
    id: draftOrderId,
    paymentPending: false
  });

  return draftOrderComplete.draftOrder.order; // Real Shopify order
}
```

### B2B Quote Flow
```javascript
// 1. Create draft order as quote
const { draftOrderCreate } = await executeGraphQL(DRAFT_ORDER_CREATE, {
  input: {
    customerId: businessCustomerId,
    lineItems: quoteItems,
    appliedDiscount: {
      value: 20.0,
      valueType: "PERCENTAGE",
      title: "Wholesale Discount"
    },
    note: "Quote valid for 30 days",
    tags: ["quote", "b2b"]
  }
});

// 2. Send invoice/quote to customer
await executeGraphQL(DRAFT_ORDER_INVOICE_SEND, {
  id: draftOrderCreate.draftOrder.id,
  email: {
    to: customerEmail,
    subject: "Your Quote from Our Store",
    customMessage: "Please review and click to complete your order."
  }
});
```

### Phone Order Flow
```javascript
// 1. Create draft order while on phone
const draftOrder = await mcp__shopify__createDraftOrder({
  lineItems: [
    { variantId: "gid://shopify/ProductVariant/123", quantity: 2 }
  ],
  email: "customer@example.com",
  note: "Phone order - customer on hold",
  tags: ["phone-order"]
});

// 2. Add shipping address
await executeGraphQL(DRAFT_ORDER_UPDATE, {
  id: draftOrder.id,
  input: {
    shippingAddress: {
      address1: "123 Main St",
      city: "New York",
      provinceCode: "NY",
      zip: "10001",
      countryCode: "US"
    }
  }
});

// 3. Complete with payment info
await executeGraphQL(DRAFT_ORDER_COMPLETE, {
  id: draftOrder.id,
  paymentPending: false
});
```

### Bulk Create Draft Orders
```javascript
const customers = await mcp__shopify__listCustomers({ limit: 50 });

for (const customer of customers.filter(c => c.tags.includes('reorder'))) {
  await mcp__shopify__createDraftOrder({
    customerId: customer.id,
    lineItems: [
      { variantId: "gid://shopify/ProductVariant/123", quantity: 1 }
    ],
    note: "Monthly reorder",
    tags: ["auto-reorder"]
  });
}
```

---

## Scopes Required

- `write_draft_orders` - Create/update draft orders
- `read_draft_orders` - View draft orders
- `write_orders` - Complete draft orders
- `read_customers` - Link to customers
- `read_products` - Add product variants
