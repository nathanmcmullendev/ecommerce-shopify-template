# Customer Management

View, search, and manage customer data, order history, and communications.

## Quick Reference

| Task | Method | MCP Tool |
|------|--------|----------|
| List customers | Query with pagination | `mcp__shopify__listCustomers` |
| Search customers | Search by name/email | `mcp__shopify__searchCustomers` |
| Get customer details | Query by ID | `mcp__shopify__getCustomer` |
| Get customer orders | Query order history | `mcp__shopify__getCustomerOrders` |
| Update customer | `customerUpdate` | `mcp__shopify__updateCustomer` |
| Create customer | `customerCreate` | GraphQL |
| Add tags | `tagsAdd` | GraphQL |
| Send email | `customerEmailMarketingConsentUpdate` | GraphQL |

---

## List Customers

```
mcp__shopify__listCustomers
  limit: 10
```

### With Sorting
```
mcp__shopify__listCustomers
  sortKey: "TOTAL_SPENT" | "CREATED_AT" | "UPDATED_AT" | "NAME"
  reverse: true
```

---

## Search Customers

```
mcp__shopify__searchCustomers
  search: "john@example.com"
```

Search works on:
- Email address
- First name
- Last name
- Phone number

---

## Get Customer Details

```
mcp__shopify__getCustomer
  customer_id: 9596970401985
```

### Full Customer Query
```graphql
query getCustomer($id: ID!) {
  customer(id: $id) {
    id
    firstName
    lastName
    email
    phone
    createdAt
    updatedAt
    state
    tags
    note
    verifiedEmail
    taxExempt

    defaultAddress {
      address1
      address2
      city
      province
      zip
      country
      phone
    }

    addresses {
      address1
      city
      province
      country
    }

    ordersCount
    totalSpent

    emailMarketingConsent {
      marketingState
      consentUpdatedAt
    }

    smsMarketingConsent {
      marketingState
      consentUpdatedAt
    }

    metafields(first: 10) {
      nodes { namespace key value }
    }
  }
}
```

---

## Get Customer Order History

```
mcp__shopify__getCustomerOrders
  customer_id: 9596970401985
  limit: 10
```

### GraphQL Query
```graphql
query customerOrders($id: ID!, $first: Int) {
  customer(id: $id) {
    orders(first: $first) {
      nodes {
        id
        name
        createdAt
        displayFinancialStatus
        displayFulfillmentStatus
        totalPriceSet { shopMoney { amount currencyCode } }
        lineItems(first: 5) {
          nodes { title quantity }
        }
      }
    }
  }
}
```

---

## Update Customer

```
mcp__shopify__updateCustomer
  customer_id: 9596970401985
  first_name: "John"
  last_name: "Smith"
  email: "john.smith@example.com"
```

### GraphQL Method
```graphql
mutation customerUpdate($input: CustomerInput!) {
  customerUpdate(input: $input) {
    customer {
      id
      firstName
      lastName
      email
    }
    userErrors { field message }
  }
}
```

**Variables:**
```json
{
  "input": {
    "id": "gid://shopify/Customer/123",
    "firstName": "John",
    "lastName": "Smith",
    "email": "new.email@example.com",
    "phone": "+15551234567",
    "note": "VIP customer",
    "tags": ["vip", "wholesale"]
  }
}
```

---

## Create Customer

```graphql
mutation customerCreate($input: CustomerInput!) {
  customerCreate(input: $input) {
    customer {
      id
      firstName
      lastName
      email
    }
    userErrors { field message }
  }
}
```

**Variables:**
```json
{
  "input": {
    "firstName": "Jane",
    "lastName": "Doe",
    "email": "jane.doe@example.com",
    "phone": "+15559876543",
    "addresses": [
      {
        "address1": "123 Main St",
        "city": "New York",
        "province": "NY",
        "zip": "10001",
        "country": "US"
      }
    ],
    "tags": ["newsletter"],
    "emailMarketingConsent": {
      "marketingState": "SUBSCRIBED",
      "marketingOptInLevel": "SINGLE_OPT_IN"
    }
  }
}
```

---

## Add Customer Tags

```graphql
mutation tagsAdd($id: ID!, $tags: [String!]!) {
  tagsAdd(id: $id, tags: $tags) {
    node { ... on Customer { id tags } }
    userErrors { field message }
  }
}
```

**Variables:**
```json
{
  "id": "gid://shopify/Customer/123",
  "tags": ["vip", "wholesale", "returning"]
}
```

---

## Remove Customer Tags

```graphql
mutation tagsRemove($id: ID!, $tags: [String!]!) {
  tagsRemove(id: $id, tags: $tags) {
    node { ... on Customer { id tags } }
    userErrors { field message }
  }
}
```

