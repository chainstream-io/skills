# Authentication

ChainStream supports three authentication modes. CLI handles auth automatically once configured.

## Mode 1: Turnkey Wallet (Recommended)

Wallet keys stored in Turnkey TEE (AWS Nitro Enclaves). Never on local disk.

```bash
# First time: create wallet
chainstream login user@example.com    # Email OTP → creates EVM + Solana wallet
chainstream login --key               # P-256 key (returning users, no email)

# CLI signs each request with X-Wallet-* headers automatically
```

Headers added per request:
```
X-Wallet-Address: <wallet address>
X-Wallet-Chain: evm | solana
X-Wallet-Signature: <signature of "chainstream:{chain}:{address}:{timestamp}:{nonce}">
X-Wallet-Timestamp: <unix seconds>
X-Wallet-Nonce: <unique per request>
```

## Mode 2: API Key

```bash
npx @chainstream-io/cli config set --key apiKey --value <key>
```

Header: `X-API-KEY: <key>`

Get a key at [app.chainstream.io](https://app.chainstream.io). Sufficient for read-only operations. DeFi operations require a wallet.

## Mode 3: x402 Wallet (USDC Quota)

When requests return 402, purchase a quota plan:

| Plan | Price (USDC) | Quota (CU) | Duration |
|------|-------------|------------|----------|
| nano | $1 | 50,000 | 30 days |
| micro | $5 | 350,000 | 30 days |
| starter | $20 | 1,500,000 | 30 days |
| growth | $50 | 4,000,000 | 30 days |
| pro | $150 | 15,000,000 | 30 days |

See [x402-payment.md](x402-payment.md) for the purchase flow.

## Auth Priority

quota-service checks in order: Wallet Signature → API Key → JWT Bearer.
CLI detects in order: Turnkey session → Raw wallet → API Key.
