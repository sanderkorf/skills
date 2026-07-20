# Bambuser Live REST — endpoints

Distilled from Bambuser OpenAPI **REST API for Live** v1. Assign these scopes on the BamHub API key.

Base: `https://liveshopping-api-eu.bambuser.com/v1` (EU) or `https://liveshopping-api.bambuser.com/v1` (Global).

Scope `—` means the OpenAPI description does not list `**Required Scope:**` (often beta chat/pinned-comment ops).

## Auth

| Method | Path | Summary | Scope |
|--------|------|---------|-------|
| POST | `/signin/email` | Send sign in link to email | `WRITE_AUTH` |

## Channels

| Method | Path | Summary | Scope |
|--------|------|---------|-------|
| POST | `/channels` | Create a channel | `WRITE_CHANNELS` |
| GET | `/channels/{channelId}` | Get channel details | `READ_CHANNELS` |

## Products

| Method | Path | Summary | Scope |
|--------|------|---------|-------|
| GET | `/products` | List of all products | `READ_PRODUCTS` |
| GET | `/products/highlighted` | List of highlighted products | `READ_PRODUCTS` |
| GET | `/products/highlighted/shows` | List of appearances by product reference | `READ_SHOWS` |
| GET | `/products/highlighted/shows/by-references` | List of appearances by one or more product references | `READ_SHOWS` |
| GET | `/products/{id}/highlighted/shows` | List of appearances by id | `READ_SHOWS` |
| DELETE | `/products/{productId}` | Remove a product | `DELETE_PRODUCTS` |
| GET | `/products/{productId}` | Get a single product | `READ_PRODUCTS` |
| PATCH | `/products/{productId}` | Updates a product | `WRITE_PRODUCTS` |

## Shows

| Method | Path | Summary | Scope |
|--------|------|---------|-------|
| GET | `/shows` | List shows | `READ_SHOWS` |
| POST | `/shows` | Create a new show | `WRITE_SHOWS` |
| DELETE | `/shows/{showId}` | Remove a show and associated data | `DELETE_SHOWS` |
| GET | `/shows/{showId}` | Get show details | `READ_SHOWS` |
| PATCH | `/shows/{showId}` | Update a show | `WRITE_SHOWS` |
| GET | `/shows/{showId}/assets` | List show assets | `READ_SHOWS` |
| POST | `/shows/{showId}/assets` | Create a new show asset | `WRITE_SHOWS` |
| DELETE | `/shows/{showId}/assets/{assetId}` | Delete a show asset | `WRITE_SHOWS` |
| GET | `/shows/{showId}/broadcasts` | Get the broadcasts of a show | `READ_BROADCASTS` |
| POST | `/shows/{showId}/channels` | Add a channel to a show | `WRITE_SHOWS` |
| PUT | `/shows/{showId}/channels` | Update channels in a show | `WRITE_SHOWS` |
| DELETE | `/shows/{showId}/channels/{channelId}` | Remove a channel from a show | `WRITE_SHOWS` |
| GET | `/shows/{showId}/chat-messages` | Get chat messages for a show | `—` |
| POST | `/shows/{showId}/chat-messages` | Send a chat message | `—` |
| PATCH | `/shows/{showId}/chat-messages/{chatMessageId}` | Update a chat message | `—` |
| GET | `/shows/{showId}/chat-transcripts` | Get chat transcripts for a show | `READ_SHOWS` |
| GET | `/shows/{showId}/examples` | Get examples for a show | `READ_SHOWS` |
| GET | `/shows/{showId}/highlights` | Get highlights of a show | `READ_SHOWS` |
| POST | `/shows/{showId}/highlights` | Add a highlight to a show | `WRITE_SHOWS` |
| DELETE | `/shows/{showId}/highlights/{highlightId}` | Remove highlight from a show | `WRITE_SHOWS` |
| PATCH | `/shows/{showId}/highlights/{highlightId}` | Update highlights in a show | `WRITE_SHOWS` |
| GET | `/shows/{showId}/pinned-comments` | Get pinned comments for a show | `—` |
| POST | `/shows/{showId}/pinned-comments` | Create a pinned comment | `—` |
| DELETE | `/shows/{showId}/pinned-comments/{pinnedCommentId}` | Remove a pinned comment | `—` |
| PATCH | `/shows/{showId}/pinned-comments/{pinnedCommentId}` | Update a pinned comment | `—` |
| GET | `/shows/{showId}/products` | Get the products of a show | `READ_PRODUCTS` |
| PATCH | `/shows/{showId}/products` | Reorder the products of a show | `WRITE_PRODUCTS` |
| POST | `/shows/{showId}/products` | Add a product to a show | `WRITE_PRODUCTS` |
| POST | `/shows/{showId}/products/batch` | Add up to 100 products to a show | `WRITE_PRODUCTS` |
| DELETE | `/shows/{showId}/products/{productId}` | Remove a product from a show | `WRITE_SHOWS` |
| POST | `/shows/{showId}/tags` | Add a tag to a show | `WRITE_SHOWS` |
| PUT | `/shows/{showId}/tags` | Update tags in a show | `WRITE_SHOWS` |
| DELETE | `/shows/{showId}/tags/{tagId}` | Remove a tag from a show | `WRITE_SHOWS` |

## Statistics

| Method | Path | Summary | Scope |
|--------|------|---------|-------|
| GET | `/stats/activity` | Get show activity in a certain period | `READ_STATS` |
| GET | `/stats/activity/orders` | Get order data in a certain period | `READ_STATS` |
| GET | `/stats/show/{showId}` | Get statistics for a show | `READ_STATS` |
| GET | `/stats/show/{showId}/orders` | Get order data for a show | `READ_STATS` |
| GET | `/stats/show/{showId}/traffic-acquisition` | Get traffic acquisition data for a show | `READ_STATS` |
| GET | `/stats/shows` | Get statistics for multiple shows | `READ_STATS` |
| GET | `/stats/shows/orders` | Get order data for multiple shows | `READ_STATS` |
| GET | `/stats/traffic-acquisition` | Get traffic acquisition data for multiple shows | `READ_STATS` |

## Tags

| Method | Path | Summary | Scope |
|--------|------|---------|-------|
| GET | `/tags` | List tags | `READ_TAGS` |
| POST | `/tags` | Create a new tag | `WRITE_TAGS` |
| GET | `/tags/{tagId}` | Get tag details | `READ_TAGS` |
| PATCH | `/tags/{tagId}` | Update an existing tag | `WRITE_TAGS` |

## Users

| Method | Path | Summary | Scope |
|--------|------|---------|-------|
| GET | `/users` | List users | `READ_USERS` |
| POST | `/users` | Invite a user | `WRITE_USERS` |
| DELETE | `/users/{userId}` | Remove a user | `DELETE_USERS` |
| PATCH | `/users/{userId}` | Update user data | `WRITE_USERS` |
| GET | `/users/{userId}/assets` | List user assets | `READ_USERS` |
| POST | `/users/{userId}/assets` | Create a new user asset | `WRITE_USERS` |
| DELETE | `/users/{userId}/assets/{assetId}` | Delete a user asset | `WRITE_USERS` |

## Webhooks

| Method | Path | Summary | Scope |
|--------|------|---------|-------|
| POST | `/webhooks` | Create webhook | `WRITE_WEBHOOKS` |
| GET | `/webhooks/{eventId}` | Get webhook event | `READ_WEBHOOKS` |
