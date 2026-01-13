# Fulfillment Management

Manage order fulfillment, shipping, and tracking information.

---

## IMPORTANT: Use executeGraphQL for Fulfillments

The `createFulfillment` MCP tool has ID format issues. **Always use `executeGraphQL` for reliable fulfillment operations.**

---

## Quick Reference

| Task | Method |
|------|--------|
| List fulfillment orders | `mcp__shopify__listFulfillmentOrders` |
| Create fulfillment | `mcp__shopify__executeGraphQL` (fulfillmentCreateV2) |
| Update tracking | `mcp__shopify__executeGraphQL` (fulfillmentTrackingInfoUpdateV2) |
| Cancel fulfillment | `mcp__shopify__executeGraphQL` (fulfillmentCancel) |
| Get shipping zones | `mcp__shopify__getShippingZones` |

---

## Complete Fulfillment Workflow

### Step 1: List Open Fulfillment Orders

```
mcp__shopify__listFulfillmentOrders
  limit: 10
  status: "open"
```

Returns fulfillment orders with:
- `id` - FulfillmentOrder ID (needed for fulfillment)
- `lineItems` - Line items with their IDs and quantities
- `order` - Associated order info
- `assignedLocation` - Fulfillment location

### Step 2: Create Fulfillment (GraphQL)

**Use `executeGraphQL` - the built-in createFulfillment tool has ID issues.**

```
mcp__shopify__executeGraphQL
  query: "mutation fulfillmentCreateV2($fulfillment: FulfillmentV2Input!) {
    fulfillmentCreateV2(fulfillment: $fulfillment) {
      fulfillment {
        id
        status
        trackingInfo { number company url }
      }
      userErrors { field message }
    }
  }"
  variables: {
    "fulfillment": {
      "lineItemsByFulfillmentOrder": [{
        "fulfillmentOrderId": "gid://shopify/FulfillmentOrder/123",
        "fulfillmentOrderLineItems": [{
          "id": "gid://shopify/FulfillmentOrderLineItem/456",
          "quantity": 1
        }]
      }],
      "trackingInfo": {
        "number": "1Z999AA10123456784",
        "company": "UPS",
        "url": "https://ups.com/track?tracknum=1Z999AA10123456784"
      },
      "notifyCustomer": true
    }
  }
```

### Key ID Types

| ID Type | Format | Used For |
|---------|--------|----------|
| FulfillmentOrder | `gid://shopify/FulfillmentOrder/123` | lineItemsByFulfillmentOrder.fulfillmentOrderId |
| FulfillmentOrderLineItem | `gid://shopify/FulfillmentOrderLineItem/456` | fulfillmentOrderLineItems.id |
| Order | `gid://shopify/Order/789` | Reference only, NOT for fulfillment |
| LineItem | `gid://shopify/LineItem/012` | NOT for fulfillment (use FulfillmentOrderLineItem) |

---

## Update Tracking Information

```
mcp__shopify__executeGraphQL
  query: "mutation fulfillmentTrackingInfoUpdateV2($fulfillmentId: ID!, $trackingInfoInput: FulfillmentTrackingInput!) {
    fulfillmentTrackingInfoUpdateV2(fulfillmentId: $fulfillmentId, trackingInfoInput: $trackingInfoInput) {
      fulfillment {
        id
        trackingInfo { number company url }
      }
      userErrors { field message }
    }
  }"
  variables: {
    "fulfillmentId": "gid://shopify/Fulfillment/123",
    "trackingInfoInput": {
      "number": "NEWTRACK123",
      "company": "FedEx",
      "url": "https://fedex.com/track?tracknum=NEWTRACK123"
    }
  }
```

---

## Cancel Fulfillment

```
mcp__shopify__executeGraphQL
  query: "mutation fulfillmentCancel($id: ID!) {
    fulfillmentCancel(id: $id) {
      fulfillment { id status }
      userErrors { field message }
    }
  }"
  variables: {
    "id": "gid://shopify/Fulfillment/123"
  }
```

---

## Get Fulfillment Details

