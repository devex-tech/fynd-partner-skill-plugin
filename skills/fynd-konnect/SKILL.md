---
name: fynd-konnect
description: >-
  Fynd Konnect expert. Use when a partner asks about Konnect — connecting
  marketplaces (Amazon, AJIO, Flipkart, Meesho), ERP/WMS (Unicommerce, SAP,
  Vinculum), POS systems, inventory sync, order management, channel mapping,
  or the Konnect REST API (31 operations). Triggers on: "Konnect", "channel
  mapping", "marketplace sync", "ERP integration", "WMS", "inventory sync",
  "multi-channel", "order status", "buffer stock", "AJIO", "Meesho",
  "Unicommerce", "Vinculum", "inbound connector konnect", "channel auth".
argument-hint: "[topic] e.g. 'how do I connect Amazon on Konnect' or 'update inventory API'"
---

# Fynd Konnect Skill

## What Is Konnect?

Fynd Konnect is a multi-channel commerce hub that connects Fynd Commerce with marketplaces, ERP/WMS systems, POS, and webstores — centralizing orders, inventory, pricing, and returns in one dashboard.

**Base URL (Staging):** `https://{{aggregator_name}}.uat.fyndx1.de/`
**Base URL (Production):** `https://{{aggregator_name}}.extensions.fynd.com/`
**Auth header:** `x-access-token: <token>`

---

## Route by Topic

| Question about... | Key content below |
|-------------------|------------------|
| Generate token, auth setup | **Authentication** section |
| Connect a marketplace/ERP | **Channels** section |
| Map products, buffer stock | **Products & Mapping** section |
| Inventory sync, reconciliation | **Inventory** section |
| Orders, statuses, dispatch | **Orders** section |
| Order flow diagrams | **Workflows** section |
| All API endpoints | **API Reference** section |

---

## Authentication

```bash
# Generate access token
GET /aggregator/v1/token?username=YOUR_USERNAME&password=YOUR_PASSWORD

# Use token in all subsequent calls:
x-access-token: <token>
```

**Company-level auth** → one credential → all stores. `locationCode` becomes **mandatory** in order APIs.
**Location-level auth** → one credential → one specific store/warehouse.

**Find credentials:**
- Company-level: Konnect extension → Settings → Username + Token
- Location-level: Konnect extension → Selling Location → Show Token

---

## Channels

### 6 Channel Types
| Type | Examples |
|------|---------|
| **Marketplaces** | Amazon, AJIO, Flipkart, Meesho, FirstCry, Limeroad, Nykaa, Trendyol |
| **Webstores** | Fynd Commerce storefront |
| **ERP/WMS** | Unicommerce, SAP, Vinculum, Browntape, EasyEcom, Increff, OMSGuru |
| **POS** | Ginesys |
| **Inventory Consumer** | FTP/SFTP-based systems |
| **Custom** | Custom API integrations |

### Auth Types
| Type | When to use |
|------|------------|
| **Company Auth** | One credential for all stores (centralized, same vendor code) |
| **Store Auth** | Per-store credentials (franchises, different credentials per location) |

### Buffer Stock Priority
1. Product-level buffer (Channel Mapping) → takes priority
2. Location-level buffer (Locations panel)
3. Account-level buffer (channel config)

> Published inventory = Fynd sellable qty − applicable buffer

---

## Products & Channel Mapping

**Map Fynd Commerce SKUs to channel-specific product codes:**
- Navigate: Products → Channel Mapping
- Identifiers: SKU, EAN, ALU, UPC, ISBN

**Per product per channel:**
- **Channel Identifier** — the code the channel uses for this product
- **Buffer Stock** — qty held back from this channel
- **Channel Status** — toggle Active/Inactive

**Bulk Mapping:** Products → Channel Mapping → Bulk Map → download XLSX → fill → upload
**Auto-Map:** Available on supported channels — matches SKUs automatically (channel identifiers become read-only after)

---

## Inventory

```bash
# Update inventory (max 500 records per call)
PUT /ims/v3/inventory
x-access-token: <token>

# Update pricing (max 500 records per call)
PUT /ims/v3/price
x-access-token: <token>
```

