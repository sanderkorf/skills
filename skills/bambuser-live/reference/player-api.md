# Player API (web)

Cross-cutting web Player API map. Domain payloads live in sibling refs — this file indexes configure keys, events, and methods.

**Canonical docs:** [Player API reference](https://bambuser.com/docs/live/player-api-reference/)

**Domain refs:** [cart-events](cart-events.md) · [product-hydration](product-hydration.md) · [wishlist](wishlist.md) · [miniplayer](miniplayer.md)

## Ready hook

```js
window.onBambuserLiveShoppingReady = (player) => {
  // player + constants/methods only valid here
}
```

Define the handler **before** `initBambuserLiveShopping` / embed script. One handler per page.

## Constants

### `player.BUTTON`

| Constant | Behavior |
|----------|----------|
| `AUTO` | Default (context-dependent) |
| `CLOSE` | Close overlay |
| `LINK` | Open URL in new tab (web) |
| `MINIMIZE` | Minimize (miniplayer) |
| `NONE` | No-op; may hide |
| `EVENT` | Custom event for host |

Recipes: [miniplayer](miniplayer.md).

### `player.MINIMIZED_POSITION`

`BOTTOM_RIGHT` (default) · `BOTTOM_LEFT` · `TOP_LEFT` · `TOP_RIGHT`

### `player.FLOATING_PLAYER_NAVIGATION_MODE`

`MANUAL` — SPA nav-behind without surf iframe. See [miniplayer](miniplayer.md).

## `player.configure(configuration)`

Call inside the ready hook. Cart needs `currency` + `locale`.

| Key | Type / notes |
|-----|----------------|
| `locale` | `languageCode-countryCode` (e.g. `en-US`). Must exist under BamHub **Settings → Translations** or falls back. Cache ~30 min for translation updates |
| `currency` | ISO 4217 three-letter (e.g. `USD`) — products/cart/checkout default |
| `autoplay` | `boolean` (default on). Autoplay may force muted where browsers require it |
| `externalTitle` | iframe `title` (sanitized, max 200). Default `"Bambuser Live Shopping Player"` |
| `innerTitle` | document title inside iframe (default = `externalTitle`) |
| `buttons` | `{ dismiss?, checkout?, product? }` → `player.BUTTON.*`. See [miniplayer](miniplayer.md) |
| `audioTrackLocale` | Default dubbed audio locale (e.g. `de-DE`) |
| `shareTargets` | Override share list (`linkedin`, `twitter`, `whatsapp`, `email`, `facebook`, …) |
| `minimizedPosition` | `player.MINIMIZED_POSITION.*` |
| `checkoutOnCartClick` | `true` → cart chrome emits `CHECKOUT` instead of opening in-player cart |
| `shareBaseUrl` | Absolute base for Share / AddToCalendar links; empty → `window.location.href` |
| `trackingTags` | `[{ key, value }]` sent once at session start via on-configuration |
| `allowShareAutoplay` | Default `true`; `false` omits `?autoplayLiveshopping=` on share URLs |
| `enableFirstPartyCookies` | Default `true`; set `false` only for visitors who declined cookies (hurts conversion tracking) |
| `cookie` | `{ domain?, activityCookieTTLDays? }` for first-party conversion cookies |
| `trimPriceTrailingZeros` | `boolean` — drop `.00` decimals |
| `deeplink` | Product appearance seek token (from appearances API); also passable on `initBambuserLiveShopping` |
| `ui` | Hide flags (below) |
| `overrideSafeAreaInsets` | `{ top?, right?, bottom?, left? }` px — WebView / gesture clearance |
| `neverCollapseTimelineBarMobile` | Keep timeline bar expanded on mobile |
| `experimental` | Unstable: `{ chatName?, hasAcceptedTerms? }` |
| `floatingPlayer` | `{ navigationMode }` — SPA MANUAL mode; [miniplayer](miniplayer.md) |

### `configuration.ui` hide flags

`hideAll` · `hideActionBar` · `hideAddToCalendar` · `hideCartButton` · `hideCartView` · `hideChatButton` · `hideChatOverlay` · `hideEmojiOverlay` · `hideProductList` · `hideProductView` · `hideShareButton` · `hideShareView` · `hideWishlist`

## Events (`player.EVENT.*`)

Register with `player.on(event, handler)`. Domain column = open that ref for payloads/skeletons.

| Event | Trigger | Domain |
|-------|---------|--------|
| `ADD_TO_CART` | ATC tap | [cart-events](cart-events.md) |
| `UPDATE_ITEM_IN_CART` | Cart qty change / remove | [cart-events](cart-events.md) |
| `CHECKOUT` | Checkout from player cart | [cart-events](cart-events.md) |
| `SYNC_CART_STATE` | Return to player — sync in-player cart | [cart-events](cart-events.md) |
| `PROVIDE_PRODUCT_DATA` | Player needs product details | [product-hydration](product-hydration.md) |
| `UPDATE_PRODUCT_HIGHLIGHT` | Highlighted product changed (`products[]` or empty) | host UI / rail |
| `PROVIDE_WISHLIST_STATUS` | Wishlist state request | [wishlist](wishlist.md) |
| `ADD_TO_WISHLIST` | Add heart | [wishlist](wishlist.md) |
| `REMOVE_FROM_WISHLIST` | Remove heart | [wishlist](wishlist.md) |
| `OPEN_WISHLIST` | View wishlist | [wishlist](wishlist.md) |
| `OPEN_WISHLIST_LOGIN` | Login after wishlist gate | [wishlist](wishlist.md) |
| `NAVIGATE_BEHIND_TO` | SPA/manual navigate behind player | [miniplayer](miniplayer.md) |
| `NOTIFY_URL_CHANGE` | Miniplayer iframe URL navigation | [miniplayer](miniplayer.md) |
| `MINIMIZE` | Player minimized | [miniplayer](miniplayer.md) |
| `ENTERED_PICTURE_IN_PICTURE` | PiP entered | [miniplayer](miniplayer.md) |
| `EXITED_PICTURE_IN_PICTURE` | PiP exited (`stopPlaybackIntent`) | [miniplayer](miniplayer.md) |
| `LOAD` | Player app loaded | — |
| `READY` | GUI ready for interaction | — |
| `CLOSE` | Player closed | — |
| `LOAD_ERROR` | Show failed to load (`{ status: 404 }` today) | — |
| `MUTED` / `UNMUTED` | Mute state | — |
| `PLAYBACK_STATUS` | Playback state change | — |
| `PLAYER_CONTAINER_UPDATE` | Container updated | — |
| `SHOW_CART` / `HIDE_CART` | Cart view | — |
| `SHOW_PRODUCT_LIST` / `HIDE_PRODUCT_LIST` | Product list | — |
| `SHOW_PRODUCT_VIEW` / `HIDE_PRODUCT_VIEW` | Product PDP in player | — |
| `SHOW_CHAT_OVERLAY` / `HIDE_CHAT_OVERLAY` | Chat overlay | — |
| `CHAT_MESSAGES` | New chat messages | — |
| `SHOW_EMOJI_BATCH` | Emoji batch | — |
| `SHOW_SHARE` | Share dialog | — |
| `SHOW_ADD_TO_CALENDAR` | Add-to-calendar | — |
| `OPEN_URL` | User navigated to a URL from player | — |
| `ACTION_CARD_CLICKED` | Action card click | — |
| `TOP_UI_ELEMENT_SHOWN` / `TOP_UI_ELEMENT_HIDDEN` | Top notification bar | — |
| `PLAYER_SWIPE_UP` / `DOWN` / `LEFT` / `RIGHT` | Swipe gestures | — |
| `CAPTIONS_SHOWN` / `CAPTIONS_HIDDEN` / `CAPTION_TRACK_CHANGED` | Closed captions | — |

### `UPDATE_PRODUCT_HIGHLIGHT` payload

`{ products: [product] }` when a product is highlighted; `{ products: [] }` when cleared or highlight is not a product. Same public product shape as other product events.

## Methods

| Method | Role | Domain |
|--------|------|--------|
| `configure(config)` | Player options (table above) | — |
| `on` / `off` / `removeAllListeners` | Event subscriptions | — |
| `close()` | Close and remove player | — |
| `minimize()` / `unminimize()` | Miniplayer | [miniplayer](miniplayer.md) |
| `requestPictureInPicture()` / `exitPictureInPicture()` | Browser PiP | [miniplayer](miniplayer.md) |
| `hideUI()` / `showUI()` | Chrome visibility (playback stays) | — |
| `play()` / `pause()` / `mute()` / `unmute()` | Playback | — |
| `updateSafeAreaInsets(insets)` | Runtime safe-area override | — |
| `showCheckout(url)` | Checkout URL (often from `CHECKOUT`) | [cart-events](cart-events.md) |
| `updateCart(cartData)` | Push in-player cart (typically empty) | [cart-events](cart-events.md) |
| `updateProduct(id, factory)` | Hydrate product | [product-hydration](product-hydration.md) |
| `updateWishlistStatus(status)` | Heart/state map | [wishlist](wishlist.md) |
| `setTrackingTags(tags)` | Session tracking tags (once) | — |
| `setRootFontSize(size)` | Root font px / `'16px'` | — |
| `patchTheme(overrides)` | Deep-merge theme (e.g. caption colors) | — |
| `showCaptions()` / `hideCaptions()` | Captions on/off (Promise) | — |
| `selectCaptionTrack(languageCode)` | BCP-47 track | — |
| `getAvailableCaptions()` | `{ availableCaptions, currentTrack, isEnabled }` | — |
| `enableHighContrast()` / `disableHighContrast()` | Forced high-contrast styling | — |

Cart ATC / update callbacks also accept object forms — see [cart-events](cart-events.md). Wishlist callbacks — see [wishlist](wishlist.md).
