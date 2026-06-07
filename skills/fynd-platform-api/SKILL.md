---
name: fynd-platform-api
description: >-
  Fynd Platform API expert. Use when a partner asks about Platform REST APIs —
  finding the right endpoint, understanding parameters, debugging a failing call
  (401/400/404), auth setup, or SDK starter code. Covers all 843 operations
  across Catalog, Orders, Cart, Content, Logistics, Payment, User, Communication,
  Configuration, and more. Triggers on: "which API", "platform API", "endpoint
  for", "fynd API", "company_id", "PlatformClient", "operationId",
  "access_token", "401 from fynd", "400 from fynd", any Fynd SDK class name.
argument-hint: "[use case or error] e.g. 'list all products' or 'getting 401 on catalog API'"
---

# Fynd Platform API Skill

## Detect Mode First

**Discovery** — partner describes a use case, needs the right endpoint
> Signs: "how do I", "which API", "I want to", "find me an endpoint", "get products", "update inventory"

**Debug** — partner has a failing call
> Signs: "getting 400/401/404/422", "error from", "not working", shares a payload or curl output

---

## Mode: Discovery

### Step 1 — Match domain

| Domain | Ops | What it covers |
|--------|-----|---------------|
| **Catalog** | 162 | Products, brands, categories, inventory, templates, attributes |
| **Content** | 122 | Pages, blogs, SEO, announcements, navigations, slideshows |
| **Logistic** | 84 | Courier partners, serviceability, TAT, pincode check |
| **Cart** | 70 | Cart CRUD, promotions, price rules, payment options |
| **Communication** | 62 | Email, SMS, push notifications, templates |
| **Orders** | 74 | Shipments, returns, invoices, fulfillment |
| **Payment** | 54 | Payment modes, aggregators, refunds |
| **Configuration** | 43 | App/store settings, currencies, languages |
| **User** | 38 | Customer management, sessions, addresses |
| **Other** | 134 | Theme, billing, webhook, discount, filestorage, analytics, rewards |

### Step 2 — Construct the response

Always include:
1. **Endpoint** — `METHOD /service/platform/{domain}/v{version}/company/{company_id}/...`
2. **operationId** — exact name from the SDK
3. **SDK call** — `platformClient.{SdkClass}.{operationId}(params)`
4. **Required params** — path + required query params with types
5. **Starter code** — minimal working example

### SDK Setup (JavaScript)
```javascript
const { PlatformClient, PlatformConfig } = require("fdk-client-javascript");

const config = new PlatformConfig({
  companyId: YOUR_COMPANY_ID,
  apiKey: process.env.API_KEY,
  apiSecret: process.env.API_SECRET,
});
const client = new PlatformClient(config);
```

### SDK Setup (Python)
```python
from fdk_client.platform import PlatformClient, PlatformConfig

config = PlatformConfig(
  company_id=YOUR_COMPANY_ID,
  api_key=os.environ["API_KEY"],
  api_secret=os.environ["API_SECRET"]
)
client = PlatformClient(config)
```

### Common Quick Answers

**List products:**
```javascript
const result = await client.catalog.getProducts({ pageNo: 1, pageSize: 20 });
```

**Update inventory:**
```javascript
await client.catalog.updateInventories({ body: { items: [{ size: "S", store_id: 1, quantity: 10, identifiers: [{ gtin_type: "sku", gtin_value: "SKU001" }] }] } });
```

**Get orders:**
```javascript
const orders = await client.order.getOrders({ pageNo: 1, pageSize: 20, stage: "pending" });
```

---

## Mode: Debug

### By Status Code

**401 Unauthorized**
- Token expired — regenerate via OAuth or token API
- Wrong `companyId` in config
- API key/secret mismatch
- Missing `x-fp-sdk-version` header (rare)

**404 Not Found**
- Wrong resource ID (product_id, order_id, etc.)
- Mismatched `company_id` vs `application_id`
- Resource belongs to a different company

**400 Bad Request / 422 Unprocessable**
- Missing required field in request body
- Wrong data type (string where integer expected)
- Business rule violation (e.g., can't update a delivered order)
- Check the operation's parameter schema

**403 Forbidden**
- API key doesn't have permission for this operation
- Trying to access another company's data

### Debug Checklist
```
1. Is company_id correct for your API key?
2. Is the resource ID (product_id, order_id) correct?
3. Check request body against the schema — all required fields present?
4. Is the token still valid? (tokens expire)
5. Are you using the right base URL? (api.fynd.com vs staging)
```

---

## Auth Reference

### Token Generation
```javascript
// OAuth flow — redirect user to:
`https://api.fynd.com/service/panel/authentication/v1.0/company/${company_id}/oauth/token`

// With headers:
// Authorization: Basic base64(api_key:api_secret)
```

### Base URLs
| Environment | URL |
|-------------|-----|
| Production | `https://api.fynd.com` |
| Staging | `https://api.fyndx1.de` |

---

## Deep Reference

If the MCP server `fynd-partner-mcp` is connected, use:
```
fynd_get_docs("platform-api", "catalog")      — 162 catalog operations
fynd_get_docs("platform-api", "api-index")    — full 843 op index
fynd_get_docs("platform-api", "auth")         — auth setup details
fynd_search("your query")                      — search across all docs
```
