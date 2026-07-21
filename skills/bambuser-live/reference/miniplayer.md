# Miniplayer / SPA nav

Overlay miniplayer lets viewers browse the site while the show keeps playing. **Not supported on `<bam-inline-player>`** — overlay only.

**Docs:** [Miniplayer](https://bambuser.com/docs/live/miniplayer/) · constants/methods in [player-api](player-api.md)

Register configure + handlers inside `window.onBambuserLiveShoppingReady` before the embed script loads.

## Enable / disable

Miniplayer turns on when any button uses `player.BUTTON.MINIMIZE`.

| Goal | Config |
|------|--------|
| Enable (dismiss → minimize) | `buttons.dismiss: player.BUTTON.MINIMIZE` |
| Minimize on checkout | `buttons.checkout: player.BUTTON.MINIMIZE` (needs cart integration) |
| Minimize on product click | `buttons.product: player.BUTTON.MINIMIZE` (disables in-player cart path for that click) |
| Disable miniplayer | `buttons.dismiss: player.BUTTON.CLOSE` and no button set to `MINIMIZE` |

### BUTTON behaviors (web)

| Constant | Behavior |
|----------|----------|
| `player.BUTTON.AUTO` | Default (context-dependent) |
| `player.BUTTON.CLOSE` | Close overlay |
| `player.BUTTON.LINK` | Open product/checkout URL in new tab (web) |
| `player.BUTTON.MINIMIZE` | Minimize to floating player |
| `player.BUTTON.NONE` | No-op; button may hide |
| `player.BUTTON.EVENT` | Fire custom event for host handler |

Configurable buttons: `dismiss`, `checkout`, `product` — see [player-api](player-api.md) `configuration.buttons`.

### Position

```js
player.configure({
  minimizedPosition: player.MINIMIZED_POSITION.BOTTOM_RIGHT, // default
  // BOTTOM_LEFT | TOP_LEFT | TOP_RIGHT
})
```

Viewer can drag after load.

## Default (iframe) vs SPA (manual)

**Default:** minimizing loads the site inside a surf iframe under a `div[id^=livecommerce-surf]`. Site must allow framing; avoid `position: fixed` on `<html>`. Top context (launch page) ≠ iframe context (separate `window`/`document`).

**SPA exemption:** keep the player in the top document and route yourself:

```js
player.configure({
  buttons: {
    dismiss: player.BUTTON.MINIMIZE,
    checkout: player.BUTTON.MINIMIZE,
  },
  floatingPlayer: {
    navigationMode: player.FLOATING_PLAYER_NAVIGATION_MODE.MANUAL,
  },
})

player.on(player.EVENT.NAVIGATE_BEHIND_TO, (event) => {
  // event.url — PDP from workspace, or URL passed to showCheckout
  // Client-route without full reload (router / history.pushState)
})
```

`NAVIGATE_BEHIND_TO` fires for product navigation and `player.showCheckout(url)`.

Also useful: `player.EVENT.NOTIFY_URL_CHANGE` when miniplayer initiates URL navigation inside the iframe overlay.

## Methods + PiP

| Method | Role |
|--------|------|
| `player.minimize()` | Programmatic minimize |
| `player.unminimize()` | Restore from minimized |
| `player.requestPictureInPicture()` | Browser PiP — call from user gesture; emits `ENTERED_PICTURE_IN_PICTURE` |
| `player.exitPictureInPicture()` | Leave PiP; emits `EXITED_PICTURE_IN_PICTURE` |

`EXITED_PICTURE_IN_PICTURE` payload: `{ stopPlaybackIntent: boolean }`.

- `true` — recent pause before exit (~1.5s); often native PiP close (not guaranteed)
- `false` — likely continue / restore, or programmatic `exitPictureInPicture`

To fully tear down the player after PiP close intent, call `player.close()`. Cross-check nearby `PLAYBACK_STATUS` when refining close vs restore.

## Skeleton

```js
window.onBambuserLiveShoppingReady = (player) => {
  player.configure({
    buttons: {
      dismiss: player.BUTTON.MINIMIZE,
      checkout: player.BUTTON.MINIMIZE,
    },
    minimizedPosition: player.MINIMIZED_POSITION.BOTTOM_RIGHT,
    // SPA only:
    // floatingPlayer: {
    //   navigationMode: player.FLOATING_PLAYER_NAVIGATION_MODE.MANUAL,
    // },
  })

  // SPA only:
  // player.on(player.EVENT.NAVIGATE_BEHIND_TO, ({ url }) => { /* client route */ })

  player.on(player.EVENT.EXITED_PICTURE_IN_PICTURE, ({ stopPlaybackIntent }) => {
    if (stopPlaybackIntent) player.close()
  })
}
```

## Completion

Button config matches product intent (minimize vs close vs link). If MANUAL SPA mode: `NAVIGATE_BEHIND_TO` routes without full document reload. Inline embeds leave miniplayer off.
