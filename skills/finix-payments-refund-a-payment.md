---
name: Refund a Finix payment
description: Reverse a settled or captured Transfer, in full or partially, and confirm the reversal state.
api: openapi/finix-payments-openapi-original.yml
operations: [getTransfer, createTransferReversal, listTransferReversals]
---

# Refund a Finix payment

Refunds in Finix are **reversals** of an existing `TR` Transfer. Use HTTP Basic Auth and the
`Finix-Version: 2022-02-01` header.

## Steps

1. **Locate the Transfer** — `getTransfer` (`GET /transfers/{transfer_id}`) to confirm its `state`
   (`SUCCEEDED`/`PENDING`) and `amount`.
2. **Create the reversal** — `createTransferReversal` (`POST /transfers/{transfer_id}/reversals`) with a
   `refund_amount` in cents (omit or match the full amount for a full refund). Include a fresh
   `idempotency_id` so a network retry does not double-refund.
3. **Confirm** — `listTransferReversals` (`GET /transfers/{transfer_id}/reversals`) to read the reversal
   Transfer and its `state`.

## Rules

- Idempotency: `POST /transfers/{id}/reversals` accepts `idempotency_id`; a repeat with a different payload
  returns `422`.
- Partial refunds are allowed up to the original captured amount.
- A failed reversal returns `failure_code`/`failure_message` (`errors/finix-payments-decline-codes.yml`).
- Subscribe to `transfer.updated` webhooks to track the reversal asynchronously
  (`asyncapi/finix-payments-webhooks.yml`).
