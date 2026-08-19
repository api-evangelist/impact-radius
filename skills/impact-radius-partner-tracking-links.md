---
name: impact-radius-partner-tracking-links
description: As a media partner, find promotable programs and ads on impact.com and mint tracking links and promo codes.
api: impact.com Partner API
version: v15
base_url: https://api.impact.com/MediaPartners/{AccountSID}
generated: '2026-08-13'
method: generated
source: openapi/impact-radius-partner-trackinglinks-v15.yml, openapi/impact-radius-partner-ads-v15.yml, openapi/impact-radius-partner-promo-codes-v15.yml
operations:
  - listPrograms
  - retrieveProgram
  - listContracts
  - retrieveContract
  - listAds
  - retrieveAd
  - retrieveAdTrackingLink
  - retrieveAdCode
  - retrieveAdIFrameCode
  - createTrackingLink
  - listPromoCodes
  - retrievePromoCode
  - listCatalogs
  - searchCatalogItems
  - listMediaProperties
  - createMediaProperty
---

# Mint impact.com tracking links as a partner

Note the path casing: the Partner API is served under `/Mediapartners/{AccountSID}`, not
`/MediaPartners`, in the published v15 paths. Copy it from the spec rather than from prose.

## Steps

1. **Know what you are contracted to.** `listPrograms` then `retrieveProgram`; `listContracts`
   and `retrieveContract` tell you the payout terms you are actually working under.
2. **Register where you will promote.** `listMediaProperties` / `createMediaProperty` — a media
   property is your site, app or social account, and brands use it to evaluate you.
3. **Find the creative.** `listAds` / `retrieveAd` scoped by `CampaignId`. For an existing ad,
   `retrieveAdTrackingLink` returns the tracking link, `retrieveAdCode` and
   `retrieveAdIFrameCode` return pasteable HTML.
4. **Mint a custom link.** `createTrackingLink` builds a link for a destination you choose. This
   is also the one impact.com MCP tool with an exact REST counterpart (`create_tracking_links`),
   so an agent can do this over MCP or over REST with identical effect.
5. **Promo codes.** `listPromoCodes` / `retrievePromoCode` for codes assigned to you.
6. **Products.** `listCatalogs` then `searchCatalogItems` to find specific items worth promoting.

## Rules that will bite you

- `createTrackingLink` has no idempotency key. Calling it twice makes two links, both live.
- Default rate limit is 1,000 requests/hour. Batch link creation, do not loop it per product
  without watching `X-RateLimit-Remaining`.
- Pin `IR-Version: 15` on every request so an account-level version change does not reshape your
  responses mid-integration.

## References

- `openapi/impact-radius-partner-trackinglinks-v15.yml`
- `mcp/impact-radius-tool-crosswalk.yml`
- `conventions/impact-radius-conventions.yml`
