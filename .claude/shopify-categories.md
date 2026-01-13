# Shopify Standard Product Taxonomy - Quick Reference

## Art & Print Categories (Most Relevant)

| Category ID | Full Path |
|-------------|-----------|
| `hg-3-4` | Home & Garden > Decor > Artwork |
| `hg-3-4-2` | Home & Garden > Decor > Artwork > Posters, Prints, & Visual Artwork |
| `hg-3-4-2-2` | Home & Garden > Decor > Artwork > Posters, Prints, & Visual Artwork > **Prints** |
| `hg-3-4-2-4` | Home & Garden > Decor > Artwork > Posters, Prints, & Visual Artwork > **Paintings** |
| `hg-3-4-2-1` | Home & Garden > Decor > Artwork > Posters, Prints, & Visual Artwork > **Posters** |
| `hg-3-4-2-3` | Home & Garden > Decor > Artwork > Posters, Prints, & Visual Artwork > **Visual Artwork** |
| `hg-3-4-1` | Home & Garden > Decor > Artwork > **Decorative Tapestries** |
| `hg-3-4-3` | Home & Garden > Decor > Artwork > **Sculptures & Statues** |

## How to Use

**Full GID format:**
```
gid://shopify/TaxonomyCategory/hg-3-4-2-2
```

**Set category in productSet:**
```graphql
mutation productSet($input: ProductSetInput!) {
  productSet(input: $input) {
    product { id category { fullName } }
    userErrors { field message }
  }
}
```
```json
{
  "input": {
    "title": "My Art Print",
    "category": "gid://shopify/TaxonomyCategory/hg-3-4-2-2"
  }
}
```

**Update existing product category:**
```graphql
mutation productUpdate($input: ProductInput!) {
  productUpdate(input: $input) {
    product { id category { fullName } }
    userErrors { field message }
  }
}
```
```json
{
  "input": {
    "id": "gid://shopify/Product/XXXXX",
    "category": "gid://shopify/TaxonomyCategory/hg-3-4-2-2"
  }
}
```

---

## Other Common Categories

### Home & Garden > Decor
| ID | Name |
|----|------|
| `hg-3-1` | Clocks |
| `hg-3-2` | Decorative Accents |
| `hg-3-3` | Flags & Banners |
| `hg-3-5` | Candles & Holders |
| `hg-3-6` | Mirrors |
| `hg-3-7` | Picture Frames |
| `hg-3-8` | Vases |
| `hg-3-9` | Wind Chimes |

### Arts & Entertainment
| ID | Name |
|----|------|
| `ae` | Arts & Entertainment (top level) |
| `ae-2-1` | Arts & Crafts |
| `ae-2-2` | Collectibles |

### Apparel (if selling wearable art)
| ID | Name |
|----|------|
| `aa` | Apparel & Accessories (top level) |
| `aa-1` | Clothing |
| `aa-2` | Accessories |

---

## Full Taxonomy Reference

Commerce Hub maintains the complete list:
```
C:\xampp\htdocs\commerce-hub\docs\shopify-categories-full.txt
```

Format: `{GID} : {Full Category Path}`

---

## Category vs productType

| Field | Purpose | Example |
|-------|---------|---------|
| `category` | Shopify Standard Taxonomy (structured) | `gid://shopify/TaxonomyCategory/hg-3-4-2-2` |
| `productType` | Free text field (your custom label) | "Art Print" |

**Use both:**
- `category` = For Google Shopping, marketplace feeds, SEO
- `productType` = For internal organization, smart collections
