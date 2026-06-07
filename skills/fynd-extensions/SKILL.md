---
name: fynd-extensions
description: >-
  Fynd Extensions expert for all three extension types. Use when building,
  debugging, or deploying a Fynd Commerce extension. Covers General extensions
  (FDK boilerplate, bindings, webhooks), Payment extensions (initiatePaymentSession,
  getPaymentStatus, checksum, gid, refund flow), and Logistics extensions
  (delivery partner, schemes, accounts, assign/cancel shipment, status sync).
  Triggers on: "extension", "FDK", "setupFdk", "fdk extension init",
  "payment extension", "initiatePaymentSession", "gid", "checksum", "refund",
  "logistics extension", "delivery partner", "DP", "scheme", "assign shipment",
  "courier", "bindings", "webhook_config", "boilerplate".
argument-hint: "[extension type + question] e.g. 'payment extension checksum' or 'logistics scheme setup'"
---

# Fynd Extensions Skill

## Step 1 — Identify Extension Type

**General Extension** — custom seller tools, UI, automation, API integrations
> Signs: "build extension", "FDK CLI", "setupFdk", "webhook_config", "boilerplate", "bindings", "platformClient", "extension preview", "Store OS", "Storefront binding"

**Payment Extension** — payment gateway integration
> Signs: "payment extension", "initiatePaymentSession", "getPaymentStatus", "updatePaymentSession", "initiateRefundSession", "getRefundStatus", "gid", "checksum", "HMAC", "aggregator", "refund"

**Logistics Extension** — delivery partner (DP) integration
> Signs: "logistics extension", "delivery partner", "DP", "scheme", "account", "assign shipment", "cancel shipment", "dp_assigned", "TAT", "serviceability", "courier partners"

If unclear, ask: *"Are you building a General, Payment, or Logistics extension?"*

---

## General Extension

### Setup
```bash
npm install -g @gofynd/fdk-cli
fdk login
fdk extension init   # Choose Node + React
```

### Minimal setupFdk
```javascript
import { setupFdk } from "fdk-extension-javascript/express";
import SQLiteStorage from "fdk-extension-javascript/express/storage/sqlite";

const fdkExtension = setupFdk({
  api_key: process.env.EXTENSION_API_KEY,
  api_secret: process.env.EXTENSION_API_SECRET,
  base_url: process.env.EXTENSION_BASE_URL,
  callbacks: {
    auth: async (data) => '/',
    uninstall: async (data) => {},
  },
  storage: new SQLiteStorage("ext.db"),
  access_mode: "offline",   // required for background platformClient calls
  webhook_config: {
    api_path: "/api/webhooks",
    notification_email: "you@company.com",
    subscribe_on_install: true,
    event_map: {
      "company/location/update": { version: "1", handler: myHandler },
    },
  },
});
```

### Call Platform API in a Route
```javascript
fdkExtension.platformApiRoutes.get("/data", async (req, res) => {
  const data = await req.platformClient.catalog.getProducts({});
  res.json(data);
});
```

### Call Platform API in Background/Webhook
```javascript
// requires access_mode: "offline"
const platformClient = await fdkExtension.getPlatformClient(company_id);
const products = await platformClient.catalog.getProducts({});
```

### 3 Binding Types
| Binding | Where it appears | Use case |
|---------|-----------------|---------|
| **Store OS** | Fynd StoreOS (offline stores) | POS integrations, store staff tools |
| **Storefront** | Online storefront (customer-facing) | Product customization widgets |
| **Platform** | Seller dashboard (company level) | Catalog tools, order management, analytics |

### Launch Types
- **Company level** — Extensions tab at company level (catalog, orders, inventory tools)
- **Application level** — Extensions tab inside a sales channel (storefront use cases)

---

## Payment Extension

### 4 Required APIs (POST endpoints your server must implement)

