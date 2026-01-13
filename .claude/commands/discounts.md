# Discounts & Gift Cards

Create and manage discount codes, automatic discounts, and gift cards.

## Quick Reference

| Task | Method | MCP Tool |
|------|--------|----------|
| List discounts | Query all | `mcp__shopify__listDiscounts` |
| Create discount code | `discountCodeBasicCreate` | `mcp__shopify__createDiscountCode` |
| List gift cards | Query all | `mcp__shopify__listGiftCards` |
| Create gift card | `giftCardCreate` | `mcp__shopify__createGiftCard` |
| Deactivate discount | `discountCodeDeactivate` | GraphQL |

---

## Discount Types

| Type | Description | Use Case |
|------|-------------|----------|
| **Code** | Customer enters code at checkout | Promos, influencer codes |
| **Automatic** | Applied automatically when conditions met | Sales, BOGO |
| **Percentage** | % off order/products | "20% OFF" |
| **Fixed Amount** | $ off order/products | "$10 OFF" |
| **Free Shipping** | Removes shipping cost | "FREE SHIPPING" |
| **Buy X Get Y** | Purchase triggers free/discounted item | "Buy 2 Get 1 Free" |

---

## List Discounts

```
mcp__shopify__listDiscounts
  limit: 10
```

### GraphQL Query
```graphql
query listDiscounts {
  discountNodes(first: 50) {
    nodes {
      id
      discount {
        ... on DiscountCodeBasic {
          title
          status
          startsAt
          endsAt
          usageLimit
          asyncUsageCount
          codes(first: 5) { nodes { code } }
          customerGets {
            value {
              ... on DiscountPercentage { percentage }
              ... on DiscountAmount { amount { amount } }
            }
          }
        }
        ... on DiscountCodeFreeShipping {
          title
          status
          codes(first: 5) { nodes { code } }
        }
        ... on DiscountAutomaticBasic {
          title
          status
          startsAt
          endsAt
        }
      }
    }
  }
}
```

---

## Create Discount Code

### Percentage Discount
```
mcp__shopify__createDiscountCode
  title: "SUMMER20"
  code: "SUMMER20"
  customerGets: {
    "value": { "percentage": 0.20 },
    "items": "all"
  }
  startsAt: "2026-01-01T00:00:00Z"
  endsAt: "2026-01-31T23:59:59Z"
  usageLimit: 100
  appliesOncePerCustomer: true
```

### Fixed Amount Discount
```graphql
mutation discountCodeBasicCreate($basicCodeDiscount: DiscountCodeBasicInput!) {
  discountCodeBasicCreate(basicCodeDiscount: $basicCodeDiscount) {
    codeDiscountNode {
      id
      codeDiscount {
        ... on DiscountCodeBasic {
          title
          codes(first: 1) { nodes { code } }
        }
      }
    }
    userErrors { field message }
  }
}
```

**Variables - $10 Off:**
```json
{
  "basicCodeDiscount": {
    "title": "SAVE10",
    "code": "SAVE10",
    "startsAt": "2026-01-01T00:00:00Z",
    "endsAt": "2026-12-31T23:59:59Z",
    "customerGets": {
      "value": {
        "discountAmount": {
          "amount": "10.00",
          "appliesOnEachItem": false
        }
      },
      "items": {
        "all": true
      }
    },
    "customerSelection": {
      "all": true
    },
    "usageLimit": 500,
    "appliesOncePerCustomer": true
  }
}
```

**Variables - 25% Off:**
```json
{
  "basicCodeDiscount": {
    "title": "FLASH25",
    "code": "FLASH25",
    "startsAt": "2026-01-15T00:00:00Z",
    "endsAt": "2026-01-16T23:59:59Z",
    "customerGets": {
      "value": {
        "percentage": 0.25
      },
      "items": {
        "all": true
      }
    },
    "customerSelection": {
      "all": true
    },
    "usageLimit": 100
  }
}
```

---

## Create Free Shipping Discount

```graphql
mutation discountCodeFreeShippingCreate($freeShippingCodeDiscount: DiscountCodeFreeShippingInput!) {
  discountCodeFreeShippingCreate(freeShippingCodeDiscount: $freeShippingCodeDiscount) {
    codeDiscountNode {
      id
      codeDiscount {
        ... on DiscountCodeFreeShipping { title }
      }
    }
    userErrors { field message }
  }
}
```

**Variables:**
```json
{
  "freeShippingCodeDiscount": {
    "title": "FREESHIP",
    "code": "FREESHIP",
    "startsAt": "2026-01-01T00:00:00Z",
    "customerSelection": {
      "all": true
    },
    "destination": {
      "all": true
    },
    "minimumRequirement": {
      "subtotal": {
        "greaterThanOrEqualToSubtotal": "50.00"
      }
    }
  }
}
```

---

## Create Automatic Discount

Applies automatically at checkout when conditions met.

```graphql
mutation discountAutomaticBasicCreate($automaticBasicDiscount: DiscountAutomaticBasicInput!) {
  discountAutomaticBasicCreate(automaticBasicDiscount: $automaticBasicDiscount) {
    automaticDiscountNode {
      id
      automaticDiscount {
        ... on DiscountAutomaticBasic { title status }
      }
    }
    userErrors { field message }
  }
}
```

**Variables - Auto 10% on orders $100+:**
```json
{
  "automaticBasicDiscount": {
    "title": "10% Off Orders Over $100",
    "startsAt": "2026-01-01T00:00:00Z",
    "customerGets": {
      "value": {
        "percentage": 0.10
      },
      "items": {
        "all": true
      }
    },
    "minimumRequirement": {
      "subtotal": {
        "greaterThanOrEqualToSubtotal": "100.00"
      }
    }
  }
}
```

---

## Create Buy X Get Y Discount

