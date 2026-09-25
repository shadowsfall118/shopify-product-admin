# Shopify Product Admin

A customer-tag-restricted storefront product table, assembled from the “Shopify SKU App Creation” conversation. Includes product thumbnails, expandable SKU lists with variant Admin links, optional columns, sortable headers, search, background loading, and 50-row browser pagination. No app installation, API key, or build step is required.

## Files

- `sections/product-admin.liquid` — complete section, settings, access check, and inline JavaScript.
- `assets/product-admin.css` — complete styles, including mobile layout and loading states.
- `README.md` — installation and operating notes.
- `LICENSE` — MIT license.

This ZIP is an add-on package, not a complete Shopify theme. Unzip it and copy the two code files into an existing theme; do not upload the ZIP as a theme.

## Setup

1. Duplicate your theme as a backup and work on the duplicate first.
2. In **Online Store → Themes → Edit code**, create `assets/product-admin.css` and paste the entire supplied CSS file. Create `sections/product-admin.liquid` and paste the entire supplied Liquid file. Save both. Replace existing versions completely instead of appending patches.
3. Open the theme editor. In the page selector, choose **Pages → Create template**. Name it `product-admin`, based on the default page.
4. In this new template, remove the normal page-content section if you do not want a duplicate title. Add **Product Admin** and save. Use one Product Admin section per page.
5. In the section settings, set the heading, required customer tag (default `product-admin`), and desired columns. Product name and thumbnail always appear.
6. Create a page under **Online Store → Pages**, titled Product Admin, with empty body content. Assign the `product-admin` theme template and save. The typical URL is `/pages/product-admin`; check the actual handle.
7. If the new template is missing from the page's template selector, ensure it also exists in the published theme. Shopify's assignment selector uses published-theme templates. Test the duplicate using theme preview before publishing your completed theme.
8. Configure customer access below, then perform the acceptance checks before sharing the page.

Alternative to steps 3–4: create `templates/page.product-admin.json` in the theme with this complete content, then continue from step 5:

```json
{
  "sections": {
    "main": {
      "type": "product-admin",
      "settings": { "required_tag": "product-admin" }
    }
  },
  "order": ["main"]
}
```

## Customer tag access

Enable customer sign-in in the store's customer-account settings. In **Customers**, open the customer record that will be used to sign into the storefront, add the exact tag `product-admin`, and save. If you customize the section's Required customer tag, use that same spelling and capitalization on the customer record.

Sign into the storefront as that customer, then return to the Product Admin page. Signing into Shopify Admin alone does not authenticate a storefront customer. The section uses Liquid's `customer` and `customer.tags`; test that your chosen customer-account configuration establishes the storefront session after login. Hosted account pages themselves do not render this section.

- Signed out: Sign in required and a login link.
- Signed in without the exact tag: Access restricted.
- Signed in with the tag: product table.
- Blank required-tag setting: access denied to everyone. This deliberately tightens the original conversation's blank-tag behavior.

The access check runs in Liquid before emitting table data or its JavaScript, including Section Rendering API requests. A customer tag grants access to this table only. It does not grant staff permissions. Product and expanded variant links open Shopify Admin in a new tab, where Shopify requires its own login and product permissions.

This section does not make the underlying storefront products private, and does not restrict other sections added to the page. Removing a tag affects subsequent requests; already delivered browser content remains until refreshed. Avoid using this theme feature as a confidential inventory system.

## Compatibility and data limits

Designed for **Dawn 2.0+** and Online Store 2.0 JSON page templates. It uses Dawn's `page-width` container and otherwise scoped CSS with no external JavaScript dependencies. Customized Dawn versions and other OS 2.0 themes may require spacing or typography adjustments. Current browsers with Fetch, URL, DOMParser, Map, AbortController, Promise.allSettled, and replaceChildren are required. Internet Explorer is unsupported.

This is a storefront view, not a complete Shopify Admin catalog or an editing app. `collections.all.products` reflects products available to the storefront and the current collection/market context. Draft, archived, unpublished, or otherwise unavailable products may be absent. Customizing the all-products collection changes the result set. Inventory is the sum of `inventory_quantity` values exposed for the variants Liquid returns; it is not a location-specific stock report. For untracked variants Shopify may expose quantities with different semantics, so the total must not be treated as authoritative tracked stock. Availability uses `product.available` and may differ from a positive inventory total, including products that allow continued selling.

Price displays the minimum storefront price, with `+` for varying prices, using the theme's money format and current context. Numeric price sorting uses that minimum value. SKU sorting compares the concatenated nonblank variant SKUs in Liquid order; it does not sort each expanded variant independently. Inventory and price sort numerically; other columns use natural, case-insensitive text comparison.

Liquid product pagination allows at most **250 products per batch** and access only through the **25,000th item**. This package caps requests at 100 batches and reports incomplete results if the total exceeds that limit. It is intended for several thousand ordinary products; large SKU lists still consume network bandwidth and memory.

Unpaginated `product.variants` exposes at most **250 variants per product**. This section does not fetch additional variant pages. Products beyond that limit can have incomplete SKU search, SKU lists, variant counts, and inventory totals even when all product batches finish loading. Use an authenticated app/backend for full high-variant or larger-catalog coverage.

