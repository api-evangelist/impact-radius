---
name: impact-radius-advocate-referral-lifecycle
description: Run a customer referral end to end on impact.com Advocate — register a participant, surface their share links, and issue and redeem rewards.
api: impact.com Advocate API
version: v1
base_url: https://app.referralsaasquatch.com/api/v1/{tenant_alias}
generated: '2026-08-13'
method: generated
source: openapi/impact-radius-brand-advocate-user-v13.yml, openapi/impact-radius-brand-advocate-referral-v13.yml, openapi/impact-radius-brand-advocate-reward-v13.yml, openapi/impact-radius-brand-advocate-webhook-v13.yml
operations:
  - upsertAccount
  - getAccount
  - createUser
  - openUserUpsert
  - getUser
  - getUserPII
  - findUserByReferralCode
  - blockUser
  - unblockUser
  - getSharelinks
  - getShareURLs
  - getCode
  - openValidateCode
  - openApplyCode
  - listReferrals
  - getReferral
  - moderateReferrals
  - createReward
  - getRewards
  - lookupReward
  - debitReward
  - cancelReward
  - listRewardBalances
  - debitRewardBalance
  - createWebhook
  - listWebhooks
  - testWebhook
  - deleteWebhook
---

# Run an Advocate referral end to end

Advocate is a different host and a different object graph from the partnership APIs. Base URL is
`https://app.referralsaasquatch.com/api/v1/{tenant_alias}` — impact.com's own GraphQL and REST
guides publish it, because Advocate is the productised Referral SaaSquatch acquisition.

## Before you start

- Authenticate with HTTP Basic using a **test-mode or live-mode** key. The key's mode must match
  the `tenant_alias` in the path or you get `RS003`, `RS006` or `RS032`.
- Server-side calls use the API key. Browser and mobile calls use a signed JWT in
  `X-SaaSquatch-User-Token` with the Account SID as a child of the JWT header. Never ship the
  tenant secret to a client — and note that using a JWT where an API key belongs is a documented
  cause of `RS033` rate-limit errors.

## Steps

1. **Create the account.** `upsertAccount` — an account groups users and is **hard-capped at
   1,000 users** (`RS013`). If you are tempted to put all your users under one `accountid`, that
   cap is the reason not to.
2. **Register the participant.** `createUser`, or `openUserUpsert` from a client-facing surface.
   `first_name` accepts ASCII letters only (`RS024`); emails must be at least 7 characters with a
   local part and a 2+ character TLD (`RS023`).
3. **Give them something to share.** `getSharelinks` / `getShareURLs` return per-medium URLs
   (EMAIL, EMBED, HOSTED, MOBILE, POPUP). `getCode` returns the referral code. If you use a custom
   short domain and the link is malformed you get `RS053`.
4. **Handle an inbound referral.** `openValidateCode` checks a code before you trust it;
   `openApplyCode` applies it; `findUserByReferralCode` resolves the referrer.
5. **Watch the referral.** `listReferrals` / `getReferral` — a referral joins `referrerUser`,
   `referredUser` and `referredReward`. `moderateReferrals` approves or rejects suspicious ones.
6. **Reward.** `createReward`, then `getRewards` / `lookupReward` to read state, `debitReward` or
   `debitRewardBalance` to redeem, `cancelReward` to void. `listRewardBalances` is the running
   balance a participant sees.
7. **Subscribe to events instead of polling.** `createWebhook` for `user.created`,
   `user.reward.balance.changed`, `coupon.created`, `reward.created`, `referral.started`,
   `referral.converted`, `export.created`, `export.completed`. `testWebhook` fires a `test` event
   at your endpoint. Delivery retries hourly for up to 72 attempts and is **not ordered** — make
   your receiver idempotent and reconcile by resource id, not by arrival order.

## Rules that will bite you

- Content-Type must be `application/json` (`RS046`) and the call must be HTTPS (`RS047`).
- `getUserPII` is a separate operation from `getUser` — personal data is deliberately not in the
  default read. Do not fetch it unless the task needs it.
- Anything not covered above almost certainly exists in GraphQL rather than REST: programs,
  segments, forms, microsites, integrations and webhook subscription management are GraphQL-only.
  The schema is in `graphql/impact-radius-advocate-schema.graphql`.
- Test data can be cleared with the `deleteTestTenantData` GraphQL mutation.

## References

- `openapi/impact-radius-brand-advocate-user-v13.yml`
- `asyncapi/impact-radius-advocate-webhooks.yml`
- `errors/impact-radius-problem-types.yml`
- `sandbox/impact-radius-sandbox.yml`
