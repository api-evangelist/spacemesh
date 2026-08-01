---
name: Submit a Spacemesh transaction
description: Parse, estimate gas for, and submit a signed transaction to a Spacemesh node, then track its state.
api: openapi/spacemesh-v2beta1-openapi-original.json
operations: [NodeService_Status, TransactionService_ParseTransaction, TransactionService_EstimateGas, TransactionService_SubmitTransaction, AccountService_List]
---

# Submit a Spacemesh transaction

Build and sign the transaction blob off-chain (e.g. with @spacemesh/sm-codec),
then submit it to a synced go-spacemesh node via the v2beta1 API. No API auth is
required; the transaction itself is authorized by its signature and account
nonce/counter.

## Steps

1. Check the node is synced with `NodeService_Status`.
2. Read the principal account's current `counter` (nonce) with `AccountService_List`.
3. Optionally sanity-check the signed blob with `TransactionService_ParseTransaction`.
4. Estimate cost with `TransactionService_EstimateGas`.
5. Submit the signed blob with `TransactionService_SubmitTransaction`.
6. Poll `TransactionService_List` for the transaction's state (mempool -> block -> applied).

## Rules

- There is no HTTP idempotency key. Replay protection is the account **counter/nonce**: resubmitting a transaction with the same counter is rejected as CONFLICTING. To retry safely, resubmit the identical signed blob (same nonce) rather than re-signing with a new nonce.
- Encode/decode transactions with the first-party codec `@spacemesh/sm-codec` (packages/spacemesh-packages.yml).
- Errors are `google.rpc.Status` — INVALID_ARGUMENT (bad blob), FAILED_PRECONDITION (node not synced). See errors/spacemesh-problem-types.yml.