### For a Specific Order
```
mcp__shopify__executeGraphQL
  query: "query getFulfillmentOrders($orderId: ID!) {
    order(id: $orderId) {
      id
      name
      fulfillmentOrders(first: 10) {
        nodes {
          id
          status
          assignedLocation { name }
          lineItems(first: 50) {
            nodes {
              id
              totalQuantity
              remainingQuantity
              lineItem { title sku }
            }
          }
        }
      }
      fulfillments(first: 10) {
        nodes {
          id
          status
          createdAt
          trackingInfo { number company url }
          fulfillmentLineItems(first: 50) {
            nodes {
              lineItem { title quantity }
            }
          }
        }
      }
    }
  }"
  variables: { "orderId": "gid://shopify/Order/123" }
```

---

## Bulk Fulfillment Workflow

### 1. Get All Open Fulfillment Orders
```
mcp__shopify__listFulfillmentOrders
  status: "open"
  limit: 50
```

### 2. Fulfill Each Order
For each fulfillment order, use the fulfillmentCreateV2 mutation with:
- The `fulfillmentOrderId` from the list
- All `fulfillmentOrderLineItems` IDs with their `remainingQuantity`

---

## Common Carriers

| Carrier | Company Name | Tracking URL Pattern |
|---------|--------------|---------------------|
| UPS | "UPS" | https://ups.com/track?tracknum={number} |
| FedEx | "FedEx" | https://fedex.com/track?tracknum={number} |
| USPS | "USPS" | https://tools.usps.com/go/TrackConfirmAction?tLabels={number} |
| DHL | "DHL Express" | https://www.dhl.com/en/express/tracking.html?AWB={number} |
| Canada Post | "Canada Post" | https://www.canadapost.ca/track-reperage/en#/search?searchFor={number} |

---

## Fulfillment Statuses

| Status | Meaning |
|--------|---------|
| `OPEN` | Ready to be fulfilled |
| `IN_PROGRESS` | Being processed |
| `CLOSED` | Fully fulfilled |
| `CANCELLED` | Fulfillment cancelled |
| `INCOMPLETE` | Partially fulfilled |
| `PENDING` | Awaiting confirmation |
| `SCHEDULED` | Scheduled for future |

---

## Practical Examples

### Fulfill Order with Tracking
```javascript
// 1. Get fulfillment order details
const result = await mcp__shopify__listFulfillmentOrders({ status: "open" });

// 2. For each open order, create fulfillment with executeGraphQL
// Use: fulfillmentOrderId, fulfillmentOrderLineItems, trackingInfo
```

### Split Shipment (Partial Fulfillment)
```javascript
// Only fulfill some items from a fulfillment order
// Set quantity less than remainingQuantity for partial
{
  "fulfillmentOrderLineItems": [
    { "id": "gid://shopify/FulfillmentOrderLineItem/456", "quantity": 1 }
    // Original had quantity 3, only shipping 1 now
  ]
}
```

### Multiple Tracking Numbers
```javascript
// Create separate fulfillments for each tracking number
// Each fulfillmentCreateV2 call creates one fulfillment with one tracking number
```

---

## Scopes Required

- `read_assigned_fulfillment_orders` - View assigned fulfillment orders
- `write_assigned_fulfillment_orders` - Fulfill assigned orders
- `read_merchant_managed_fulfillment_orders` - View merchant-managed orders
- `write_merchant_managed_fulfillment_orders` - Fulfill merchant-managed orders
- `read_third_party_fulfillment_orders` - View 3PL orders
- `write_third_party_fulfillment_orders` - Fulfill 3PL orders
- `read_fulfillments` - View fulfillments
- `write_fulfillments` - Create/update fulfillments

---

## Troubleshooting

| Error | Cause | Fix |
|-------|-------|-----|
| "Access denied for fulfillmentOrders" | Missing scope | Add fulfillment order scopes, reinstall app, refresh token |
| "invalid id" | Wrong ID type | Use FulfillmentOrderLineItem IDs, not LineItem IDs |
| "Fulfillment order not found" | Wrong fulfillmentOrderId | Get fresh IDs from listFulfillmentOrders |
| "Cannot fulfill" | Order not open | Check fulfillment order status is OPEN |
