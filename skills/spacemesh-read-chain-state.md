---
name: Read Spacemesh chain state
description: Read accounts, transactions, layers and rewards from a Spacemesh node via the v2beta1 gRPC-gateway API.
api: openapi/spacemesh-v2beta1-openapi-original.json
operations: [NetworkService_Info, NodeService_Status, AccountService_List, TransactionService_List, RewardService_List, LayerService_List]
---

# Read Spacemesh chain state

Spacemesh is a proof-of-space-time L1. Its node exposes a gRPC API with a
grpc-gateway REST/JSON facade. There is **no authentication** — the API is a
read-mostly node interface, gated by network reachability and per-service node
config, not by API keys or OAuth. Point requests at a running go-spacemesh
node's gateway host (there is no live vendor-hosted endpoint).

## Steps

1. Confirm the network with `NetworkService_Info` (genesis time, layer duration, network id).
2. Confirm the node is synced with `NodeService_Status` before trusting results — an un-synced node returns partial state (expect FAILED_PRECONDITION/UNAVAILABLE otherwise).
3. Look up account balance/nonce with `AccountService_List`, passing the bech32 `addresses`.
4. List a party's transactions with `TransactionService_List`; page with `offset` + `limit`.
5. List smeshing rewards with `RewardService_List` filtered by `coinbase` and layer range.
6. List layers and their status (applied/verified) with `LayerService_List`.

## Rules

- Pagination is `offset` + `limit` (uint64); do not assume cursors.
- Errors are `google.rpc.Status` (numeric gRPC code + message), not RFC 9457 problem+json — see errors/spacemesh-problem-types.yml.
- Reads are idempotent; there is no idempotency-key header.
