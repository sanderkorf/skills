# Wishlist

Bridge in-player wishlist to the host store. Enable in BamHub theme editor: **Player settings → Product CTAs → Enable Wishlist** (optional: show wishlist on list items). Integration methods below are required for correct heart/state UI.

**Docs:** [Wishlist integration](https://bambuser.com/docs/live/wishlist-integration/) · event constants in [player-api](player-api.md)

All handlers go inside `window.onBambuserLiveShoppingReady`.

## Events

| Event | Purpose |
|-------|---------|
| `PROVIDE_WISHLIST_STATUS` | Player asks wishlist state for products (startup after product list fetch) → reply with `updateWishlistStatus` |
| `ADD_TO_WISHLIST` | User taps add; `(event, callback)` |
| `REMOVE_FROM_WISHLIST` | User taps remove; `(event, callback)` |
| `OPEN_WISHLIST` | User taps View wishlist → open host wishlist |
| `OPEN_WISHLIST_LOGIN` | User taps Login after `login-required` callback → open host login |

## `player.updateWishlistStatus`

```ts
player.updateWishlistStatus({
  statuses: {
    'sku-123': true,
    'sku-456': false,
  },
})
```

Keys are product/variant `sku` strings. Call in response to `PROVIDE_WISHLIST_STATUS` and anytime host wishlist changes while the player is open.

## ADD / REMOVE payload

| Field | Type | Notes |
|-------|------|--------|
| `sku` | string | Variant identity (same family as cart ATC sku) |
| `raw` | object \| undefined | Extra Product Feed fields when feed is used |
| `callback` | function | See below |

### Callback shape

```js
callback({
  success: true,
  sku,
  reason: null,
})

callback({
  success: false,
  sku,
  reason: 'login-required', // shows Login → OPEN_WISHLIST_LOGIN
})

callback({
  success: false,
  sku,
  reason: 'more-info-required', // opens product details (e.g. pick size/color first)
})
```

## Skeleton

```js
window.onBambuserLiveShoppingReady = (player) => {
  player.on(player.EVENT.PROVIDE_WISHLIST_STATUS, (event) => {
    const skus = (event.products ?? []).map((p) => p.sku)
    void adapters.getWishlistStatuses?.(skus).then((statuses) => {
      player.updateWishlistStatus({ statuses: statuses ?? {} })
    })
  })

  player.on(player.EVENT.ADD_TO_WISHLIST, (event, callback) => {
    const { sku } = event
    // if (!loggedIn) return callback({ success: false, sku, reason: 'login-required' })
    // if (needsVariant) return callback({ success: false, sku, reason: 'more-info-required' })
    adapters
      .addToWishlist?.(sku)
      .then(() => callback({ success: true, sku, reason: null }))
      .catch(() => callback({ success: false, sku, reason: null }))
  })

  player.on(player.EVENT.REMOVE_FROM_WISHLIST, (event, callback) => {
    const { sku } = event
    adapters
      .removeFromWishlist?.(sku)
      .then(() => callback({ success: true, sku, reason: null }))
      .catch(() => callback({ success: false, sku, reason: null }))
  })

  player.on(player.EVENT.OPEN_WISHLIST, () => {
    void adapters.openWishlist?.()
  })

  player.on(player.EVENT.OPEN_WISHLIST_LOGIN, () => {
    void adapters.openWishlistLogin?.()
  })
}
```

## Gotchas

- Theme toggle alone is not enough — without handlers, hearts/state stay wrong.
- Hide wishlist chrome via `player.configure({ ui: { hideWishlist: true } })` when the domain is unused ([player-api](player-api.md)).
- Sku identity should match hydration size/variant skus used for cart.