## Search, sorting, and background loading

Liquid renders an initial batch of up to 250 products. JavaScript keeps the returned row nodes in memory and mounts only 50 rows at a time in the table after initialization. These are detached DOM nodes, not a compact JSON database; initial rendering can briefly contain the full batch. Additional batches are fetched through Shopify's Section Rendering API, two requests at a time, using the current page URL, locale/preview context, same-origin customer credentials, and the pagination parameter supplied by Liquid.

The loader fetches every other batch even if the initial URL contains `?page=2`. Product IDs prevent duplicate rows. Search matches a case-insensitive substring across product title, exposed SKUs, vendor, and product type, including columns hidden in settings. Sorting and search apply to all loaded rows before the browser chooses the visible 50. Changing search or sort resets browser pagination to page one. New batches reapply the current search and sort while retaining the browser page number; rows can move as more products arrive.

The default order is **SKU A–Z**, using natural, case-insensitive comparison (SKU2 precedes SKU10). Each incoming batch joins that sorted order. Blank SKUs sort first. The SKU header starts with an ascending arrow; its first click switches to Z–A. This default also applies when the SKU column is hidden.

Until the status says all products are loaded, results cover only loaded products. Failed, empty, unexpected, timed-out, or access-denied requests produce an incomplete-results message. Successfully fetched rows remain usable; reload to retry. A changing catalog can shift server pages during loading; this is not a transactional snapshot. Refresh to obtain updated prices and quantities. No data is written back to Shopify or stored in localStorage.

With JavaScript disabled, the current Liquid batch remains visible with basic batch navigation. Search, header sorting, and automatic loading require JavaScript. If you add this section for the first time in the theme editor and controls are inactive, reload the preview; test the real storefront page as well.

## Optional robots noindex guidance

To discourage indexing, add the following **inside `<head>` in `layout/theme.liquid`**, using the exact template suffix chosen above:

```liquid
{% if request.page_type == 'page' and template.suffix == 'product-admin' %}
  <meta name="robots" content="noindex, nofollow">
{% endif %}
```

Use this condition for every page assigned this template, including signed-out responses. Check for existing robots directives or SEO-app output to avoid conflicts. Do not put the meta tag inside the section body or inject it with JavaScript. Verify it in the page source after saving.

`noindex` requests exclusion from search results; `nofollow` addresses following links. Neither is authentication, and neither guarantees immediate removal of an already indexed URL. Do not simultaneously disallow this URL in robots.txt if you want crawlers to read its noindex tag. Noindex is optional and is not automatically installed by this package.

## Acceptance checks and troubleshooting

Before publishing, check all three customer states in separate sessions: signed out, untagged customer, and tagged customer. For a denied session, also request the page with `?section_id=SECTION_ID` (the section ID is visible in page source): the response must contain no product rows or data attributes. Never bypass the access check just to preview the table.

With a tagged customer:

1. Load a catalog with more than 250 products, wait for completion, and search for a SKU from a later batch. Also test a no-match search and clearing search.
2. Toggle each sort twice, including numeric inventory and price. Go Next and Previous and confirm a maximum of 50 visible rows.
3. Open a multi-SKU disclosure and test its variant link, plus a product title and thumbnail link. These generic `/admin/products/ID[/variants/ID]` routes depend on Shopify's Admin redirects; confirm them on your shop.
4. Reload with `?page=2`; all other batches should still load. Simulate a failed network request and check the incomplete-results message.
5. Verify mobile scrolling, keyboard focus, optional columns, empty collections, and any high-variant products.

If access fails, check the exact tag, section setting, storefront login, and template assignment. If loading fails, check connectivity, session expiry, and whether the correct section is assigned to that page. If styling is missing, confirm the asset filename. Remove the two code files and the dedicated template/page to uninstall, and remove the optional head condition if installed; preserve a backup first.

## Verification and provenance

The package consolidates the conversation's base section and CSS, sortable headers, expanded variant links, and background loader. Packaging fixes include a single pagination scope, escaped text, fail-closed blank tags, page-parameter handling, duplicate prevention, explicit incomplete-load handling, and accessible sort states. The truncated final mobile CSS in the cached source was completed.

Local verification covers Shopify Theme Check, JavaScript syntax, schema/settings and asset consistency, and simulated browser behavior. Live storefront authentication, actual Shopify responses, Admin redirects, theme appearance, and store-specific inventory semantics require the acceptance checks above; no live-store certification is implied.

## References

- [Shopify Section Rendering API](https://shopify.dev/docs/api/ajax/section-rendering)
- [Liquid paginate limits](https://shopify.dev/docs/api/liquid/tags/paginate)
- [Liquid product and variant limits](https://shopify.dev/docs/api/liquid/objects/product)
- [Liquid customer](https://shopify.dev/docs/api/liquid/objects/customer)
- [Liquid variant inventory semantics](https://shopify.dev/docs/api/liquid/objects/variant)
- [Google robots meta tag guidance](https://developers.google.com/search/docs/crawling-indexing/robots-meta-tag)

## License

MIT; see `LICENSE`. Retain the copyright and permission notice when redistributing substantial portions. This is an independent theme customization, not an official Shopify product.
