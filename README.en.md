# Chain Anchor

Public verifiable anchoring for self-hosted FISCO BCOS blockchain.

[中文](README.md)

## What is this?

This repository serves as **tamper-evident time proof** for our self-hosted FISCO BCOS blockchain. We operate a single-machine 4-node FISCO BCOS network for product traceability and other business applications. Since all nodes are controlled by a single operator, an external anchoring mechanism is necessary to establish a verifiable timeline.

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

## What this proves and what it doesn't

**Can prove:**
- Data existed on the chain at a specific point in time (temporal existence proof)
- Data has not been tampered with after anchoring (post-anchor integrity)
- The anchoring timeline is continuous (daily records + Bitcoin timestamps)

**Cannot prove:**
- Whether the on-chain data reflects real-world business events
- Whether the operator recorded data truthfully before putting it on chain

**Root cause:** When all nodes are controlled by the same operator, the operator can fabricate data, put it on chain, then anchor it — the timeline remains perfectly valid. This limitation can only be eliminated by introducing independent third-party nodes.

**Value of the current approach:**
1. **Locks the timeline** — When third-party nodes join later, historical anchor records prove data wasn't backfilled
2. **Raises tampering cost** — Tampering requires compromising the private chain + GitHub history + Bitcoin OTS records simultaneously
3. **Demonstrates transparency** — Proactive external anchoring lays the trust foundation for future public disclosure

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

### Step 2: Confirm blockHash matches the chain

Query the FISCO BCOS node for the same block number:

```bash
curl http://<webase-url>/WeBASE-Front/1/web3/blockByNumber/<blockNumber>
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

**Evolution path:** When independent nodes join the FISCO BCOS network, Layer 1 upgrades from "operator-controlled" to "multi-party consensus", completing the full trust chain.

## License

This repository and its contents are public for transparency and verification purposes.
