# Shopify Claude Skills System

A comprehensive set of Claude Code skills for managing Shopify stores via MCP (Model Context Protocol).

## What This Is

This directory contains **skill files** that teach Claude Code how to perform Shopify operations. Each skill is a markdown file with:
- Working GraphQL queries and mutations
- MCP tool usage examples
- Common workflows
- Troubleshooting guides

## Quick Start

```
# In Claude Code, invoke a skill:
/fulfillment    - Manage order fulfillment
/orders         - Order management
/analytics      - Sales reports
/customers      - Customer management
/discounts      - Promo codes and gift cards
/inventory      - Stock management
/draft-orders   - B2B and wholesale orders
/collections    - Smart collections
/metafields     - Custom data storage
/bulk-products  - Catalog generation
/import-store   - Store migration
/export-store   - Store backup
```

## Architecture

```
.claude/
├── README.md                    # This file
├── commands/
│   ├── SKILL-BUILDER-PLAN.md   # Master status tracking
│   ├── shopify-admin-token.md  # Token management guide
│   │
│   ├── # Core Operations
│   ├── orders.md               # Order management
│   ├── fulfillment.md          # Fulfillment & tracking
│   ├── customers.md            # Customer management
│   ├── inventory.md            # Stock levels
│   │
│   ├── # Commerce Features
│   ├── discounts.md            # Promo codes & gift cards
│   ├── draft-orders.md         # B2B/wholesale
│   ├── collections.md          # Product organization
│   │
│   ├── # Analytics & Reporting
│   ├── analytics.md            # Sales reports, segments
│   │
│   ├── # Bulk Operations
│   ├── bulk-products.md        # Catalog generation
│   ├── bulk-update-categories.md
│   ├── import-store.md         # Store migration
│   ├── export-store.md         # Store backup
│   │
│   └── # Reference
│       └── integrations-research.md  # Frontend integrations (Klaviyo, Subscriptions)
```

## MCP Server Configuration

The skills use the Shopify MCP server configured in `~/.claude.json`:

```json
{
  "mcpServers": {
    "shopify": {
      "command": "node",
      "args": ["path/to/shopify-mcp-custom/package/build/index.js"],
      "env": {
        "SHOPIFY_STORE_DOMAIN": "your-store.myshopify.com",
        "SHOPIFY_ACCESS_TOKEN": "shpca_xxxxx"
      }
    }
  }
}
```

## API Version

These skills are tested with **Shopify Admin GraphQL API 2025-01**.

Key changes in 2025-01:
- `ordersCount` → `numberOfOrders`
- `totalSpent` → `amountSpent { amount currencyCode }`
- `shopifyqlQuery` deprecated (use order/customer queries)
- `ShopifyPaymentsBalanceTransactionFilter` removed

## Known Issues & Workarounds

| Issue | Workaround |
|-------|------------|
| `createFulfillment` MCP tool broken | Use `executeGraphQL` with `fulfillmentCreateV2` |
| Analytics MCP tools error | Use direct order/customer queries |
| `CustomerSortKeys` no TOTAL_SPENT | Query all, sort client-side |

## Token Management

Tokens expire every 24 hours. To refresh:

```bash
curl -X POST "https://YOUR-STORE.myshopify.com/admin/oauth/access_token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=client_credentials" \
  -d "client_id=YOUR_CLIENT_ID" \
  -d "client_secret=YOUR_CLIENT_SECRET"
```

See `commands/shopify-admin-token.md` for full guide.

## Required Scopes

Enable ALL scopes when creating the Shopify app to avoid reinstallation:
- `read_orders`, `write_orders`, `read_all_orders`
- `read_products`, `write_products`
- `read_customers`, `write_customers`
- `read_fulfillments`, `write_fulfillments`
- `read_assigned_fulfillment_orders`, `write_assigned_fulfillment_orders`
- `read_merchant_managed_fulfillment_orders`, `write_merchant_managed_fulfillment_orders`
- `read_draft_orders`, `write_draft_orders`
- `read_inventory`, `write_inventory`
- `read_discounts`, `write_discounts`
- ... (see shopify-admin-token.md for complete list)

## Skill Status

See `commands/SKILL-BUILDER-PLAN.md` for current status of each skill.

## Contributing

When adding or updating skills:

1. Test with real Shopify operations
2. Document any API version requirements
3. Include working GraphQL examples with variables
4. Add troubleshooting section for common errors
5. Update SKILL-BUILDER-PLAN.md status

## Related

- [Shopify GraphQL Admin API](https://shopify.dev/docs/api/admin-graphql/latest)
- [MCP Protocol](https://modelcontextprotocol.io/)
- [Claude Code](https://claude.ai/claude-code)
