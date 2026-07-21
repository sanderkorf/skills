---
name: bambuser-live
description: >
  Bambuser Live web player — storefront embed, cart bridge, product hydration,
  wishlist, miniplayer/SPA nav, player.configure / Player API chrome. Use when
  embedding bam-inline-player or embed.js, wiring onBambuserLiveShoppingReady,
  bridging native cart or wishlist to player events, enabling miniplayer or
  SPA navigate-behind, or calling player.configure / Player API methods.
  Distinct from bambuser-api (server REST) and BamHub product catalog feeds.
  Web only — not Swift/Kotlin Commerce SDK.
---

# Bambuser Live (web embed)

Client-side Live Shopping player. Wire Bambuser into a host repo that already owns UI, product lookup, cart, and wishlist — keep Bambuser-specific code thin.

**References:** [cart-events](reference/cart-events.md) · [product-hydration](reference/product-hydration.md) · [wishlist](reference/wishlist.md) · [miniplayer](reference/miniplayer.md) · [player-api](reference/player-api.md)

**Docs:** [Player API](https://bambuser.com/docs/live/player-api-reference/) · [Cart](https://bambuser.com/docs/live/cart-integration/) · [Wishlist](https://bambuser.com/docs/live/wishlist-integration/) · [Miniplayer](https://bambuser.com/docs/live/miniplayer/) · [Inline player](https://bambuser.com/docs/live/inline-player-setup/)

## Integration boundaries

| Surface | Role |
|---------|------|
| **Web embed / Player API** | This skill — player, cart, hydration, wishlist, miniplayer, configure |
| Live **REST** API | Server-side — use **bambuser-api** |
| Product **catalog feed** | BamHub ingest; can replace `PROVIDE_PRODUCT_DATA` but cart events still required |
| **Mobile** Commerce SDK (Swift/Kotlin) | Out of scope |
| App Framework / Channels library | Out of scope |

## Workflow

Complete each step before claiming that domain done. Open the linked reference when a gate applies — payloads and skeletons live there, not here.

1. **Embed** — Copy brand-specific embed script URL + show id from BamHub → show Setup (match Global vs EU). Register **one** `window.onBambuserLiveShoppingReady` **before** the embed script loads. Mount `<bam-inline-player>` and/or overlay via `initBambuserLiveShopping`. Done when script region matches BamHub and ready hook runs before init.
2. **Adapters** — Declare host seams in `BambuserLiveAdapters` (below). Done when every required domain has a host function mapped.
3. **Cart + hydration** — When in-player ATC or cart is shown: open [product-hydration](reference/product-hydration.md) and [cart-events](reference/cart-events.md); wire `PROVIDE_PRODUCT_DATA` → `updateProduct` and required cart events + `locale`/`currency`. Done when every required cart event has a handler and ATC is enabled (or Product Feed replaces hydration only).
4. **Wishlist** — When BamHub theme has wishlist on or the user asks for it: open [wishlist](reference/wishlist.md); wire status + add/remove + open/login. Done when `updateWishlistStatus` answers `PROVIDE_WISHLIST_STATUS` and add/remove callbacks cover success and `login-required` / `more-info-required`.
5. **Miniplayer / SPA** — When dismiss/checkout/product use minimize, or host is SPA nav-behind: open [miniplayer](reference/miniplayer.md). Done when button config matches intent and SPA `NAVIGATE_BEHIND_TO` (if MANUAL mode) routes without full reload.
6. **Configure / chrome** — When touching buttons, UI hide flags, cookies, tracking, theme, captions, playback, or other Player API surface: open [player-api](reference/player-api.md). Done when every touched key/method is applied inside the ready hook.
7. **Host UI** — Compose rail, PDP, CartSheet beside the player with the same product/cart/wishlist data. Bambuser does not own those components.

## Embed

| Region | BamHub host | Default embed script |
|--------|-------------|----------------------|
| Global | `lcx.bambuser.com` | `https://lcx-embed.bambuser.com/default/embed.js` |
| Europe | `lcx-eu.bambuser.com` | `https://lcx-embed-eu.bambuser.com/default/embed.js` |

Prefer the **brand-specific** script URL from show Setup over the generic `default` URL when BamHub provides one. Use the official Bambuser-hosted script only.

### Inline player

```html
<style>
  bam-inline-player {
    min-width: 320px;
    min-height: 320px;
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
  // Domain handlers — see reference/
}
```

`player` and its methods exist only inside this handler.

## Adapter contract

Map host APIs onto these seams. Cart/product/wishlist logic stays in the project.

```ts
type HostProduct = {
  name: string
  brand?: string
  sku: string // product-level ref; size-level skus used for ATC / wishlist
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
      sku: string // ATC / wishlist identity
      inStock: boolean
      currency?: string
      current: number
      original?: number
      perUnit?: number
      unitAmount?: number
      unitDisplayName?: string
    }>
  }>
}

type BambuserLiveAdapters = {
  getProduct: (ref: string, url: string) => Promise<HostProduct>
  addToCart: (sku: string) => Promise<void>
  updateCartItem: (sku: string, quantity: number) => Promise<void>
  checkout: () => void | Promise<void>
  getCartState?: () => Promise<{ items: { sku: string; quantity: number }[] }>
  openHostCart?: () => void
  getWishlistStatuses?: (skus: string[]) => Promise<Record<string, boolean>>
  addToWishlist?: (sku: string) => Promise<void>
  removeFromWishlist?: (sku: string) => Promise<void>
  openWishlist?: () => void | Promise<void>
  openWishlistLogin?: () => void | Promise<void>
}
```

Wire adapters inside `onBambuserLiveShoppingReady`. Host UI can share the same `getProduct` / cart / wishlist mutations without going through the player.

**In-player cart** needs the required cart event set or ATC stays disabled. Host-UI-only ATC still benefits from hydration; implement cart events if in-player ATC is shown.

## Next.js notes

- Set `onBambuserLiveShoppingReady` in a client module that loads **before** the embed script.
- Load embed with `next/script` (`strategy="afterInteractive"`) from env (e.g. `NEXT_PUBLIC_BAMBUSER_EMBED_SRC`).
- Declare custom element / globals in a `.d.ts`.
- CSP: allowlist `https://lcx-embed.bambuser.com` or `https://lcx-embed-eu.bambuser.com` (match region).

## Rules

- One `window.onBambuserLiveShoppingReady` per page — merge integrations into a single handler.
- All player listeners and `configure` calls go **inside** that handler.
- Cart configs: `locale` + `currency`.
- In-player ATC → full required cart event set ([cart-events](reference/cart-events.md)).
- Live REST (`liveshopping-api`) stays server-side (**bambuser-api**).
- Bridge into host cart/wishlist/checkout; prefer host UI over Bambuser chrome when the project already has those surfaces.
- Match BamHub region (Global vs EU) for embed host.
- Hydrate for cart-quality data unless Product Feed covers it — scrape alone is thin.
