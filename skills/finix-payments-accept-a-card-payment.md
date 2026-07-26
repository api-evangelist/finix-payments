---
name: Accept a card payment with Finix
description: Authorize and capture a card payment by creating an Identity, tokenizing a card as a Payment Instrument, then authorizing and capturing a Transfer.
api: openapi/finix-payments-openapi-original.yml
operations: [createIdentity, createPaymentInstrument, createAuthorization, captureAuthorization, createTransfer]
---

# Accept a card payment with Finix

Use the Finix API (version `2022-02-01`) to take a card payment. All requests use HTTP Basic Auth
(`Authorization: Basic base64(APIKeyId:APIKeySecret)`) and must send `Finix-Version: 2022-02-01`.
Test against `https://finix.sandbox-payments-api.com` before going live.

## Steps

1. **Create a buyer Identity** — `createIdentity` (`POST /identities`). Supply the buyer's `entity` details.
   Returns an `ID`-prefixed identity id.
2. **Tokenize the card** — collect card data client-side with finix.js (never handle raw PAN), then
   `createPaymentInstrument` (`POST /payment_instruments`) referencing the `identity`. Returns a
   `PI`-prefixed instrument.
3. **Authorize** — `createAuthorization` (`POST /authorizations`) with `merchant`, `source` (the `PI`),
   `amount` (in cents), `currency`. Include a fresh UUID `idempotency_id` to make the call safe to retry.
   A decline returns `failure_code` + `failure_message`.
4. **Capture** — `captureAuthorization` (`PUT /authorizations/{authorization_id}`) with
   `capture_amount` to move funds; this creates the `TR` Transfer. Authorizations must be captured within 7 days.

## Alternative: direct sale

Skip auth/capture and call `createTransfer` (`POST /transfers`) with `merchant`, `source`, `amount`,
`currency`, and an `idempotency_id` to charge in one step.

## Rules

- Idempotency: send `idempotency_id` on `POST /authorizations` and `POST /transfers`; a repeat id with a
  differing payload returns `422`.
- Errors: non-2xx responses return `_embedded.errors[]` (`code`, `logref`, `message`). See
  `errors/finix-payments-problem-types.yml` and decline handling in `errors/finix-payments-decline-codes.yml`.
- Test declines by passing trigger amounts/cards from `sandbox/finix-payments-sandbox.yml`.
