---
name: bambuser-live
description: >
  Bambuser Live Shopping storefront embed — inline/overlay player, cart bridge,
  product hydration via PROVIDE_PRODUCT_DATA / updateProduct, host-repo UI and
  cart adapters. Use when embedding bam-inline-player or embed.js, wiring
  onBambuserLiveShoppingReady, integrating native cart with Bambuser player
  events, or composing a liveshow page with the project's own components.
  Distinct from bambuser-api (server REST) and BamHub product catalog feeds.
---

# Bambuser Live (embed + cart)

Client-side Live Shopping player and cart bridge. Wire Bambuser into a host repo that already owns UI, product lookup, and cart — do not invent a parallel shop UI inside the Bambuser layer.

**References:** [cart-events](reference/cart-events.md) · [product-hydration](reference/product-hydration.md)

**Docs:** [Inline player](https://bambuser.com/docs/live/inline-player-setup/) · [Cart integration](https://bambuser.com/docs/live/cart-integration/) · [Player API](https://bambuser.com/docs/live/player-api-reference/)

## Integration boundaries

| Surface | Role |
|---------|------|
| **Embed / player SDK** | This skill — storefront player, cart events, product hydration |
| Live **REST** API | Server-side shows/webhooks — use **bambuser-api** |
| Product **catalog feed** | BamHub ingest (XML/CSV/SFTP); can replace `PROVIDE_PRODUCT_DATA` but cart events still required |
| App Framework | BamHub iframe apps — out of scope |
| Channels library embed | Out of scope beyond knowing it exists |

## Workflow

1. Copy embed script URL + show id from BamHub → show Setup (match Global vs EU region).
2. Register **one** `window.onBambuserLiveShoppingReady` **before** the embed script loads.
3. Mount player: `<bam-inline-player show-id="…">` and/or overlay via `initBambuserLiveShopping`.
4. Implement host **adapters** (product lookup + cart ops + optional host cart UI).
5. On `PROVIDE_PRODUCT_DATA`, hydrate with `player.updateProduct` (see [product-hydration](reference/product-hydration.md)).
6. Wire required cart events (see [cart-events](reference/cart-events.md)).
7. Compose host UI beside the player (rail, PDP dialog, CartSheet) using the same product/cart data — Bambuser does not own those components.

## Embed

| Region | BamHub host | Default embed script |
|--------|-------------|----------------------|
| Global | `lcx.bambuser.com` | `https://lcx-embed.bambuser.com/default/embed.js` |
| Europe | `lcx-eu.bambuser.com` | `https://lcx-embed-eu.bambuser.com/default/embed.js` |

Prefer the **brand-specific** script URL from show Setup over the generic `default` URL when BamHub provides one. Use the official Bambuser-hosted script only — no self-hosted copy.

### Inline player

```html
<style>
  bam-inline-player {
    min-width: 320px;
    min-height: 320px;
    /* Size the container for your layout / show aspect ratio */
  }
</style>
<bam-inline-player show-id="SHOW_ID_HERE"></bam-inline-player>
```

Mini-player is **not** supported for inline. Configure via `onBambuserLiveShoppingReady` the same as overlay.

### Overlay (button trigger)

```js
window.initBambuserLiveShopping({
  showId: 'SHOW_ID_HERE',
  node: document.getElementById('join-show'),
  type: 'overlay',
})
```

### Ready hook (before embed)

```js
window.onBambuserLiveShoppingReady = (player) => {
  player.configure({
    currency: 'EUR', // required for cart
    locale: 'en-US', // required for cart
  })
  // Register cart + product handlers here — see reference/
}
```

## Adapter contract

Map host APIs onto these seams. Keep Bambuser-specific code thin; cart/product logic stays in the project.

```ts
type HostProduct = {
  name: string
  brand?: string
  sku: string // product-level ref; size-level skus used for ATC
  introduction?: string
  description?: string // text or HTML
  defaultVariationIndex?: number
  variations: Array<{
    name: string
    sku: string
    colorName?: string
    colorHexCode?: string
    imageUrls: string[]
    sizes: Array<{
      name: string
      sku: string // ATC identity
      inStock: boolean
      currency?: string
      current: number
      original?: number
    }>
  }>
}

type BambuserLiveAdapters = {
  getProduct: (ref: string, url: string) => Promise<HostProduct>
  addToCart: (sku: string) => Promise<void>
  updateCartItem: (sku: string, quantity: number) => Promise<void>
  checkout: () => void | Promise<void>
  getCartState?: () => Promise<{ items: { sku: string; quantity: number }[] }>
  openHostCart?: () => void // CartSheet / drawer — prefer host UI
}
```

Wire adapters inside `onBambuserLiveShoppingReady`: `getProduct` → `updateProduct`; cart methods → event callbacks. Host UI (timeline rail, variant picker) can share the same `getProduct` / cart mutations without going through the player.

**In-player cart** needs all required events (`PROVIDE_PRODUCT_DATA`, `ADD_TO_CART`, `UPDATE_ITEM_IN_CART`, `CHECKOUT`) or ATC stays disabled. Host-UI-only ATC still benefits from hydration so player product cards stay accurate; implement cart events if in-player ATC is shown.

## Next.js notes

- Call `ensureBambuserReadyHook()` / set `onBambuserLiveShoppingReady` in a client module that loads **before** the embed script (e.g. cart bridge mounts first).
- Load embed with `next/script` (`strategy="afterInteractive"`) from env (e.g. `NEXT_PUBLIC_BAMBUSER_EMBED_SRC`).
- Declare custom element / globals in a `.d.ts` (`bam-inline-player`, `onBambuserLiveShoppingReady`, `initBambuserLiveShopping`).
- If CSP blocks third-party scripts, allowlist `https://lcx-embed.bambuser.com` or `https://lcx-embed-eu.bambuser.com` (match region).

## Rules

- One `window.onBambuserLiveShoppingReady` per page — merge integrations into a single handler.
- All cart/product listeners go **inside** that handler.
- Required cart configs: `locale` + `currency`.
- In-player ATC requires the full required event set (see [cart-events](reference/cart-events.md)).
- Do not call Live REST (`liveshopping-api`) from the browser — that is **bambuser-api**, server-side.
- Do not replace the project's cart UI with Bambuser chrome; bridge events into host cart and open host checkout/cart when possible.
- Match BamHub region (Global vs EU) for embed host.
- Product scrape from PDP URLs is thin (title, thumbnail, brand, ref) — always hydrate for cart-quality data unless Product Feed covers it.
