---
name: impact-radius-catalog-sync
description: Publish and maintain a brand product catalog in impact.com so partners can promote real, in-stock items.
api: impact.com Brand API
version: v14
base_url: https://api.impact.com/Advertisers/{AccountSID}
generated: '2026-08-13'
method: generated
source: openapi/impact-radius-brand-catalogs-v14.yml, openapi/impact-radius-brand-jobs-v14.yml
operations:
  - listCatalogs
  - getCatalogById
  - updateCatalogPullSettings
  - uploadCatalogFile
  - listCatalogItems
  - createCatalogItem
  - getCatalogItemById
  - updateCatalogItem
  - deleteCatalogItem
  - bulkUpdateCatalogItems
  - listJobs
  - getJobById
  - downloadJobResult
---

# Sync a product catalog into impact.com

## Steps

1. **Find the catalog.** `listCatalogs` scoped by `CampaignId`, then `getCatalogById`.
2. **Decide push or pull.** `updateCatalogPullSettings` configures impact.com to fetch a feed
   from you on a schedule. If you would rather push, skip to step 3.
3. **Bulk load.** `uploadCatalogFile` submits a whole file and returns a job — this is the right
   path for a full sync. `bulkUpdateCatalogItems` applies a batch of changes.
4. **Track the job.** `getJobById` until complete, then `downloadJobResult` to read per-row
   outcomes. A catalog upload that returns 200 has been *accepted*, not applied — the job result
   is where rejected rows appear.
5. **Single items.** `createCatalogItem`, `updateCatalogItem`, `getCatalogItemById` and
   `deleteCatalogItem` for surgical corrections. `CatalogItem` carries 74 fields, so read one
   existing item before constructing a new one.
6. **Verify.** `listCatalogItems` filtered by `CatalogId`.

## Rules that will bite you

- **`/Catalogs` has its own hourly limit of 3,600 requests** — higher than the 1,000/hour
  default. It is the one endpoint family where per-item calls are affordable, but bulk is still
  cheaper and safer.
- No idempotency key exists. A retried `uploadCatalogFile` is a second upload. Record the job id
  returned by the first attempt and poll it instead of resubmitting.
- `CatalogItem.PromotionIds` links items to promotions and `ItemGroupId` groups variants — treat
  them as foreign keys, not free text.

## References

- `openapi/impact-radius-brand-catalogs-v14.yml`
- `data-model/impact-radius-data-model.yml`
- `rate-limits/impact-radius-rate-limits.yml`
