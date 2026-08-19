---
name: impact-radius-partner-recruitment-and-contracting
description: Recruit a partner into an impact.com program, review their application, and accept or decline the contract.
api: impact.com Brand API
version: v14
base_url: https://api.impact.com/Advertisers/{AccountSID}
generated: '2026-08-13'
method: generated
source: openapi/impact-radius-brand-partners-v14.yml, openapi/impact-radius-brand-contracts-v14.yml, openapi/impact-radius-brand-partnergroups-v14.yml
operations:
  - listPrograms
  - getProgramById
  - listPartners
  - getPartnerById
  - createPartner
  - updatePartner
  - listContracts
  - getContractById
  - acceptContract
  - declineContract
  - listPartnerGroups
  - createPartnerGroup
  - updatePartnerGroup
  - listNotes
  - createNote
---

# Recruit and contract an impact.com partner

Use to onboard a publisher, creator or referral partner and put them under commercial terms.

## Steps

1. **Pick the program.** `listPrograms`, then `getProgramById`. Keep `CampaignId` — every call
   below is scoped by it.
2. **Check whether the partner already exists.** `listPartners` filtered by `CampaignId`, then
   `getPartnerById`. Creating a duplicate partner is not recoverable through the API.
3. **Add the partner.** `createPartner`, or `updatePartner` to correct an existing record.
4. **Read the contract.** `listContracts` with `CampaignId` and `PartnerId`, then
   `getContractById`. A contract carries `CampaignTerms` (each with `EventPayouts`, and each
   payout with `PayoutGroups`, `Limits`, `PayoutRestrictions`, `PerformanceBonus` and
   `PayoutScheduling`), plus `Terms`, `TemplateTerms` and — when relevant — `ScheduledTerms` for
   terms that take effect later. Read `ScheduledTerms` before you conclude what a partner is
   being paid; the current terms may already be superseded.
5. **Decide.** `acceptContract` accepts the partner's application (optionally into a `groupId`);
   `declineContract` declines the proposal with an optional reason and `groupId`.
6. **Group and annotate.** `listPartnerGroups` / `createPartnerGroup` to segment partners, and
   `createNote` against the partner to leave a record of why the decision was made.

## Rules that will bite you

- **No idempotency key exists.** `createPartner`, `acceptContract` and `declineContract` are all
  unguarded against replay. Check with `listPartners` / `listContracts` before retrying.
- Accept and decline are terminal from the API's side. There is no un-accept operation.
- Scoped tokens gate this by API category. If `acceptContract` returns `403` while
  `listContracts` returns `200`, the token has read enabled and write disabled for Contracts.
- Follow `@nextpageuri` when walking partner or contract lists.

## References

- `openapi/impact-radius-brand-contracts-v14.yml`
- `data-model/impact-radius-data-model.yml`
- `authentication/impact-radius-authentication.yml`
