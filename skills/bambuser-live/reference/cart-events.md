# Cart events

All handlers go inside `window.onBambuserLiveShoppingReady`. In-player cart / ATC stays disabled until these are implemented.

Configure keys, non-cart events, and method index: [player-api](player-api.md). Wishlist is a separate domain: [wishlist](wishlist.md).

| Event | Purpose |
|-------|---------|
| `PROVIDE_PRODUCT_DATA` | Hydrate show products (`updateProduct`) — or use Product Feed instead |
| `ADD_TO_CART` | Native add; `callback(true\|false)` |
| `UPDATE_ITEM_IN_CART` | Qty change / remove (`quantity === 0`); `callback(true\|false)` |
| `CHECKOUT` | Open host checkout / cart |
| `SYNC_CART_STATE` | Align player cart with host (typically empty player cart after host checkout) |

Also set `player.configure({ locale, currency })`.

**Product Feed alternative:** BamHub feed can replace `PROVIDE_PRODUCT_DATA`, but `ADD_TO_CART`, `UPDATE_ITEM_IN_CART`, `CHECKOUT`, and `SYNC_CART_STATE` are still required. Multi-market catalogs and near-real-time stock usually still need `PROVIDE_PRODUCT_DATA`. See [product-hydration](product-hydration.md).

## Skeleton

```js
window.onBambuserLiveShoppingReady = (player) => {
  player.configure({ currency: 'EUR', locale: 'en-US' })

  player.on(player.EVENT.PROVIDE_PRODUCT_DATA, (event) => {
    // → reference/product-hydration.md
  })

  player.on(player.EVENT.ADD_TO_CART, (addedItem, callback) => {
    adapters
      .addToCart(addedItem.sku)
      .then(() => {
        callback(true)
        adapters.openHostCart?.()
      })
      .catch((error) => {
        if (error?.type === 'out-of-stock') {
          callback({ success: false, reason: 'out-of-stock' })
        } else {
          callback(false)
        }
      })
  })

  player.on(player.EVENT.UPDATE_ITEM_IN_CART, (updatedItem, callback) => {
    adapters
      .updateCartItem(updatedItem.sku, updatedItem.quantity)
      .then(() => callback(true))
      .catch(() => callback(false))
  })

  player.on(player.EVENT.CHECKOUT, () => {
    // Prefer host checkout / cart UI
    void adapters.checkout()
    // Or: player.showCheckout(window.location.origin + '/cart')
  })

  player.on(player.EVENT.SYNC_CART_STATE, () => {
    // Player cart starts empty each session; bi-directional sync is limited.
    // Typical: clear player cart after host checkout emptied the native cart.
    void (async () => {
      const state = await adapters.getCartState?.()
      if (!state || state.items.length === 0) {
        player.updateCart({ items: [] })
      }
    })()
  })
}
```

## ADD_TO_CART

| Field | Type | Notes |
|-------|------|--------|
| `addedItem.sku` | string | Size/variation sku from hydration (ATC identity) |
| `addedItem.raw` | object \| undefined | Extra Product Feed fields when feed is used **and** no `PROVIDE_PRODUCT_DATA` handler |
| `callback` | function | `true` / `false` / `{ success, reason, message? }` |

## UPDATE_ITEM_IN_CART

| Field | Type | Notes |
|-------|------|--------|
| `updatedItem.sku` | string | Item identity |
| `updatedItem.quantity` | number | New qty; `0` = remove |
| `updatedItem.previousQuantity` | number | Prior qty |
| `updatedItem.raw` | object \| undefined | Same feed rules as ADD_TO_CART |
| `callback` | function | Same as ADD_TO_CART |

## CHECKOUT

Fires when viewer taps Checkout in the player cart. Route to host cart/checkout (`adapters.checkout` / `openHostCart`) or `player.showCheckout(url)`.

## SYNC_CART_STATE

Fires when viewer returns to the player. Use `player.updateCart({ items })` to push state; emptying (`items: []`) is the common case after native checkout. Full bi-directional sync is not supported — new player instances start with an empty cart.

## Callback errors

```js
callback({
  success: false,
  reason: 'custom-error', // or 'out-of-stock'
  message: 'Optional custom message',
})
```

## Gotchas

- Stock UI is not live for existing viewers — reject sold-out ATC via `callback` with `out-of-stock`.
- Max **two** variation axes in the in-player PDP (e.g. color + size). Flatten a third axis into one of them or skip in-player cart for those SKUs.
- Nested variations (sizes under colors) → two dropdowns; flat list → one combined dropdown.
