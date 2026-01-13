# Shopify MCP Skill Builder Plan

Master tracking document for building and enhancing Shopify MCP skills.

---

## Current Status Overview

| Skill | Status | Priority | Last Updated | Notes |
|-------|--------|----------|--------------|-------|
| fulfillment.md | ✅ Complete | High | 2026-01-13 | Fixed - uses executeGraphQL, documents ID types |
| shopify-admin-token.md | ✅ Complete | High | 2026-01-13 | Full scope documentation |
| orders.md | ✅ Fixed | High | 2026-01-13 | Added warning, points to fulfillment.md |
| customers.md | ✅ Good | Medium | 2026-01-13 | Verified - comprehensive coverage |
| discounts.md | ✅ Good | Medium | 2026-01-13 | Verified - all discount types |
| draft-orders.md | ✅ Good | Medium | 2026-01-13 | Verified - MCP tool works |
| inventory.md | ✅ Good | Medium | 2026-01-13 | Verified - complete GraphQL examples |
| collections.md | ✅ Good | Medium | 2026-01-13 | Verified - rule-based collections |
| analytics.md | ✅ Fixed | Low | 2026-01-13 | MCP tools deprecated, full GraphQL alternatives documented |
| metafields.md | ✅ Good | Low | 2026-01-13 | Verified - complete coverage |
| bulk-products.md | ✅ Good | Low | 2026-01-13 | Verified - productSet workflow complete |
| bulk-update-categories.md | ✅ Good | Low | 2026-01-13 | Verified - taxonomy mapping |
| import-store.md | ✅ Good | Low | 2026-01-13 | Verified - full import workflow |
| export-store.md | ✅ Good | Low | 2026-01-13 | Verified - JSON and CSV export |

**Legend:**
- ✅ Complete - Fully working, tested, documented
- ⚠️ Needs Fix - Has known issues
- 📝 Review - Needs verification and possible enhancement

---

## Priority Order (Product Interactions)

### Tier 1: Core Operations (High Priority)
1. **Orders** - View, fulfill, refund, cancel
2. **Fulfillment** - Create fulfillments, tracking, notifications
3. **Customers** - CRUD, segments, marketing consent
4. **Products** (via listProducts, getProduct, createProduct, updateProduct)

### Tier 2: Commerce Features (Medium Priority)
5. **Draft Orders** - B2B, phone orders, custom pricing
6. **Inventory** - Stock levels, adjustments, locations
7. **Discounts & Gift Cards** - Promotions, codes
8. **Collections** - Product organization

### Tier 3: Advanced (Lower Priority)
9. **Metafields** - Custom data storage
10. **Analytics** - Sales reports, customer analytics
11. **Bulk Operations** - Mass product creation/updates
12. **Import/Export** - Store data migration

---

## Known Issues to Fix

### 1. createFulfillment MCP Tool (CRITICAL)
**File:** `C:\xampp\htdocs\claude\shopify-mcp-custom\package\build\index.js` lines 618-658
**Problem:** Uses wrong ID types (Order/LineItem instead of FulfillmentOrder/FulfillmentOrderLineItem)
**Workaround:** Use `executeGraphQL` with `fulfillmentCreateV2` mutation
**Status:** ✅ Documented in fulfillment.md, orders.md updated

### 2. orders.md References Broken Tool
**Problem:** Still shows `createFulfillment` as a valid approach
**Fix:** ✅ Added warning pointing to fulfillment.md

### 3. Analytics MCP Tools (getSalesReport, etc.)
**Problem:** Uses `ShopifyPaymentsBalanceTransactionFilter` which was removed in API 2025-01
**Root Cause:** MCP uses API version 2025-01 (`ApiVersion.January25`) but analytics tools were written for older API
**Workaround:** Use `executeGraphQL` with order/customer/product queries
**Status:** ✅ Fixed - analytics.md rewritten with working GraphQL methods

---

## Enhancement Checklist

### For Each Skill File:
- [ ] Verify MCP tools work correctly
- [ ] Add warning if tool has issues (use executeGraphQL instead)
- [ ] Include complete GraphQL examples with variables
- [ ] Add ID type reference (gid://shopify/Type/ID)
- [ ] Include common workflows
- [ ] Add troubleshooting section
- [ ] List required scopes

### Quality Standards:
1. **Tested** - All examples verified working
2. **Complete** - Covers common use cases
3. **Accurate** - Correct ID types and field names
4. **Practical** - Real-world workflows included

---

## Recent Changes Log

### 2026-01-13 (Session 2)
- **SKILL-BUILDER-PLAN.md** - Created
  - Master tracking document for all skills
  - Priority order and status for each skill
  - Known issues and workarounds
  - Token/store configuration

- **orders.md** - Fixed fulfillment section
  - Added warning about broken createFulfillment tool
  - Points to fulfillment.md for correct workflow

- **analytics.md** - Added API issue warning
  - MCP tools have schema errors
  - Added GraphQL alternatives for reporting

- **Verified working:** customers.md, discounts.md, draft-orders.md, inventory.md, collections.md, metafields.md

### 2026-01-13 (Session 1)
- **fulfillment.md** - Complete rewrite
  - Added warning about broken createFulfillment tool
  - Documented correct workflow using executeGraphQL
  - Added ID types table (FulfillmentOrder vs Order, FulfillmentOrderLineItem vs LineItem)
  - Added update tracking, cancel fulfillment, get details examples
  - Added troubleshooting section

- **shopify-admin-token.md** - Updated
  - Added "ENABLE ALL SCOPES" guidance
  - Current credentials and refresh instructions
  - Troubleshooting for scope issues

---

## Store Configuration

**Store:** YOUR-STORE.myshopify.com
**App:** Your Custom App (with required scopes)
**Token:** Get from `shopify-admin-token.md` workflow

### Token Locations:
1. MCP server `.env` file
2. `~/.claude.json` mcpServers.shopify.env

### Token Refresh:
```bash
curl -X POST "https://YOUR-STORE.myshopify.com/admin/oauth/access_token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=client_credentials" \
  -d "client_id=YOUR_CLIENT_ID" \
  -d "client_secret=YOUR_CLIENT_SECRET"
```

---

## Workflow: Enhancing a Skill

1. **Read** current skill file
2. **Test** each MCP tool mentioned
3. **Identify** broken tools or missing features
4. **Document** workarounds (executeGraphQL)
5. **Add** complete examples with variables
6. **Update** this plan with status

---

## Next Steps

1. ~~Fix fulfillment.md~~ ✅
2. ~~Fix orders.md~~ ✅
3. ~~Review draft-orders.md~~ ✅
4. ~~Review inventory.md~~ ✅
5. ~~Review collections.md, analytics.md, metafields.md~~ ✅
6. ~~Review bulk-products.md~~ ✅
7. ~~Review bulk-update-categories.md~~ ✅
8. ~~Review import-store.md, export-store.md~~ ✅

**ALL SKILLS REVIEWED ✅**

Future improvements:
- Fix analytics MCP tools (API schema issue)
- Fix createFulfillment MCP tool (wrong ID types)
- Add products.md skill for basic product CRUD

---

*Last updated: 2026-01-13*
