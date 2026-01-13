# Shopify Admin API Token Guide

How to create, configure, and refresh Admin API tokens with proper scopes.

---

## CRITICAL: Enable ALL Scopes From Start

**ALWAYS enable ALL available scopes when creating or updating an app.** This prevents repeated uninstall/reinstall cycles when you discover you need additional access.

Shopify's scope system means:
- Adding new scopes requires creating a new app version
- New versions require reinstalling the app on each store
- Reinstalling requires getting a new access token
- All consuming code must then be updated with the new token

**The cost of unused scopes is zero. The cost of missing scopes is hours of debugging and reconfiguration.**

---

## Quick Reference

| Task | Method |
|------|--------|
| Refresh existing token | `curl` command (see below) |
| Add scopes to app | Dev Dashboard → App → Configuration → **Select ALL** |
| Create new app | Dev Dashboard → Create App → **Enable ALL scopes** |
| Test token | `curl` GraphQL query |

---

## Understanding Token Types

| Prefix | Type | Use Case |
|--------|------|----------|
| `shpca_` | Client Credentials | Server-side apps, MCP, Vercel functions |
| `shpua_` | User Access | User-authenticated apps |
| `shpat_` | Admin API (legacy) | Older custom apps |
| `shpss_` | Client Secret | Never expose - used to obtain tokens |

**For MCP and serverless functions, use `shpca_` tokens (Client Credentials flow).**

---

## Creating an App with Admin Scopes

### Step 1: Access Dev Dashboard
```
https://dev.shopify.com/dashboard/{org_id}/apps
```

### Step 2: Create Custom App
1. Click "Create App"
2. Choose "Custom app" (not public)
3. Name it descriptively (e.g., "Headless Storefront Admin")
4. Select "Custom distribution" for your organization

### Step 3: Configure Admin API Scopes - ENABLE ALL

Go to **Configuration** → **Admin API access scopes** and **SELECT EVERY SCOPE**.

Use these categories as a checklist - enable ALL of them:

**Products & Inventory:**
```
read_products, write_products
read_inventory, write_inventory
read_locations, write_locations
read_product_listings, write_product_listings
read_product_feeds, write_product_feeds
```

**Orders & Fulfillment:**
```
read_orders, write_orders
read_all_orders
read_draft_orders, write_draft_orders
read_fulfillments, write_fulfillments
read_assigned_fulfillment_orders, write_assigned_fulfillment_orders
read_merchant_managed_fulfillment_orders, write_merchant_managed_fulfillment_orders
read_third_party_fulfillment_orders, write_third_party_fulfillment_orders
read_returns, write_returns
read_shipping, write_shipping
```

**Customers:**
```
read_customers, write_customers
read_customer_events
```

**Discounts & Gift Cards:**
```
read_discounts, write_discounts
read_price_rules, write_price_rules
read_gift_cards, write_gift_cards
```

**Content & Metafields:**
```
read_metaobjects, write_metaobjects
read_metaobject_definitions, write_metaobject_definitions
read_content, write_content
read_themes, write_themes
read_files, write_files
```

**Payments & Finance:**
```
read_shopify_payments_accounts
read_shopify_payments_payouts
read_shopify_payments_disputes, write_shopify_payments_disputes
read_payment_terms, write_payment_terms
```

**Marketing & Analytics:**
```
read_marketing_events, write_marketing_events
read_analytics
read_reports, write_reports
```

**Other (Enable All):**
```
read_channels, write_channels
read_checkouts, write_checkouts
read_companies, write_companies
read_markets, write_markets
read_locales, write_locales
read_translations, write_translations
read_publications, write_publications
read_online_store_pages, write_online_store_pages
read_online_store_navigation, write_online_store_navigation
read_script_tags, write_script_tags
read_pixels, write_pixels
```

### Step 4: Create App Version
1. Go to **Versions** tab
2. Click "Create version"
3. Give it a name (e.g., "v1-all-scopes")
4. Verify ALL scopes are selected
5. Click "Create"

### Step 5: Install App to Store
1. Go to **Distribution** tab (in Partners Dashboard)
2. Copy the custom install link
3. Navigate to that link
4. Select your store and click "Install"
5. Approve the scopes

---

## Obtaining Access Token (Client Credentials Flow)

After app is installed, get a token:

```bash
curl -X POST "https://{store}.myshopify.com/admin/oauth/access_token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=client_credentials" \
  -d "client_id={CLIENT_ID}" \
  -d "client_secret={CLIENT_SECRET}"
```

**Response:**
```json
{
  "access_token": "shpca_xxxxxxxxxxxxx",
  "scope": "write_customers,write_orders,...",
  "expires_in": 86399
}
```

---

## Your Store Credentials

### Your Custom App (Full Scopes)

