---
name: fynd-ai-pim
description: >-
  Fynd AI PIM expert. Use when a partner asks about AI PIM (Product Information
  Management) — SKU enrichment, taxonomy setup (categories/attributes/templates),
  inbound/outbound connectors, rule engine, marketplace transformations, AI PIM
  REST API, authentication, or user roles. Triggers on: "AI PIM", "aipim",
  "SKU enrichment", "taxonomy", "inbound connector", "outbound connector",
  "mapper", "rule engine", "transformations", "SPD", "PPD", "staged product",
  "production product", "ai-pim API", "product enrichment".
argument-hint: "[topic] e.g. 'how do I set up an inbound connector' or 'what are AI PIM roles'"
---

# Fynd AI PIM Skill

## What Is AI PIM?

Fynd AI PIM is an AI-powered Product Information Management platform. It centralizes product data, enriches SKUs using AI (extracting attributes from images, generating SEO content), and publishes formatted data to marketplaces like Amazon, Flipkart, and Myntra.

**Base URL:** `https://api.aipim.fynd.com`

---

## Route by Topic

| Question about... | Key content below |
|-------------------|------------------|
| SKU lifecycle, SPD/PPD/APD | **Products** section |
| Categories, attributes, templates, trees | **Taxonomy** section |
| AI enrichment, rule engine, transformations | **Enrichment & Automation** section |
| Inbound/outbound connectors, mappers | **Connectors** section |
| API endpoints | **API Reference** section |
| Auth, tokens | **Authentication** section |
| Roles, permissions | **Roles** section |

---

## Products — SKU Lifecycle

```
SPD (Staged)  →  PPD (Production)  →  APD (Archived, 30-day retention)
Raw → Enrichment → Audit → Production (Sync to Production)
```

| Stage | Who | Status |
|-------|-----|--------|
| Raw | Catalog Manager | SKU exists, no data |
| Enrichment | Content Enricher team | Filling in attributes |
| Audit | Auditor team | Quality check |
| Production | Catalog Manager | Sync to live storefront |

**Create single SKU:** Products → Staged Product Data → Add Product
**Bulk import:** Products → Staged Product Data → Bulk Options → Bulk SKU Data (Import)
**Publish:** Select SKUs in Audit Done → ellipsis → Sync to Production

---

## Taxonomy

Build in this order: **Attributes → Categories → Templates → Category Mappings → Relationships**

| Concept | What It Is |
|---------|-----------|
| **Attribute** | A single data field (Color, Size, Description). Types: Short Text, Number, List, Media, JSONata |
| **Category** | Organizes products by shared traits (Footwear, Electronics) |
| **Template** | A named group of attributes applied to products |
| **Category Mapping** | Links a category to a template — gives an SKU its attributes via `Category Tree Code` |
| **Relationship** | Variant (same product, different attrs) or Parent-Child (bundles) |

---

## Enrichment & Automation

### Two Enrichment Flows
**Full Automation:** Upload images → AI auto-detects category → maps to global template → extracts attributes + generates SEO content

**Partial Automation:** Create taxonomy → upload SKU package → configure rules → AI fills gaps

### Rule Engine (Settings → Rule Engine)
Rules trigger on data events → filter SKUs → run up to 5 sequential actions:
1. **JavaScript Editor** — custom JS code, can call external APIs
2. **Background Generator** — AI adds image background (prompt-based)
3. **Background Removal** — AI removes image background
4. **Content Extraction** — AI reads images → auto-populates attributes
5. **Content Generation** — AI generates titles, descriptions, bullet points
6. **Map Values to Attributes** — must be the **last** action in a rule

### Transformations (Settings → Transformations)
Map your taxonomy → AI PIM's global taxonomy once → all future products auto-format for Amazon/Flipkart/Myntra/Nykaa. One-time setup.

---

## Connectors

### Inbound — External System → AI PIM

Each inbound connector generates **app credentials** (app ID + token).

**External system sends data:**
```bash
POST {base_url}/service/public/catalog/v1.0/product/import
Headers:
  x-app-id: <connector_app_id>
  x-app-token: <connector_app_token>

Body (single SKU):
{ "sku": "PROD-001", "name": "Blue T-Shirt", "price": 499 }
```

**Response:**
```json
{ "success": true, "total": 1, "message": "1 Products received and will be processed", "errors": [] }
```

**Processing stages:** Pending → In Progress → Completed / Partial Complete / Failed

### Outbound — AI PIM → External System

Pushes product events to your webhook URL when: products created/updated/deleted, attributes/templates/relationships change.

**Integration types:** Rest API (any HTTPS endpoint) or Fynd Platform (direct to Fynd store)

**5 event categories:** Attribute, Category Mapping, Product, Relationship Mapping, Template

**HMAC signing (optional):** AI PIM signs each request body with HMAC-SHA256 → `x-cc-signature` header.

### Mappers — Transform Outbound Payloads

JavaScript transform scripts — run before outbound delivery:

```javascript
async function transform(eventData) {
  const HEADER_MAPPING = ctx.getHeaderMapping();
  const VALUE_MAPPING = ctx.getValueMapping();

  return {
    sku: eventData.product_data?.sku,
    category: VALUE_MAPPING[eventData.product_data?.category_code],
  };
}
```

**Available libraries:** Lodash (`_`), CryptoJS, axios (GET/POST), JSONata
**Timeout:** 60 seconds

---

## API Reference (50 operations across 8 groups)

| Group | Ops | Base path |
|-------|-----|-----------|
| Category Tree | 11 | `/templates/v1.0/org/{orgId}/category/mapping` |
| Inbound | 9 | `/integrations/v2.0/org/{orgId}/inbound` |
| Attribute | 6 | `/templates/v1.0/org/{orgId}/attribute` |
| Outbound (Subscribers) | 7 | `/webhook/v1.0/company/{company_id}/subscriber` |
| Template | 6 | `/templates/v1.0/org/{orgId}/template` |
| Mapper | 6 | `/webhook/v1.0/org/{orgId}/mapper` |
| Category | 5 | `/templates/v1.0/org/{orgId}/category` |

All Panel API paths: `/service/panel/...`

---

## Authentication

| API Type | Method | Header |
|----------|--------|--------|
| Panel APIs | Session cookie | `Cookie: cc.session=...` |
| Panel APIs (programmatic) | Bearer token | `Authorization: Bearer <token>` |
| Inbound data push | App credentials | `x-app-id` + `x-app-token` |

**Get Bearer token:** Manage Account → Token (Brand Admin only)

---

## Roles

| Role | Key Permissions |
|------|----------------|
| **Owner** | Full access, one per org |
| **Admin** | Full access, manages members/tokens |
| **Supervisor** | Tasks, taxonomy, SPD + PPD |
| **Content Enricher** | Enrichment tasks only |
| **Auditor** | Audit tasks, can publish to production |

---

## Deep Reference

If `fynd-partner-mcp` MCP is connected:
```
fynd_get_docs("ai-pim", "connectors")     — full inbound/outbound setup
fynd_get_docs("ai-pim", "api-reference")  — all 50 API operations
fynd_get_docs("ai-pim", "taxonomy")       — categories, attributes, templates
fynd_search("your query")
```
