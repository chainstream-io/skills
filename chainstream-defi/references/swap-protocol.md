# Swap Protocol Reference

## CLI → REST API Flow

```
Phase 1: chainstream dex quote
  → GET /v2/dex/:chain/quote
  → Returns: amountOut, minAmountOut, priceImpact, route

Phase 2: User confirms

Phase 3: chainstream dex swap
  → POST /v2/dex/:chain/swap
  → Returns: { serializedTx: base64 } (UNSIGNED transaction)
  → CLI signs locally (TEE or raw key)
  → POST /v2/transaction/:chain/send { signedTx: base64 }
  → Returns: { signature, jobId }

Phase 4: chainstream job status --id JOB_ID --wait
  → GET /v2/job/:id (polling) or SSE streaming
  → Returns: { status, hash, ... }
```

## DeFi API Endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/v2/dex/:chain/quote` | Get swap quote (read-only) |
| POST | `/v2/dex/:chain/swap` | Build unsigned swap transaction |
| POST | `/v2/dex/:chain/route` | Aggregator route + build transaction |
| POST | `/v2/transaction/:chain/send` | Broadcast signed transaction |
| GET | `/v2/transaction/:chain/gas-price` | Current gas price (EVM only) |
| POST | `/v2/transaction/:chain/estimate-gas-limit` | Estimate gas limit (EVM only) |
| GET | `/v2/job/:id` | Job status |
| GET | `/v2/job/:id/streaming` | Job status via SSE |

## Transaction Signing (Non-Custodial)

ChainStream does NOT hold wallet keys. The API returns **unsigned transactions** that must be signed locally.

### Solana

Server returns: `serializedTx` = base64-encoded `VersionedTransaction` with placeholder signatures.
CLI: deserialize → sign with wallet keypair → serialize → base64 → send.

### EVM

Server returns: `serializedTx` = base64-encoded unsigned RLP.
CLI: deserialize → sign with wallet private key → encode signed RLP → base64 → send.

## Swap Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `chain` | Yes | sol / bsc / eth |
| `userAddress` | Yes | Sender wallet address |
| `inputMint` | Yes | Input token address |
| `outputMint` | Yes | Output token address |
| `amount` | Yes | Input amount (smallest unit) |
| `slippage` | No | Tolerance (e.g. 0.01 = 1%) |
| `dex` | No | Specific DEX protocol |

## Supported DEX Protocols

| Chain | Protocols |
|-------|-----------|
| sol | Jupiter, Raydium, Orca |
| bsc | PancakeSwap, Kyberswap, OpenOcean |
| eth | Uniswap, Kyberswap |