```graphql
mutation discountAutomaticBxgyCreate($automaticBxgyDiscount: DiscountAutomaticBxgyInput!) {
  discountAutomaticBxgyCreate(automaticBxgyDiscount: $automaticBxgyDiscount) {
    automaticDiscountNode { id }
    userErrors { field message }
  }
}
```

**Variables - Buy 2 Get 1 Free:**
```json
{
  "automaticBxgyDiscount": {
    "title": "Buy 2 Get 1 Free",
    "startsAt": "2026-01-01T00:00:00Z",
    "customerBuys": {
      "items": { "all": true },
      "value": { "quantity": "2" }
    },
    "customerGets": {
      "items": { "all": true },
      "value": {
        "discountOnQuantity": {
          "quantity": "1",
          "effect": { "percentage": 1.0 }
        }
      }
    },
    "usesPerOrderLimit": 1
  }
}
```

---

## Discount with Minimum Purchase

```json
{
  "basicCodeDiscount": {
    "title": "$15 Off $75+",
    "code": "SPEND75SAVE15",
    "startsAt": "2026-01-01T00:00:00Z",
    "customerGets": {
      "value": {
        "discountAmount": {
          "amount": "15.00",
          "appliesOnEachItem": false
        }
      },
      "items": { "all": true }
    },
    "customerSelection": { "all": true },
    "minimumRequirement": {
      "subtotal": {
        "greaterThanOrEqualToSubtotal": "75.00"
      }
    }
  }
}
```

---

## Deactivate Discount

```graphql
mutation discountCodeDeactivate($id: ID!) {
  discountCodeDeactivate(id: $id) {
    codeDiscountNode {
      codeDiscount {
        ... on DiscountCodeBasic { status }
      }
    }
    userErrors { field message }
  }
}
```

---

## Delete Discount

```graphql
mutation discountCodeDelete($id: ID!) {
  discountCodeDelete(id: $id) {
    deletedCodeDiscountId
    userErrors { field message }
  }
}
```

---

# Gift Cards

## List Gift Cards

```
mcp__shopify__listGiftCards
  limit: 10
```

### GraphQL Query
```graphql
query listGiftCards {
  giftCards(first: 50) {
    nodes {
      id
      displayValue
      balance { amount currencyCode }
      initialValue { amount currencyCode }
      lastCharacters
      expiresOn
      createdAt
      customer { email }
      enabled
    }
  }
}
```

---

## Create Gift Card

```
mcp__shopify__createGiftCard
  initialValue: "50.00"
  note: "Birthday gift for VIP customer"
```

### With Recipient Email
```
mcp__shopify__createGiftCard
  initialValue: "100.00"
  recipientEmail: "recipient@example.com"
  recipientMessage: "Happy Birthday! Enjoy this gift card."
```

### GraphQL Method
```graphql
mutation giftCardCreate($input: GiftCardCreateInput!) {
  giftCardCreate(input: $input) {
    giftCard {
      id
      code
      displayValue
      balance { amount }
      expiresOn
    }
    userErrors { field message }
  }
}
```

**Variables:**
```json
{
  "input": {
    "initialValue": "75.00",
    "note": "Employee appreciation",
    "expiresOn": "2027-01-01"
  }
}
```

---

## Disable Gift Card

```graphql
mutation giftCardDisable($id: ID!) {
  giftCardDisable(id: $id) {
    giftCard { id enabled }
    userErrors { field message }
  }
}
```

---

## Adjust Gift Card Balance

Add or remove balance:

```graphql
mutation giftCardUpdate($id: ID!, $input: GiftCardUpdateInput!) {
  giftCardUpdate(id: $id, input: $input) {
    giftCard {
      id
      balance { amount }
    }
    userErrors { field message }
  }
}
```

---

## Common Discount Strategies

### Welcome Discount (First Purchase)
```json
{
  "basicCodeDiscount": {
    "title": "Welcome - 15% First Order",
    "code": "WELCOME15",
    "customerGets": {
      "value": { "percentage": 0.15 },
      "items": { "all": true }
    },
    "customerSelection": { "all": true },
    "appliesOncePerCustomer": true
  }
}
```

### Flash Sale (Limited Time)
```json
{
  "basicCodeDiscount": {
    "title": "24-Hour Flash Sale",
    "code": "FLASH30",
    "startsAt": "2026-01-15T00:00:00Z",
    "endsAt": "2026-01-15T23:59:59Z",
    "customerGets": {
      "value": { "percentage": 0.30 },
      "items": { "all": true }
    },
    "usageLimit": 50
  }
}
```

### VIP Exclusive
```json
{
  "basicCodeDiscount": {
    "title": "VIP Exclusive 25% Off",
    "code": "VIP25",
    "customerGets": {
      "value": { "percentage": 0.25 },
      "items": { "all": true }
    },
    "customerSelection": {
      "customers": {
        "add": ["gid://shopify/Customer/123", "gid://shopify/Customer/456"]
      }
    }
  }
}
```

### Tiered Shipping
```javascript
// $5 off shipping for orders $50-99
// Free shipping for orders $100+
const shippingTiers = [
  { min: 50, max: 99, discount: 5, code: "SHIP5" },
  { min: 100, max: null, discount: "free", code: "FREESHIP100" }
];
```

---

## Discount Analytics Query

```graphql
query discountPerformance($id: ID!) {
  discountNode(id: $id) {
    discount {
      ... on DiscountCodeBasic {
        title
        asyncUsageCount
        codes(first: 1) { nodes { code } }
      }
    }
  }
}
```

---

## Scopes Required

- `read_discounts` - View discounts
- `write_discounts` - Create, update, delete discounts
- `read_gift_cards` - View gift cards
- `write_gift_cards` - Create, manage gift cards

