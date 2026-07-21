# Product hydration

BamHub stores thin scraped PDP data (title, thumbnail, brand, reference). Cart-quality data (images, variations, prices, stock) comes from `PROVIDE_PRODUCT_DATA` → `player.updateProduct`, or from a BamHub Product Feed.

## Event payload

`player.EVENT.PROVIDE_PRODUCT_DATA` — `event.products[]`:

| Field | Meaning |
|-------|---------|
| `id` | Bambuser product id — first arg to `updateProduct` |
| `ref` | Product reference from workspace (SKU / catalog id / URL fallback) |
| `url` | PDP URL used when the product was added |

Resolve host catalog with `adapters.getProduct(ref, url)`. Prefer `ref` when it matches your catalog identity; fall back to parsing `url` (path → product URI/slug) when scrape used the URL as reference.

## updateProduct factory

Map `HostProduct` into the factory chain. Size-level `.sku()` is what `ADD_TO_CART` receives.

```js
player.on(player.EVENT.PROVIDE_PRODUCT_DATA, (event) => {
  event.products.forEach(async ({ ref, url, id: bambuserId }) => {
    const product = await adapters.getProduct(ref, url)

    player.updateProduct(bambuserId, (productFactory) =>
      productFactory.product((productDetailFactory) =>
        productDetailFactory
          .name(product.name)
          .brandName(product.brand ?? '')
          .introduction(product.introduction ?? '')
          .description(product.description ?? '')
          .sku(product.sku)
          .defaultVariationIndex(product.defaultVariationIndex ?? 0)
          .variations((variationFactory) =>
            product.variations.map((variation) =>
              variationFactory()
                .name(variation.name)
                .sku(variation.sku)
                .imageUrls(variation.imageUrls)
                .attributes((attributeFactory) => {
                  let attrs = attributeFactory
                  if (variation.colorName) attrs = attrs.colorName(variation.colorName)
                  if (variation.colorHexCode) attrs = attrs.colorHexCode(variation.colorHexCode)
                  return attrs
                })
                .sizes((sizeFactory) =>
                  variation.sizes.map((size) =>
                    sizeFactory()
                      .name(size.name)
                      .inStock(size.inStock)
                      .sku(size.sku)
                      .price((priceFactory) => {
                        let price = priceFactory.current(size.current)
                        if (size.currency) price = price.currency(size.currency)
                        if (size.original != null) price = price.original(size.original)
                        if (size.perUnit != null) price = price.perUnit(size.perUnit)
                        if (size.unitAmount != null) price = price.unitAmount(size.unitAmount)
                        if (size.unitDisplayName) price = price.unitDisplayName(size.unitDisplayName)
                        return price
                      }),
                  ),
                ),
            ),
          ),
      ),
    )
  })
})
```

## Structure rules

- Nest **sizes under variations** (e.g. colors) for two dropdowns. Flat single-level lists collapse into one combined dropdown.
- In-player PDP supports at most **two** variation axes.
- Sale display: set both `.current()` and `.original()` when discounted; player shows both when current &lt; original.
- Unit price display (e.g. `$77 / 100ml`): optional `.perUnit()`, `.unitAmount()`, `.unitDisplayName()` on the price factory.
- Product-level `.sku()` should align with BamHub product reference; ATC / wishlist use the **size** sku from the selected combination.

## Identity mapping

| Source | Typical use |
|--------|-------------|
| `ref` | SKU / item group / catalog id from BamHub or feed |
| `url` | PDP link — parse locale-aware path to host product lookup key |
| Size `sku` | Exact line item for host `addToCart` |

Keep mapping in one host helper (URL → URI, ref → SKU). Fail soft: skip `updateProduct` for unresolved products; log for ops.

## Stock updates

Call `updateProduct` again anytime (not only on `PROVIDE_PRODUCT_DATA`) when inventory changes. Viewers already in-session may keep stale stock UI until refresh; reject ATC with `out-of-stock` callback in the meantime (see [cart-events](cart-events.md)).

## Product Feed note

Feed sync (BamHub → Settings → Integration → Product Feed) can omit `PROVIDE_PRODUCT_DATA`. Limitations: one feed per BamHub, sync interval ≥ ~6 hours, multi-market catalogs often still need the event handler. Cart events remain required either way.
