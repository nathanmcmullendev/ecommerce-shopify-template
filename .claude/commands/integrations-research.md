# Frontend Integrations Research

Reference guide for Shopify Hydrogen/headless frontend integrations.

---

## Status Summary

| Integration | Status | Official Solution |
|-------------|--------|-------------------|
| **Klaviyo V2** | ✅ Solved | klaviyo-labs/Hydrogen-2-Example |
| **Subscriptions** | ✅ Solved | Shopify hydrogen/examples/subscriptions |
| **Recharge** | ✅ Solved | Recharge official docs |

---

## Klaviyo V2 (Hydrogen 2024.10+)

### Official Solution
**Repo:** https://github.com/klaviyo-labs/Hydrogen-2-Example
- 3 stars, 1 fork (as of Jan 2026)
- Complete implementation
- **Demo:** https://hydrogen2example-3ad64e75e88dc3df299c.o2.myshopify.dev/

### Features Included
- Active on Site tracking
- Viewed Product event
- Added to Cart event
- Email/SMS signup forms
- Customer identification

### Key Files (from official repo)
```
app/
├── components/
│   └── klaviyo/
│       ├── KlaviyoOnsite.client.jsx
│       ├── KlaviyoIdentify.client.jsx
│       └── KlaviyoPublishProductView.client.jsx
└── root.jsx  (script injection)
```

### Implementation Pattern
```tsx
// 1. Add Klaviyo script to root.tsx
<script
  async
  type="text/javascript"
  src={`//static.klaviyo.com/onsite/js/klaviyo.js?company_id=${KLAVIYO_PUBLIC_KEY}`}
/>

// 2. Track product views
useEffect(() => {
  if (typeof window.klaviyo !== 'undefined') {
    window.klaviyo.push(['track', 'Viewed Product', {
      ProductName: product.title,
      ProductID: product.id,
      Categories: product.collections,
      ImageURL: product.featuredImage?.url,
      URL: window.location.href,
      Brand: product.vendor,
      Price: selectedVariant.price.amount,
    }]);
  }
}, [product, selectedVariant]);

// 3. Newsletter signup
window.klaviyo.push(['identify', { $email: email }]);
```

### Outdated Reference (V1)
- https://github.com/klaviyo-labs/klaviyo-shopify-hydrogen-example (V1, deprecated)

---

## Subscriptions / Selling Plans

### Official Solution
**Location:** Shopify Hydrogen repo `examples/subscriptions/`
**Docs:** https://shopify.dev/docs/storefronts/headless/hydrogen/cookbook/subscriptions

### Features Included
- SellingPlanSelector component
- Cart integration with selling plans
- Subscription pricing display
- Customer subscription management

### GraphQL Query Pattern
```graphql
fragment SellingPlanMoney on MoneyV2 {
  amount
  currencyCode
}

fragment SellingPlan on SellingPlan {
  id
  name
  description
  recurringDeliveries
  priceAdjustments {
    adjustmentValue {
      ... on SellingPlanPercentagePriceAdjustment {
        adjustmentPercentage
      }
      ... on SellingPlanFixedAmountPriceAdjustment {
        adjustmentAmount { ...SellingPlanMoney }
      }
      ... on SellingPlanFixedPriceAdjustment {
        price { ...SellingPlanMoney }
      }
    }
    orderCount
  }
}

fragment SellingPlanGroup on SellingPlanGroup {
  name
  options { name values }
  sellingPlans(first: 10) {
    nodes { ...SellingPlan }
  }
}
```

### Cart Integration
```tsx
<CartForm
  action={CartForm.ACTIONS.LinesAdd}
  inputs={{
    lines: [{
      merchandiseId: variant.id,
      quantity: 1,
      sellingPlanId: selectedSellingPlanId, // Add this!
    }]
  }}
>
  <button>{sellingPlanId ? 'Subscribe' : 'Add to Cart'}</button>
</CartForm>
```

---

## Recharge Subscriptions

### Official Docs
**URL:** https://storefront.rechargepayments.com/client/docs/examples/hydrogen/overview/

### Key Points
- **Pro Plan Required** for Storefront API access ($99/mo)
- Uses Shopify Selling Plans under the hood
- Checkout handled by Shopify (no custom checkout)
- Customer portal via Recharge hosted page or API

### Alternative: Native Shopify Subscriptions
For testing without Recharge Pro:
- Use Shopify's native subscription features
- Same Selling Plans API
- No monthly fee for basic functionality

---

## What's Unique (Our MCP Skills)

These admin/backend skills have NO official equivalent:

| Skill | Purpose | Official Alternative |
|-------|---------|---------------------|
| `bulk-products.md` | Mass product creation via Claude | None |
| `fulfillment.md` | Order fulfillment automation | None |
| `analytics.md` | Sales reports (API 2025-01 fixed) | None |
| `import-store.md` | Store data migration | None |
| `export-store.md` | Store backup | None |
| `draft-orders.md` | B2B/wholesale via Claude | None |
| `inventory.md` | Stock management | None |
| `discounts.md` | Promo code automation | None |

**The MCP admin skills are the differentiator.**

---

## Future Expansion

### Frontend Integration Skills to Add
When building frontend features, reference the official implementations:

1. **Klaviyo Skill** - Document how to add Klaviyo to this Vite/React template
   - Adapt patterns from Hydrogen-2-Example
   - Test with real Klaviyo account

2. **Subscriptions Skill** - Document Selling Plans for Vite/React
   - Adapt patterns from Shopify's hydrogen/examples/subscriptions
   - Works with any subscription app (Recharge, Bold, Skio)

3. **Customer Accounts Skill** - Headless customer auth
   - Reference: Shopify Customer Account API
   - OAuth flow for headless

### Architecture Consideration
This template uses **Vite + React** (not Hydrogen/Remix):
- Patterns from Hydrogen need adaptation
- Use React hooks instead of Remix loaders
- Client-side data fetching instead of SSR

---

## Resources

### Official Repos
- [Klaviyo Hydrogen V2](https://github.com/klaviyo-labs/Hydrogen-2-Example)
- [Shopify Hydrogen Examples](https://github.com/Shopify/hydrogen/tree/main/examples)
- [Recharge Storefront Docs](https://storefront.rechargepayments.com/)

### Shopify Docs
- [Subscriptions Cookbook](https://shopify.dev/docs/storefronts/headless/hydrogen/cookbook/subscriptions)
- [Selling Plans API](https://shopify.dev/docs/api/storefront/latest/objects/sellingplan)
- [Customer Account API](https://shopify.dev/docs/api/customer)

### Our MCP Skills
- See `.claude/commands/` for admin automation skills
- See `.claude/README.md` for skill system overview

---

*Last updated: 2026-01-13*