**Client ID:** `YOUR_CLIENT_ID` (from Shopify Admin > Apps > Your App)
**Client Secret:** `YOUR_CLIENT_SECRET` (from Shopify Admin > Apps > Your App)

**Refresh Token Command:**
```bash
curl -X POST "https://YOUR-STORE.myshopify.com/admin/oauth/access_token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=client_credentials" \
  -d "client_id=YOUR_CLIENT_ID" \
  -d "client_secret=YOUR_CLIENT_SECRET"
```

**Token Format:** `shpca_xxxxxxxxxxxxxxxxxxxxxxxxxxxxx`
**Token Expires:** ~24 hours from generation

---

## Refreshing Expired Tokens

Tokens expire after ~24 hours. To refresh:

1. Run the curl command above
2. Copy the new `access_token` from the response
3. Update all locations where the token is stored (see below)

---

## Where to Update Tokens

After refreshing, update in these locations:

### 1. MCP Server .env
Update your Shopify MCP server `.env` file:
```env
SHOPIFY_ACCESS_TOKEN=shpca_NEW_TOKEN_HERE
SHOPIFY_STORE_DOMAIN=your-store.myshopify.com
```

### 2. Restart Claude Code
The MCP server caches the token in memory. After updating .env, restart Claude Code for changes to take effect.

### 3. Vercel Environment Variables (if applicable)
```
Project → Settings → Environment Variables → SHOPIFY_ADMIN_TOKEN
```

### 4. Local Credentials
Store your credentials securely in a location outside of git repositories.

---

## Testing Token & Scopes

### Verify All Granted Scopes
```bash
curl -s -X POST "https://YOUR-STORE.myshopify.com/admin/api/2024-01/graphql.json" \
  -H "Content-Type: application/json" \
  -H "X-Shopify-Access-Token: YOUR_TOKEN" \
  -d '{"query": "{ currentAppInstallation { accessScopes { handle } } }"}'
```

### Test Customer Access
```bash
curl -s -X POST "https://YOUR-STORE.myshopify.com/admin/api/2024-01/graphql.json" \
  -H "Content-Type: application/json" \
  -H "X-Shopify-Access-Token: {TOKEN}" \
  -d '{"query": "{ customers(first: 1) { nodes { email } } }"}'
```

### Test Fulfillment Orders Access
```bash
curl -s -X POST "https://YOUR-STORE.myshopify.com/admin/api/2024-01/graphql.json" \
  -H "Content-Type: application/json" \
  -H "X-Shopify-Access-Token: {TOKEN}" \
  -d '{"query": "{ fulfillmentOrders(first: 1) { nodes { id status } } }"}'
```

**Success:** Returns data
**Access Denied:** Token missing required scope - need to add scope, reinstall app, refresh token
**401 Unauthorized:** Token invalid/expired - refresh token

---

## Adding Scopes to Existing App

If you need to add scopes (because you didn't enable all initially):

1. Go to Dev Dashboard → Your App → **Configuration**
2. Under "Admin API access scopes", **SELECT ALL SCOPES**
3. Go to **Versions** → Create new version with ALL scopes
4. Go to Partners Dashboard → App → **Distribution**
5. Get the custom install link
6. **Uninstall** the app from the store first
7. **Reinstall** using the custom install link
8. **Refresh** the access token (old token won't have new scopes)
9. **Update** all locations with the new token
10. **Restart** Claude Code / any services using the token

---

## Troubleshooting

| Error | Cause | Fix |
|-------|-------|-----|
| 401 Unauthorized | Token expired/invalid | Refresh token |
| Access Denied for {field} | Missing scope | Add ALL scopes, create new version, reinstall, refresh token |
| Invalid client | Wrong client_id/secret | Check credentials in Dev Dashboard → Settings |
| App not installed | App removed from store | Reinstall via Distribution custom link |
| Scope not appearing | Scope is "protected/restricted" | For public apps, some scopes require Partner approval |

---

## Protected/Restricted Scopes Note

Some scopes are restricted for public apps and require Partner Dashboard approval:
- Fulfillment order scopes (`write_assigned_fulfillment_orders`, etc.)
- Some payment-related scopes
- Customer data scopes (require "Protected Customer Data" access)

**For custom apps distributed to your own stores, these are granted immediately when you reinstall the app.**

---

## Links

- [Shopify Dev Dashboard](https://dev.shopify.com/)
- [Partners Dashboard](https://partners.shopify.com/)
- [Admin API Reference](https://shopify.dev/docs/api/admin-graphql)
- [Access Scopes List](https://shopify.dev/docs/api/usage/access-scopes)
- [Client Credentials Flow](https://shopify.dev/docs/apps/build/authentication-authorization/client-credentials)
