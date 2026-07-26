---
name: Onboard a seller/merchant with Finix
description: Provision a merchant for platform payments by creating a seller Identity, creating a Merchant, and running verification.
api: openapi/finix-payments-openapi-original.yml
operations: [createIdentity, createMerchant, createMerchantVerification, getMerchant, listMerchantVerifications]
---

# Onboard a seller/merchant with Finix

For platform / marketplace payments, provision each seller as a `Merchant`. Use HTTP Basic Auth and the
`Finix-Version: 2022-02-01` header.

## Steps

1. **Create the seller Identity** — `createIdentity` (`POST /identities`) with the business `entity`
   (legal name, tax id, owners, etc.). Returns an `ID` identity.
2. **Provision the Merchant** — `createMerchant` (`POST /identities/{identity_id}/merchants`) referencing a
   processor. The merchant returns `PROVISIONING`; a `merchant.created` webhook fires.
3. **Run verification / underwriting** — `createMerchantVerification`
   (`POST /merchants/{merchant_id}/verifications`). Track results with `listMerchantVerifications`
   (`GET /merchants/{merchant_id}/verifications`).
4. **Poll or subscribe** — `getMerchant` (`GET /merchants/{merchant_id}`) reads `onboarding_state`
   (`APPROVED` / `REJECTED` / `UPDATE_REQUESTED`); the `merchant.underwritten` webhook signals approval.

## Rules

- A merchant must be `APPROVED` before it can process Transfers; charging an unapproved merchant returns `422`.
- Use `sandbox/finix-payments-sandbox.yml` name-verification and bank-validation sentinels to exercise
  onboarding outcomes in Sandbox.
- Default webhook subscriptions include `merchant.created`, `merchant.updated`, `merchant.underwritten`
  (`asyncapi/finix-payments-webhooks.yml`).
