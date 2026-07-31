---
name: Send a MobileCoin payment
description: Build and submit a MobileCoin transaction and give the recipient a receipt to verify delivery.
api: MobileCoin Full-Service Wallet API (JSON-RPC 2.0)
operations:
  - get_account_status
  - build_and_submit_transaction
  - create_receiver_receipts
  - check_receiver_receipt_status
---

# Send a MobileCoin payment

Use the Full-Service JSON-RPC 2.0 API (default `http://127.0.0.1:9090/wallet/v2`).
Requests are POSTs of `{ "method": "...", "params": {...}, "jsonrpc": "2.0", "id": 1 }`.

## Steps

1. **Check funds** — call `get_account_status` for the sending `account_id` and
   confirm the account is synced and has enough spendable balance (plus fee) for
   the target `token_id`.
2. **(Optional) preview** — call `build_transaction` first to inspect the
   `tx_proposal` (amount, fee, outputs) before spending. For the one-shot path,
   skip to step 3.
3. **Send** — call `build_and_submit_transaction` with `account_id`,
   `recipient_public_address`, and `amount` (`{ value, token_id }`). Capture the
   returned `transaction_log` and `tx_proposal`.
4. **Give the recipient proof** — call `create_receiver_receipts` with the
   `tx_proposal` and hand the receipts to the recipient. They call
   `check_receiver_receipt_status` to watch the transaction move to delivered.

## Rules
- Building and submitting are two phases; a built `tx_proposal` is not on-chain until submitted. See `conventions/mobilecoin-conventions.yml`.
- No idempotency key exists — do not blindly retry `build_and_submit_transaction` on a timeout; reconcile with `get_transaction_logs` first to avoid a double-spend attempt.
- Surface failures from the JSON-RPC `error` object; see `errors/mobilecoin-problem-types.yml`.
