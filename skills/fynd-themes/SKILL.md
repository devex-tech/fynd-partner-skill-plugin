---
name: fynd-themes
description: >-
  Fynd Commerce Themes expert. Use when a partner asks about building,
  customizing, or submitting React-based Fynd Commerce themes — FDK-CLI setup,
  theme structure, ServerFetch (SSR), AuthGuard, FPI client, useGlobalStore,
  fdk-store, sections/blocks/canvas, color palette CSS variables, Tailwind CSS
  integration, headless themes, i18n, code splitting, marketplace submission,
  or theme best practices. Triggers on: "theme", "FDK-CLI", "fdk theme",
  "ServerFetch", "AuthGuard", "FPI", "fdk-store", "useGlobalStore", "useFPI",
  "sections", "blocks", "canvas", "settings_data.json", "theme editor",
  "Tailwind", "SSR theme", "storefront theme", "submit theme".
argument-hint: "[topic] e.g. 'how do I use ServerFetch' or 'submit theme to marketplace'"
---

# Fynd Commerce Themes Skill

## What Are Fynd Themes?

React v18-based storefronts that define the look, feel, and user interactions of a Fynd Commerce seller's store. Partners can customize existing themes, build bespoke themes, or sell themes on the Fynd Partners Marketplace.

---

## Route by Topic

| Question about... | Key content below |
|-------------------|------------------|
| Getting started, FDK-CLI | **Setup** section |
| Directory structure, pages, sections, blocks | **Structure** section |
| ServerFetch, AuthGuard, FPI client | **Development** section |
| fdk-store, hooks, mutations, resolvers | **Data Management** section |
| Tailwind, headless, locale, code splitting | **Advanced** section |
| Submitting to marketplace, rejection reasons | **Submission** section |
| Best practices checklist | **Best Practices** section |

---

## Setup

```bash
npm install -g @gofynd/fdk-cli
fdk login
fdk theme new --name my-theme   # generates boilerplate
cd my-theme
fdk theme serve                  # localhost:5001
fdk theme sync                   # push to Fynd Commerce
fdk theme open                   # verify in browser
fdk theme package                # create zip for submission
```

---

## Structure

```
theme-name/
├── themes/
│   ├── pages/          ← system pages (home, pdp, plp, cart, checkout...)
│   ├── sections/       ← reusable section components
│   ├── custom-templates/ ← pages at /c/page-name routes
│   ├── config/
│   │   ├── settings_data.json    ← saved setting values
│   │   └── settings_schema.json  ← setting definitions
│   └── index.jsx       ← entry file — bootstraps theme bundle
└── webpack.config.js
```

### index.jsx Bootstrap
```jsx
export default async ({ applicationID, applicationToken, domain, storeInitialData }) => {
  const { client: fpi } = new FPIClient({ applicationID, applicationToken, domain, storeInitialData });
  return {
    fpi,
    sections,
    getHeader: () => Header,
    getFooter: () => Footer,
    // Pages — use dynamic import for code splitting
    getProductListing: () => import(/* webpackChunkName:"getProductListing" */ "./pages/product-listing"),
    globalDataResolver,
    pageDataResolver,
    getGlobalProvider: () => GlobalProvider,
  };
};
```

---

## Development

### ServerFetch — SSR Data Fetching
Runs on the server before HTML is rendered. Use for PDP, PLP, SEO-critical pages.

```jsx
ProductListing.serverFetch = async ({ fpi, router, cookies, themeId }) => {
  // FPI calls automatically include themeCookie — no need to pass it explicitly
  await fpi.executeGQL(PRODUCT_LISTING_QUERY, {
    slug: router.params?.slug,
    pageNo: router.filterQuery?.page_no ?? 1,
  });

  // Use cookies.themeCookie only for custom/non-FPI APIs
  const themeMode = cookies?.themeCookie || 'default';
  return Promise.resolve();
};
```

**Combined SSR + CSR:**
```jsx
// SSR: server fetches data first
ProductListing.serverFetch = async ({ fpi, router }) => {
  return fpi.products.fetchProductListing({ pageNo: 1, pageSize: 12 });
};

// CSR: fallback if SSR didn't run
const ProductListing = ({ fpi }) => {
  const products = useGlobalStore(fpi.getters.PRODUCTS) || {};
  useEffect(() => {
    if (!Object.keys(products).length) fpi.products.fetchProducts({});
  }, [products]);
  return <>{/* render */}</>;
};
```

### AuthGuard — Route Protection
```jsx
// Redirect logged-in users away from /login or /register
const loginGuard = async ({ fpi, store, redirectUrl }) => {
  const loggedIn = await isLoggedIn({ fpi, store });
  if (loggedIn && isRunningOnClient()) window.location.navigate(redirectUrl ?? "/");
};
LoginPage.authGuard = loginGuard;

// Restrict page to logged-in users only
const requireLogin = async ({ fpi, store }) => {
  const { payload } = await fpi.auth.fetchUserData();
  return !!(payload?.user ?? false);
};
CartPage.authGuard = requireLogin;
```

