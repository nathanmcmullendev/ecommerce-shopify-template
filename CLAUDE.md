# Headless Shopify Storefront - Claude Code Instructions

## Project Identity
- **Name:** Ecommerce React Shopify (Headless Storefront)
- **Repo:** github.com/nathanmcmullendev/ecommerce-react-shopify
- **Type:** Production Shopify headless storefront
- **Goal:** Senior developer-quality headless commerce with proper testing, TypeScript, and Shopify best practices

## Directory Context
```
Current Repo:     C:\xampp\htdocs\ecommerce-react-shopify\
Credentials:      C:\xampp\htdocs\PRIVATE\CREDENTIALS-MASTER.md
Related:          C:\xampp\htdocs\HYDROGEN-FORGE\hydrogen-forge-repo\
```

## Available MCP Servers (Global)
All MCPs configured in `~/.claude.json` - available in every session:

| MCP | Purpose | Key Tools |
|-----|---------|-----------|
| **shopify** | Shopify Admin API | `executeGraphQL`, product/order/draft order operations |
| **woocommerce** | WooCommerce REST API | `wc_execute_rest`, product operations |
| **playwright** | Browser automation | Navigate, screenshot, click, fill - E2E testing |
| **github** | GitHub operations | Repo management, commits, PRs |
| **supabase** | Database | PostgreSQL queries |
| **wordpress** | WP management | Site operations |

## Shopify Store Connection
- **Store:** dev-store-749237498237498787.myshopify.com
- **Admin API:** Configured via MCP
- **Storefront API:** Via environment variables
- **Protected Customer Data:** ✅ Enabled (Name, Email, Phone, Address)

## Related Projects
| Project | Location | Purpose |
|---------|----------|---------|
| Hydrogen Forge | `C:\xampp\htdocs\HYDROGEN-FORGE\hydrogen-forge-repo\` | Developer toolkit, MCPs |
| Commerce Hub | `C:\xampp\htdocs\commerce-hub\` | Multi-platform sync |
| RapidWoo | `C:\xampp\htdocs\rapidwoo-storefront\` | WooCommerce headless |
| Credentials | `C:\xampp\htdocs\PRIVATE\CREDENTIALS-MASTER.md` | All API keys |

## Tech Stack
- **Framework:** React + Vite
- **Language:** TypeScript (strict mode)
- **Styling:** Tailwind CSS
- **Testing:** Playwright (E2E), Vitest (unit)
- **Payment:** Stripe (headless checkout)
- **Deployment:** Vercel

## Quality Standards
- ✅ TypeScript strict mode - no `any` types
- ✅ ESLint passing - zero warnings policy
- ✅ Test coverage for critical paths
- ✅ Lighthouse score 90+
- ✅ Error boundaries and graceful degradation

## Development Commands
```bash
npm run dev          # Start development server
npm run build        # Production build
npm run typecheck    # TypeScript validation
npm run lint         # ESLint
npm run test         # Run tests
npx playwright test  # E2E tests
```

## Key Features Implemented
- ✅ Protected Customer Data integration
- ✅ Draft order functionality (Shopify Admin API)
- ✅ Stripe checkout flow
- ✅ Order creation synced to Shopify

## Credentials Access
When credentials needed:
```
Read: C:\xampp\htdocs\PRIVATE\CREDENTIALS-MASTER.md
```

## Session Workflow
```
START
├── Check current branch and recent commits
├── Review any open PRs or issues
├── Run tests to ensure clean state

WORK
├── Follow TypeScript strict standards
├── Test before commit (typecheck, lint, build)
├── Use Playwright MCP for E2E testing when needed

END
├── Commit with meaningful message
├── Push to GitHub
├── Verify deployment on Vercel
```

## Important Rules
1. **Follow Shopify guidelines** - Use official patterns and APIs
2. **TypeScript strict** - No shortcuts, proper typing
3. **Test everything** - Unit + E2E for critical paths
4. **Performance first** - Optimize images, lazy load, cache
5. **Never commit credentials** - Use environment variables

---
*This file auto-loads when Claude Code starts in this directory.*
