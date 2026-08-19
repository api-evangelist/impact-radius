---
name: impact-radius-conversion-reconciliation
description: Investigate why an impact.com conversion (Action) is missing, wrong, or not payable, and correct or reverse it.
api: impact.com Brand API
version: v14
base_url: https://api.impact.com/Advertisers/{AccountSID}
generated: '2026-08-13'
method: generated
source: openapi/impact-radius-brand-actions-v14.yml, openapi/impact-radius-brand-actioninquiries-v14.yml
operations:
  - listActions
  - getActionById
  - listActionItems
  - getActionItemBySku
  - listActionUpdates
  - getActionUpdateById
  - updateActionOrItems
  - reverseAction
  - listActionInquiries
  - getActionInquiryById
  - updateActionInquiry
---

# Reconcile an impact.com conversion

Use when a partner says a sale did not track, a commission looks wrong, or finance asks why an
action is not payable yet.

## Before you start

- Authenticate with HTTP Basic: `Authorization: Basic base64(AccountSID:AuthToken)`. HTTPS only.
- You need `CampaignId`. Get it from `listPrograms`; `Program.CampaignId` is the join key for
  nearly every other list call.
- Pin the API version explicitly with the `IR-Version` header (or `IrVersion` query parameter)
  so a later account-level upgrade does not silently change your field set.

## Steps

1. **Find the action.** Call `listActions` with `CampaignId` and the narrowest date window you
   can. Date rules are enforced by the API, not just the docs: `StartDate` cannot be more than
   3 years back, the `StartDate`/`EndDate` range cannot exceed 45 days, and specifying
   `StartDate` obliges you to specify `EndDate`. With neither, you get the last 7 days only.
2. **Mind what the dates mean.** `StartDate`/`EndDate` filter on *last updated*, not event date,
   and a state transition (PENDING to APPROVED) does not count as an update. To find actions by
   approval date use `LockingDateStart`/`LockingDateEnd` instead. This is the single most common
   reason a reconciliation query "loses" a conversion.
3. **Read the action.** `getActionById` for the full record — `State` (PENDING / APPROVED /
   REVERSED), payout, `MediaPartnerId`, `AdId`, `SharedId`, `CustomerId`.
4. **Read the line items.** `listActionItems`, or `getActionItemBySku` when the dispute is about
   one product. Item-level discounts and quantities are where payout mismatches usually live.
5. **Read the history before you conclude anything.** `listActionUpdates` is an append-only
   changelog of every modification, and `getActionUpdateById` opens one entry. Poll this rather
   than diffing `listActions` yourself — it is the platform's audit feed and it carries
   `ContractId` and `PaystubId`, which tell you which contract priced the action and whether it
   has been paid.
6. **Check for an existing dispute.** `listActionInquiries` filtered by `ActionId` or `OrderId`.
   If the partner already raised one, resolve that inquiry with `updateActionInquiry` rather than
   editing the action underneath them.
7. **Correct or reverse.** `updateActionOrItems` adjusts the action or its items;
   `reverseAction` reverses it outright. Both are irreversible from the API's point of view.

## Rules that will bite you

- **There is no idempotency key.** impact.com publishes no idempotency header on any of its 242
  operations. A retried `updateActionOrItems` or `reverseAction` is a second real mutation.
  De-duplicate client-side on `OrderId` and record what you sent before you send it.
- **Paginate by following the response.** Use `Page`/`PageSize`, then follow `@nextpageuri` from
  the response envelope rather than incrementing `Page` yourself. `@total` and `@numpages` tell
  you when to stop. If the total exceeds ten times your page size the API errors instead of
  truncating — narrow the date range or raise `PageSize`.
- **Rate limits are low.** 1,000 requests/hour by default. Watch `X-RateLimit-Remaining` and
  `X-RateLimit-Reset` (seconds to reset) and back off on `429`.
- **Errors are not RFC 9457.** On failure you get `{"Status":"ERROR","Message":"...","Errors":[{"Field":"...","Message":"..."}]}`.
  Read `Errors[].Field` to find the offending parameter. A `403` means your scoped token does not
  enable this endpoint, which is a different fix from a `401`.

## References

- `openapi/impact-radius-brand-actions-v14.yml`
- `errors/impact-radius-problem-types.yml`
- `conventions/impact-radius-conventions.yml`
- `rate-limits/impact-radius-rate-limits.yml`