---

## Update Marketing Consent

### Email Marketing
```graphql
mutation customerEmailMarketingConsentUpdate($input: CustomerEmailMarketingConsentUpdateInput!) {
  customerEmailMarketingConsentUpdate(input: $input) {
    customer {
      id
      emailMarketingConsent { marketingState }
    }
    userErrors { field message }
  }
}
```

**Variables:**
```json
{
  "input": {
    "customerId": "gid://shopify/Customer/123",
    "emailMarketingConsent": {
      "marketingState": "SUBSCRIBED",
      "marketingOptInLevel": "SINGLE_OPT_IN"
    }
  }
}
```

**Marketing States:**
- `SUBSCRIBED` - Opted in
- `NOT_SUBSCRIBED` - Not opted in
- `UNSUBSCRIBED` - Opted out
- `PENDING` - Awaiting confirmation

---

## Add Customer Note

```graphql
mutation customerUpdate($input: CustomerInput!) {
  customerUpdate(input: $input) {
    customer { id note }
    userErrors { field message }
  }
}
```

**Variables:**
```json
{
  "input": {
    "id": "gid://shopify/Customer/123",
    "note": "Prefers express shipping. Birthday: March 15"
  }
}
```

---

## Customer Segments (Query Filters)

### High-Value Customers
```graphql
query {
  customers(first: 50, query: "total_spent:>500") {
    nodes { id firstName lastName email totalSpent ordersCount }
  }
}
```

### Recent Customers
```graphql
query {
  customers(first: 50, query: "created_at:>2026-01-01") {
    nodes { id email createdAt }
  }
}
```

### Customers with Tag
```graphql
query {
  customers(first: 50, query: "tag:vip") {
    nodes { id firstName lastName email tags }
  }
}
```

### Customers by Location
```graphql
query {
  customers(first: 50, query: "country:US AND state:NY") {
    nodes { id firstName lastName defaultAddress { city province } }
  }
}
```

---

## Common Workflows

### VIP Customer Report
```javascript
const vips = await executeGraphQL(`
  query {
    customers(first: 100, query: "total_spent:>500", sortKey: TOTAL_SPENT, reverse: true) {
      nodes {
        firstName
        lastName
        email
        totalSpent
        ordersCount
        createdAt
      }
    }
  }
`);

console.table(vips.customers.nodes.map(c => ({
  name: `${c.firstName} ${c.lastName}`,
  email: c.email,
  spent: `$${c.totalSpent}`,
  orders: c.ordersCount
})));
```

### Tag Repeat Customers
```javascript
// Find customers with 3+ orders
const repeats = await executeGraphQL(`
  query {
    customers(first: 100, query: "orders_count:>=3") {
      nodes { id ordersCount tags }
    }
  }
`);

// Add "repeat-customer" tag
for (const customer of repeats.customers.nodes) {
  if (!customer.tags.includes('repeat-customer')) {
    await executeGraphQL(TAGS_ADD_MUTATION, {
      id: customer.id,
      tags: ['repeat-customer']
    });
  }
}
```

### Export Customer List
```javascript
const allCustomers = await executeGraphQL(`
  query {
    customers(first: 250) {
      nodes {
        email
        firstName
        lastName
        totalSpent
        ordersCount
        emailMarketingConsent { marketingState }
        defaultAddress { city province country }
      }
    }
  }
`);

// Convert to CSV
const csv = allCustomers.customers.nodes.map(c =>
  [c.email, c.firstName, c.lastName, c.totalSpent, c.ordersCount,
   c.emailMarketingConsent?.marketingState, c.defaultAddress?.city].join(',')
).join('\n');
```

---

## Customer States Reference

| State | Meaning |
|-------|---------|
| `ENABLED` | Active customer account |
| `DISABLED` | Account disabled |
| `INVITED` | Invitation sent, not accepted |
| `DECLINED` | Declined account invitation |

---

## Privacy & Data

### Delete Customer Data (GDPR)
```graphql
mutation customerRequestDataErasure($customerId: ID!) {
  customerRequestDataErasure(customerId: $customerId) {
    userErrors { field message }
  }
}
```

**Note:** This queues data deletion per GDPR requirements.

### Request Customer Data Export
```graphql
mutation customerGenerateAccountActivationUrl($customerId: ID!) {
  customerGenerateAccountActivationUrl(customerId: $customerId) {
    accountActivationUrl
    userErrors { field message }
  }
}
```

---

## Scopes Required

- `read_customers` - View customer data
- `write_customers` - Create, update customers

