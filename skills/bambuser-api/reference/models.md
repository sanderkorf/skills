# Bambuser Live REST — models

Compact field lists from Bambuser OpenAPI **REST API for Live** v1. Prefer validating at boundaries (e.g. Zod) before persisting.

## Show

Response shape for `GET /shows`, `GET /shows/{showId}`, create/update responses.

| Field | Type | Notes |
|-------|------|--------|
| `id` | string | Stable show id |
| `orgId` | string | Organization that manages the show |
| `title` | string | |
| `description` | string | |
| `preview` | string | Preview image from the show stream |
| `state` | string | `upcoming` \| `scheduled` \| `live` \| `ended` |
| `published` | boolean | Viewable in the player |
| `isTestShow` | boolean | Test shows skip statistics |
| `isLive` | boolean | Currently live streaming |
| `isArchived` | boolean | Archived shows reject new media |
| `allowArchivedPlayback` | boolean | Playable after ended |
| `scheduledStartAt` | string \| null | ISO; player countdown |
| `createdAt` | string | ISO |
| `startedAt` | string | ISO |
| `endedAt` | string | ISO |
| `showLengthSeconds` | integer | Total length |
| `contributors` | string[] | User ids eligible to stream |
| `channels` | string[] | Linked channel ids |
| `products` | string[] | Product ids on the show |
| `tags` | object[] | Beta |
| `curtains` | object | `pending` / `paused` / `ended` with optional `backgroundImage` |
| `settings` | object | Chat visibility, moderation, viewer count, etc. |

## CreateShow

`POST /shows` — **required:** `title`.

| Field | Type | Notes |
|-------|------|--------|
| `title` | string | Required |
| `description` | string | |
| `scheduledStartAt` | string | ISO |
| `published` | boolean | |
| `contributors` | string[] | |
| `allowArchivedPlayback` | boolean | |
| `isTestShow` | boolean | Cannot change state later once set as test |
| `settings` | object | |
| `curtains` | object | |

## UpdateShow

`PATCH /shows/{showId}` — all fields optional.

| Field | Type | Notes |
|-------|------|--------|
| `title` | string | |
| `description` | string | |
| `scheduledStartAt` | string | Empty string clears schedule |
| `published` | boolean | |
| `contributors` | string[] | |
| `isArchived` | boolean | Archive / unarchive |
| `allowArchivedPlayback` | boolean | |
| `settings` | object | |
| `curtains` | object | |

## ProductModel

Used when adding/listing products. **required:** `publicUrl`.

| Field | Type | Notes |
|-------|------|--------|
| `id` | string | Bambuser product id |
| `publicUrl` | string | Required |
| `productReference` | string | External catalog reference (e.g. SKU / item id) |
| `title` | string \| null | |
| `thumbnail` | string \| null | |
| `brand` | string \| null | |

## ShowHighlights

Highlight clip on a show (beta). **required:** `id`, `startAt`, `products`, `startRel`.

| Field | Type | Notes |
|-------|------|--------|
| `id` | string | Highlight id |
| `showId` | string | |
| `orgId` | string | |
| `startAt` | string | ISO |
| `startRel` | number | Seconds from show start |
| `products` | string[] | Product ids highlighted |

## CreateWebhookRequest

`POST /webhooks` — **required:** `name`, `url`, `topics`, `headers`.

| Field | Type | Notes |
|-------|------|--------|
| `name` | string | Webhook name |
| `url` | string | Delivery URL |
| `topics` | string[] | Enum: `show`, `product`, `user`, `product-highlight`, `broadcast` |
| `headers` | `Record<string, string>` | Headers Bambuser sends with each delivery |

## WebhookEventModel

Inbound event body / `GET /webhooks/{eventId}` response.

| Field | Type | Notes |
|-------|------|--------|
| `collection` | string | `show` \| `product` \| `user` |
| `action` | string | `add` \| `update` \| `remove` |
| `payload` | object | Resource payload |
| `eventId` | string | Use to re-fetch and verify |
| `timestamp` | string | ISO |

## Channel

| Field | Type | Notes |
|-------|------|--------|
| `id` | string | |
| `title` | string | |
| `previewImages` | string[] | |
| `playlists` | object[] | Shows grouped by lifecycle / feature |

**CreateChannelRequest** — **required:** `title`.

## TagModel

| Field | Type | Notes |
|-------|------|--------|
| `id` | string | |
| `name` | string | |

## BroadcastModel

**required:** `id`.

| Field | Type | Notes |
|-------|------|--------|
| `id` | string | |
| `createdAt` | string | ISO |
| `length` | number | Seconds |
| `width` / `height` | number | |
| `type` | string | e.g. `archived`, `live` |
