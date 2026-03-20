# Authentication

ChainStream supports wallet signature and API key authentication. Choose the integration path that matches your agent's wallet setup.

## Which Path to Use

| Your Agent Has | Integration Path | Auth | x402 Payment |
|---------------|-----------------|------|-------------|
| No wallet | CLI (`npx @chainstream-io/cli`) | CLI creates Turnkey wallet, signs automatically | CLI auto-handles |
| Own wallet (AgentKit, Privy, etc.) | SDK (`@chainstream-io/sdk`) | Implement `WalletSigner` interface | Use `@x402/fetch` |
| Dashboard API key (human user) | CLI or SDK with API Key | `X-API-KEY` header | Not applicable (pre-paid) |

## Path 1: CLI with Built-in Wallet (no existing wallet)

For agents that don't have their own wallet. CLI handles everything:

```bash
# Create wallet (first time)
npx @chainstream-io/cli login

# All subsequent calls: auth + x402 payment handled automatically
npx @chainstream-io/cli token search --keyword PUMP --chain sol
```

## Path 2: SDK with Your Own Wallet (agent already has wallet)

For agents that already have a wallet (Coinbase AgentKit, Privy, Thirdweb, etc.). Use the SDK's `WalletSigner` interface:

```typescript
import { ChainStreamClient, type WalletSigner } from "@chainstream-io/sdk";
import { x402Client } from "@x402/core/client";
import { wrapFetchWithPayment } from "@x402/fetch";
import { ExactEvmScheme } from "@x402/evm/exact/client";

// Step 1: Implement WalletSigner with your existing wallet
const myWallet: WalletSigner = {
  chain: "evm",
  address: "0xYourAgentWalletAddress",
  signMessage: async (message) => {
    // Use your agent's existing wallet to sign
    return yourWallet.personalSign(message);
  },
};

// Step 2: Create SDK client with your wallet
const client = new ChainStreamClient("", {
  serverUrl: "https://api.chainstream.io",
  walletSigner: myWallet,
});

// Step 3: Call APIs (wallet signature headers added automatically)
const tokens = await client.token.search({ q: "PUMP", chains: ["sol"] });
```

For x402 payment handling with your own wallet, wrap requests with `@x402/fetch`:

```typescript
// Your wallet must support signTypedData for x402 (EIP-3009)
const x402 = new x402Client();
x402.register("eip155:*", new ExactEvmScheme(yourViemAccount));
const x402Fetch = wrapFetchWithPayment(fetch, x402);

// Standard GET — x402Fetch handles 402 → sign → retry transparently
const resp = await x402Fetch("https://api.chainstream.io/x402/purchase?plan=nano");
```

Required packages: `@chainstream-io/sdk`, `@x402/core`, `@x402/fetch`, `@x402/evm`, `viem`

## Path 3: API Key (dashboard users)

For human users with a dashboard account. Read-only operations only (DeFi requires wallet).

```bash
npx @chainstream-io/cli config set --key apiKey --value <key>
```

Get a key at [app.chainstream.io](https://app.chainstream.io).

## SIWX Authentication (Standard)

All wallet-based requests use SIWX (Sign-In-With-X) authentication. SDK/CLI handles this automatically.

**How it works**: Sign a SIWE/SIWS message once → get a token valid for 1 hour → include in every request.

```
Authorization: SIWX <base64(message)>.<signature>
```

The message follows EIP-4361 (SIWE for EVM, SIWS for Solana) format:
```
api.chainstream.io wants you to sign in with your Ethereum account:
0xYourAddress

Sign in to ChainStream API

URI: https://api.chainstream.io
Version: 1
Chain ID: 8453
Nonce: <random>
Issued At: <ISO 8601>
Expiration Time: <ISO 8601, default 1h>
```

Signing method: `personal_sign` (EIP-191 for EVM, ed25519 for Solana).

SDK/CLI automatically caches the token and refreshes before expiry. Agent never needs to manage token lifecycle.

## Auth Priority

Server checks in order: SIWX → API Key → JWT Bearer.

## NEVER Do

- NEVER construct SIWX tokens manually — use SDK or CLI
- NEVER construct x402 `Payment-Signature` manually — use `@x402/fetch` or CLI
- NEVER expose wallet private keys in logs, command arguments, or chat messages