---

## Data Management

### Core Hooks (`fdk-core/utils`)
```jsx
import { useFPI, useGlobalStore, useClientInfo, getPageSlug,
         convertActionToUrl, convertUrlToAction } from "fdk-core/utils";

// Get FPI instance
const fpi = useFPI();

// Subscribe to Redux store slice (re-renders on change)
const page = useGlobalStore(fpi.getters.PAGE) || {};
const products = useGlobalStore(fpi.getters.PRODUCTS) || {};

// Client info (SSR-safe)
const { themeCookie, userAgent } = useClientInfo();

// URL ↔ action object conversion
convertActionToUrl({ type: "page", page: { type: "locate-us" } }); // → "/locate-us"
```

### Custom Store Values
```jsx
fpi.custom.setValue('key', 'value');
const custom = useGlobalStore(fpi.getters.CUSTOM_VALUE); // → { key: 'value' }
```

### Resolvers (exported from index.jsx)
```jsx
// Called once on initial app load
async function globalDataResolver({ fpi, applicationID }) {
  return Promise.all([
    fpi.configuration.fetchApplication(),
    fpi.content.fetchLandingPage(),
    fpi.content.fetchAppSeo(),
  ]);
}

// Called on every route change
async function pageDataResolver({ fpi, router, themeId }) {
  const pageValue = getPageSlug(router);
  const currentPage = fpi.store.getState()?.theme?.page?.value;
  if (pageValue !== currentPage) {
    await fpi.theme.fetchPage({ pageValue, themeId });
  }
}
```

### Host Components (`fdk-core/components`)
```jsx
import { FDKLink, SectionRenderer, HTMLContent } from "fdk-core/components";

// Internal navigation (use instead of <a>)
<FDKLink to="/products">Shop Now</FDKLink>

// Render page sections
<SectionRenderer sections={useGlobalStore(fpi.getters.PAGE)?.sections || []} />
```

---

## Color Palette CSS Variables

```css
/* General */
--primaryColor, --textHeading, --textBody, --textLabel, --textSecondary
--buttonPrimary, --buttonSecondary, --buttonLink
--bgColor, --pageBackground, --accentColor, --linkColor

/* Sale */
--saleBadgeBackground, --saleBadgeText, --saleDiscountText, --saleTimer

/* Header/Footer */
--headerBackground, --headerNav, --headerIcon
--footerBackground, --footerHeadingText, --footerBodyText, --footerIcon

/* Alerts */
--successBackground, --successText, --errorBackground, --errorText
--informationBackground, --informationText, --dialogBackground, --overlay
```

Usage: `color: var(--textHeading, #26201a);`

---

## Advanced

### Tailwind CSS
```bash
npm install tailwindcss postcss autoprefixer && npx tailwindcss init
```
Configure `content: ["./themes/**/*.{js,jsx}"]` in `tailwind.config.js`.

### Code Splitting
```jsx
// In index.jsx — each page is a separate Webpack chunk
getProductListing: () => import(/* webpackChunkName:"getProductListing" */ "./pages/product-listing"),
```
⚠️ `webpackChunkName` must match the key exactly.

### Sync Between Themes
```bash
fdk theme context -n my-context   # in source theme directory
fdk theme sync                     # pushes to target theme
```
⚠️ React themes can only sync with React themes.

---

## Submission (7 Steps)

1. `fdk theme package` → creates `.zip` file
2. Partners Panel → Themes → Submit Theme → upload zip + release notes
3. Theme Details (title, tagline, description, screenshots)
4. Value Proposition (industries, catalog size, price)
5. Attributes (section titles, key highlights, features)
6. Variations (layout screenshots, demo URL)
7. Documentation & Support (docs URL, support email) → Submit

**Common rejection reasons:** Missing essential Fynd features, hardcoded URLs, SSR bugs, accessibility failures, unlicensed demo content, incomplete listing.

---

## Best Practices Checklist

- [ ] No `window`/`document` without browser check (SSR-safe)
- [ ] No hardcoded URLs — use `convertActionToUrl()`
- [ ] `serverFetch` on all SEO-critical pages (PLP, PDP)
- [ ] Images use `transformImage` utility for responsive loading
- [ ] `useCallback` instead of anonymous functions in JSX
- [ ] Unique `key` props on all list items
- [ ] Semantic HTML + ARIA attributes (min contrast ratio 4:1)
- [ ] `fdk theme sync` runs cleanly before submission

---

## Deep Reference

If `fynd-partner-mcp` MCP is connected:
```
fynd_get_docs("themes", "development")      — ServerFetch + AuthGuard + color palette
fynd_get_docs("themes", "data-management")  — hooks, mutations, resolvers, components
fynd_get_docs("themes", "submission")       — full 7-step submission + rejection checklist
fynd_search("your query")
```