| API | When Fynd calls it | Your response |
|-----|-------------------|---------------|
| `initiatePaymentSession` | Customer starts checkout | Return `redirect_url` to payment page |
| `getPaymentStatus` | Fynd polls for status | Return current payment state |
| `initiateRefundSession` | Refund requested | Initiate refund, return `refund_id` |
| `getRefundStatus` | Fynd polls for refund status | Return current refund state |

### 2 Optional Update APIs
- `updatePaymentSession` — push status proactively (don't wait for poll)
- `updateRefundSession` — push refund status proactively

### Payment Status State Machine
```
started → pending → complete
                 → failed
```

### Refund Status State Machine
```
refund_initiated → refund_pending → refund_done
                                 → refund_failed
                                 → refund_rejected
                                 → refund_disputed
```

### Checksum Generation (HMAC-SHA256)

**Python:**
```python
import hmac, hashlib
def generate_checksum(api_secret: str, body: str) -> str:
    return hmac.new(api_secret.encode(), body.encode(), hashlib.sha256).hexdigest()
```

**JavaScript:**
```javascript
const crypto = require("crypto");
function generateChecksum(apiSecret, body) {
  return crypto.createHmac("sha256", apiSecret).update(body).digest("hex");
}
```

### Key Facts
- **`gid`** = Global ID — Fynd's unique transaction identifier. Always use `gid` to look up transactions, never your own order ID.
- Verify checksum on ALL incoming requests from Fynd
- Generate checksum on ALL outgoing status updates to Fynd
- Implement cron jobs — Fynd polls `getPaymentStatus` and `getRefundStatus` periodically

---

## Logistics Extension

### Core Concepts
| Term | Meaning |
|------|---------|
| **DP** | Delivery Partner — the courier company you're integrating |
| **Scheme** | A DP's service offering (surface delivery, air express, etc.) |
| **Account** | Seller + Scheme combination — created when seller enables a scheme |
| **Standard Scheme** | One scheme → many accounts (general plans) |
| **Custom Scheme** | One scheme → one account (per-seller contracts) |

### 2 Critical Webhook Events (must implement both)
```javascript
webhook_config: {
  api_path: "/api/webhook-events",
  notification_email: "you@company.com",
  event_map: {
    "application/courier-partners/assign": {
      handler: handleAssignShipment,   // OMS ready to assign to DP
      version: "1",
    },
    "application/courier-partners/cancel": {
      handler: handleCancelShipment,   // Order cancelled
      version: "1",
    },
  },
}
```

### After Receiving Assign Shipment → Respond with dp_assigned
```javascript
const platformClient = await fdkExtension.getPlatformClient(companyId);
await platformClient.order.updateShipmentStatus({
  body: {
    statuses: [{
      status: "dp_assigned",
      shipments: [{
        identifier: shipmentId,
        data_updates: {
          entities: [{
            data: {
              meta: {
                courier_partner_extension_id: process.env.EXTENSION_API_KEY,
                courier_partner_scheme_id: schemeId,
                waybill: ["AWB_NUMBER"],
                tracking_url: "https://track.yoursite.com/AWB_NUMBER",
              },
              delivery_awb_number: "AWB_NUMBER",
            }
          }]
        }
      }]
    }]
  }
});
```

### Order Journey
```
CREATED → Confirm → Invoice → Assign DP → Pack → Dispatch → DELIVERED
                                              ↓ (if fail) → dp_not_assigned
                     ↓ (return) → return_dp_assigned → return_bag_delivered
                     ↓ (RTO)   → rto_initiated → rto_bag_delivered
```

### Self-Ship Status Flow
- **Forward:** `dp_assigned` → `bag_picked` → `delivery_done`
- **Return:** `return_dp_assigned` → `return_bag_in_transit` → `return_bag_delivered`
- **RTO:** `dp_assigned` → `bag_picked` → `rto_bag_delivered`

---

## Deep Reference

If `fynd-partner-mcp` MCP is connected:
```
fynd_get_docs("extensions/general", "development")   — webhooks, bindings, code
fynd_get_docs("extensions/payment", "security")      — checksum code in 3 languages
fynd_get_docs("extensions/logistics", "development") — full status sync code
fynd_search("your query")
```