**Best practices:**
- Schedule daily (minimum); real-time for high-velocity SKUs
- Use **delta updates** — only changed SKUs, not full catalog
- `quantity = 0` → stock-out; negative quantities not supported

**Inventory Reconciliation:** Products → Inventory Reconciliation — compare Fynd stock vs channel-reported stock

---

## Orders

### 12 Order Statuses

| Status | OMS State | What it means |
|--------|-----------|--------------|
| `CREATED` | `placed` | New order |
| `CONFIRMED` | `bag_confirmed` | Ready for invoice + courier |
| `PROCESSING` | `bag_invoiced`/`dp_assigned` | Pre-dispatch |
| `COMPLETED` | `bag_packed` | Packed, ready |
| `TRANSIT` | `bag_picked` | Courier picked up |
| `DELIVERED` | `delivery_done` | Delivered |
| `RETURN_PROCESSING` | `return_initiated` | Return/RTO in progress |
| `RETURN_DELIVERED` | `return_bag_delivered` | Return at seller warehouse |
| `RETURN_COMPLETED` | `return_bag_accepted` | Return accepted after QC |
| `CANCELLED` | — | Cancelled |
| `CREDIT_NOTE_GENERATED` | `credit_note_generated` | Credit note issued |

### Fetch Orders
```bash
GET /oms/v3/shipment?orderStatus=CREATED&locationCode=LOC001
```

Required: `orderStatus`. Required if company-level auth: `locationCode`.

### 18 Supported Ordering Channels
`FYND`, `FYND-STORE`, `ECOMM`, `AMAZON_MLF`, `MYNTRA_IN`, `FLIPKART`, `FLIPKARTASSURED`, `AJIO_VMS`, `NYKAA`, `NYKAA_FASHION`, `TATACLIQ_IN`, `TATACLIQ_LUXURY`, `JIOMART`, `UNIKET`, `SHOPIFY_IN`, `TRELL`, `MAGICPIN`, `FY_NEXUS`

---

## Workflows

### Forward Flow (Marketplace/Fynd Logistics)
```
Fetch CREATED → POST /confirm → POST /invoiceUpdate → POST /pack
→ GET /courierDetails (AWB) → POST /labels → POST /dispatch → DELIVERED
```

### Return Flow
```
Fetch RETURN_DELIVERED → POST /return (QC) → POST /invoiceUpdate (credit note)
```

### Self-Ship (is_self_ship: true)
```
PUT /oms/v3/shipment/status with:
  Forward:  dp_assigned → bag_picked → delivery_done
  Return:   return_dp_assigned → return_bag_in_transit → return_bag_delivered
  RTO:      dp_assigned → bag_picked → rto_bag_delivered
```

---

## API Reference (31 operations)

| Group | Key endpoints |
|-------|--------------|
| **Auth** | `GET /aggregator/v1/token` |
| **Catalog** (11) | `GET /v3/catalog/departments`, `GET /ims/v3/listings`, `POST /v3/catalog/product`, `PUT /v3/catalog/product` |
| **Pricing** | `PUT /ims/v3/price` |
| **Inventory** | `PUT /ims/v3/inventory` |
| **Orders** (14) | `GET/POST /oms/v3/shipment`, confirm, cancel, pack, dispatch, invoiceUpdate, courierDetails, labels, invoice, awb, status, track, return |

---

## ERP/WMS Polling Intervals

| Operation | Recommended Interval |
|-----------|---------------------|
| Fetch new orders (CREATED) | Every 15–30 min |
| Fetch confirmed orders | Every 15–30 min |
| Inventory update | At least daily |

---

## Deep Reference

If `fynd-partner-mcp` MCP is connected:
```
fynd_get_docs("konnect", "orders")        — all 12 statuses + order APIs
fynd_get_docs("konnect", "api-reference") — all 31 endpoints
fynd_get_docs("konnect", "workflows")     — forward/RTO/return flow diagrams
fynd_search("your query")
```
