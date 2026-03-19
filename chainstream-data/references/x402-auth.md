# x402 Wallet Authentication

Keyless access to ChainStream APIs using USDC on Base or Solana. No account registration needed.

## Billing Model

x402 uses **quota-based billing** (not per-call payment):

1. Purchase a USDC plan once -- get a pool of Compute Units (CU)
2. API calls consume CU from the pool (different endpoints cost different CU)
3. Quota is valid for 30 days from purchase

This means only the initial plan purchase involves a blockchain transaction. Subsequent API calls just need wallet signature headers (no on-chain payment per request).

## Plans

| Plan | Price (USDC) | Quota (CU) | Duration |
|------|-------------|------------|----------|
| nano | $1 | 50,000 | 30 days |
| micro | $5 | 350,000 | 30 days |
| starter | $20 | 1,500,000 | 30 days |
| growth | $50 | 4,000,000 | 30 days |
| pro | $150 | 15,000,000 | 30 days |
| business | $500 | 55,000,000 | 30 days |

## User Confirmation

When encountering HTTP 401 (unauthorized) or 402 (payment required), the agent should:

1. Inform the user that authentication is required
2. Present x402 as an option: "You can purchase a quota plan with USDC. For example, the nano plan costs $1 for 50,000 API calls over 30 days."
3. Ask for confirmation before proceeding with any USDC expenditure
4. Only after explicit user approval, execute the purchase flow

Never spend USDC without user confirmation. Always be transparent about costs.

## Purchase Flow

After user confirms:

```bash
# Step 1: Request purchase -- returns 402 with payment requirements
curl -X POST https://api.chainstream.io/x402/purchase \
  -H "Content-Type: application/json" \
  -d '{"plan": "nano", "network": "eip155:8453", "walletAddress": "0xYOUR_WALLET"}'

# Step 2: Decode X-Payment-Required header, sign USDC payment, retry
curl -X POST https://api.chainstream.io/x402/purchase \
  -H "Content-Type: application/json" \
  -H "X-Payment: <base64-signed-payment>" \
  -d '{"plan": "nano", "network": "eip155:8453", "walletAddress": "0xYOUR_WALLET"}'
# Returns 200 with plan details, quota allocated to wallet
```

## Wallet Signature Auth

After purchasing a plan, every API call must include 5 wallet auth headers:

| Header | Value | Description |
|--------|-------|-------------|
| `X-Wallet-Address` | `0xAddr` or Base58 | Wallet that purchased the plan |
| `X-Wallet-Chain` | `evm` or `solana` | Chain type (NOT network ID) |
| `X-Wallet-Signature` | hex or base58 | Signature of the message |
| `X-Wallet-Timestamp` | Unix seconds | Current timestamp |
| `X-Wallet-Nonce` | unique string | Unique per request |

### Signature Message Format

```
chainstream:{chain}:{address}:{timestamp}:{nonce}
```

- **EVM**: sign with EIP-191 `personal_sign` -> 65-byte hex signature with `0x` prefix
- **Solana**: sign with ed25519 -> 64-byte base58 signature
- **Timestamp window**: 300 seconds -- older timestamps are rejected
- **Nonce**: must be unique per request (enforced server-side)

### Example

```bash
TIMESTAMP=$(date +%s)
NONCE=$(uuidgen)
MESSAGE="chainstream:evm:0xYOUR_WALLET:${TIMESTAMP}:${NONCE}"
SIGNATURE="0x..."  # EIP-191 personal_sign of MESSAGE

curl -H "X-Wallet-Address: 0xYOUR_WALLET" \
  -H "X-Wallet-Chain: evm" \
  -H "X-Wallet-Signature: ${SIGNATURE}" \
  -H "X-Wallet-Timestamp: ${TIMESTAMP}" \
  -H "X-Wallet-Nonce: ${NONCE}" \
  "https://api.chainstream.io/v2/token/search?keyword=PUMP&chain=sol"
```

## SDK WalletSigner Interface

The ChainStream SDK provides a `WalletSigner` interface that handles both plan purchase and per-request signing. It is wallet-agnostic -- any wallet that supports `signMessage` and token transfers works.

```typescript
interface WalletSigner {
  getAddress(): string | Promise<string>;
  getChain(): string;                           // "evm" or "solana"
  signPayment(req: PaymentRequirements): Promise<string>; // USDC transfer for plan purchase
  signMessage(message: string): Promise<string>;          // per-request auth signature
}
```

### Usage

```typescript
import { ChainStreamClient } from '@chainstream-io/sdk';

const client = new ChainStreamClient('', {
  walletSigner: myWalletSigner, // any wallet implementing the interface above
});

// SDK handles purchase (with 402 detection) and per-request signing automatically
const results = await client.token.search({ keyword: 'PUMP', chain: 'sol' });
```

### Compatible Wallets

The `WalletSigner` interface can be implemented with any wallet provider:

- **Coinbase AgentKit** (CDP) -- programmatic signing for AI agents
- **Privy** -- embedded wallets with server-side signing
- **Thirdweb** -- `wrapFetchWithPayment` for x402
- **Dynamic** -- multi-chain embedded wallets
- **MetaMask / Phantom** -- browser extension wallets
- **Private key** -- direct `ethers.Wallet` or `@solana/web3.js Keypair`

## Supported Networks

| Network | Chain ID | USDC Contract | Env |
|---------|----------|---------------|-----|
| Base | `eip155:8453` | `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913` | Production |
| Base Sepolia | `eip155:84532` | `0x036CbD53842c5426634e7929541eC2318f3dCF7e` | Testnet |
| Solana | `solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp` | `EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v` | Production |
| Solana Devnet | `solana:EtWTRABZaYq6iMfeYKouRu166VU2xqa1` | `4zMMC9srt5Ri5X14GAgXhaHii3GnPAEERYPJgZJDncDU` | Testnet |

## Pricing Endpoint

```bash
GET https://api.chainstream.io/x402/pricing
# Returns all plans with prices and quotas
```
