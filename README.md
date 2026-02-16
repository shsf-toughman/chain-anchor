# Chain Anchor

Public verifiable anchoring for self-hosted FISCO BCOS blockchain.

## What is this?

This repository serves as **tamper-evident proof** that our FISCO BCOS blockchain data existed at specific points in time. Since we operate a self-hosted 4-node FISCO BCOS network, an external anchoring mechanism is necessary to prevent disputes about data fabrication.

## How it works

```
FISCO BCOS (self-hosted)        External Anchoring
┌─────────────────────┐     ┌──────────────────────────┐
│  All business data   │     │  Daily at 02:00 UTC+8:   │
│  across all apps     │     │                          │
│         │            │     │  1. Fetch latest block   │
│         ▼            │     │  2. Submit blockHash     │
│  Sealed in blocks    │────▶│     to OpenTimestamps    │
│  with blockHash      │     │  3. Commit to this repo  │
└─────────────────────┘     └──────────────────────────┘
```

**One daily blockHash anchors the entire chain** — every transaction from every application, from genesis block to the anchored block, is implicitly committed by that single hash.

## Directory structure

```
anchors/          — Daily anchor records (JSON)
  └── YYYY/MM/DD.json
proofs/           — OpenTimestamps proof files
  └── YYYY/MM/DD.json.ots
docs/             — GitHub Pages public site
  └── index.html
```

## Anchor record format

```json
{
  "blockNumber": 12345,
  "blockHash": "0xabc123...",
  "anchoredAt": "2026-02-16T02:00:00+08:00"
}
```

## How to verify

### Prerequisites

```bash
pip install opentimestamps-client
```

### Step 1: Verify OTS proof against Bitcoin

```bash
ots verify proofs/2026/02/16.json.ots
```

This confirms the blockHash was submitted to Bitcoin's blockchain before a certain time.

### Step 2: Confirm blockHash matches your chain

Query your FISCO BCOS node for the same block number:

```bash
# Via WeBASE-Front API
curl http://<webase-url>/WeBASE-Front/1/web3/blockByNumber/<groupId>/<blockNumber>
```

Compare the returned `hash` field with `blockHash` in the anchor record. They must match.

### Step 3: Verify a specific transaction existed before the anchor

1. Find which block contains your transaction (by `tx_hash` or `block_number` stored in your application database).
2. Confirm that block number < anchored block number.
3. Since blockHash chains are sequential, the anchored blockHash commits to all prior blocks.

**Conclusion:** Your transaction provably existed before the OTS-anchored Bitcoin timestamp.

## Trust model

| Layer | Trust basis | Tamperable? |
|-------|------------|-------------|
| FISCO BCOS (self-hosted) | Operator-controlled | Yes (single operator) |
| This GitHub repo | GitHub's commit timestamps | Theoretically (with GitHub cooperation) |
| OpenTimestamps → Bitcoin | Bitcoin PoW consensus | No (practically impossible) |

The three layers reinforce each other. Even if one layer is compromised, the remaining two provide independent evidence.

## License

This repository and its contents are public for transparency and verification purposes.
