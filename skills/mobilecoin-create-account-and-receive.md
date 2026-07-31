---
name: Create an account and receive MobileCoin
description: Provision a new Full-Service wallet account, expose a receiving address, and confirm incoming funds.
api: MobileCoin Full-Service Wallet API (JSON-RPC 2.0)
operations:
  - create_account
  - assign_address_for_account
  - get_account_status
  - get_address_status
---

# Create an account and receive MobileCoin

Use the Full-Service JSON-RPC 2.0 API (default `http://127.0.0.1:9090/wallet/v2`).
Every request is a POST with body `{ "method": "...", "params": {...}, "jsonrpc": "2.0", "id": 1 }`.
If the service was started with `MC_API_KEY`, send it in the `X-API-KEY` header.

## Steps

1. **Create the account** — call `create_account` with a friendly `name`. Persist
   the returned `account.id`. Immediately call `export_account_secrets` and store
   the mnemonic securely; it is the only way to recover the account.
2. **Get a receiving address** — call `assign_address_for_account` with the
   `account_id` to mint a subaddress, or reuse the main subaddress. Share the
   returned b58 `public_address` with the sender (optionally wrap it in a
   payment request via `create_payment_request`).
3. **Wait for sync** — the ledger must scan before balances are accurate. Poll
   `get_account_status` and check the syncing/`is_synced` fields before trusting
   a balance.
4. **Confirm receipt** — call `get_address_status` (or `get_account_status`) and
   read the per-`token_id` balances to confirm the incoming TXO landed.

## Rules
- Handle errors from the JSON-RPC `error` object (`code`/`message`/`data.server_error`); see `errors/mobilecoin-problem-types.yml`.
- There is no idempotency key — account creation is not automatically de-duplicated, so guard against re-calling `create_account`.
- Balances are only meaningful once the account is fully synced.
