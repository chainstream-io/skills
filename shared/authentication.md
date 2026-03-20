# Authentication

ChainStream supports wallet signature and API key authentication. Choose the integration path that matches your agent's wallet setup.

## Which Path to Use

| Your Agent Has | Integration Path | Auth | x402 Payment |
|---------------|-----------------|------|-------------|
| No wallet | CLI (`npx @chainstream-io/cli`) | CLI creates Turnkey wallet, signs automatically | CLI auto-handles |
| Own wallet private key | CLI with `wallet set-raw` | CLI signs with imported key | CLI auto-handles |
| Own wallet (embedded, MPC, etc.) | SDK (`@chainstream-io/sdk`) | Implement `WalletSigner` interface | Use `@x402/fetch` |
| Dashboard API key (human user) | CLI or SDK with API Key | `X-API-KEY` header | Not applicable (pre-paid) |

## Path 1: CLI with Built-in Wallet (recommended)

For agents that don't have their own wallet. CLI creates a Turnkey TEE wallet on first login — no email, no registration form, no seed phrase.

```bash
# First time: creates EVM + Solana wallet (no email required)
npx @chainstream-io/cli login

# Output:
#   Welcome! Your ChainStream wallet has been created.
#     EVM:    0x...
#     Solana: ...

# All subsequent calls: auth + x402 payment handled automatically
npx @chainstream-io/cli token search --keyword PUMP --chain sol
```

What happens under the hood:
1. CLI generates a P-256 key pair locally (`~/.config/chainstream/keys/`)
2. Sends the public key to ChainStream auth service
3. Auth service creates a Turnkey sub-organization with EVM + Solana wallets
4. CLI stores the organization ID and wallet addresses in `~/.config/chainstream/config.json`
5. All API calls use SIWX wallet signature auth; x402 payment uses EIP-3009 signed authorization

### Optional: Bind email for recovery

After creating a wallet, you can optionally bind an email for account recovery:

```bash
npx @chainstream-io/cli bind-email user@example.com
# Enter verification code: xxxxxx
# Email user@example.com bound successfully.
```

### Optional: Email OTP login

If you prefer email-based login (e.g., to recover an existing wallet on a new device):

```bash
# Interactive (TTY terminal):
npx @chainstream-io/cli login --email user@example.com
# Enter OTP code: xxxxxx
# Logged in successfully.

# Non-interactive (CI/CD, piped, headless agent):
npx @chainstream-io/cli login user@example.com
# Output: {"otpId":"...","email":"user@example.com"}
npx @chainstream-io/cli verify --otp-id <otpId> --code <code> --email user@example.com
```

## Path 1b: CLI with Your Own Private Key

For agents that already have a Base (EVM) or Solana private key and want to use the CLI (no Turnkey, no email):

```bash
# Import your existing private key (Base or Solana)
npx @chainstream-io/cli wallet set-raw --chain base
# Enter private key: <your hex private key>

# Verify address
npx @chainstream-io/cli wallet address
# EVM: 0xYourAddress

# All commands now use your imported key for auth + x402 payment
npx @chainstream-io/cli token search --keyword PUMP --chain sol
```

The CLI will use your private key for both SIWX authentication and x402 payment. This works with any standard Base (EVM) or Solana private key.

## Path 2: SDK with Your Own Wallet (agent already has wallet)

For agents using Privy, Circle, Crossmint, Lit Protocol, ZeroDev, or any wallet provider with `signMessage` + `signTypedData` capability.

**Two things your wallet must support:**
- `signMessage` (EIP-191 `personal_sign`) — for SIWX authentication on every API call
- `signTypedData` (EIP-712) — for x402 payment (one-time USDC plan purchase via EIP-3009)

### Full integration example

```typescript
import { ChainStreamClient, type WalletSigner } from "@chainstream-io/sdk";
import { x402Client } from "@x402/core/client";
import { wrapFetchWithPayment } from "@x402/fetch";
import { ExactEvmScheme } from "@x402/evm/exact/client";

// ── Step 1: Create WalletSigner for API authentication ──
// Adapt this to your wallet provider (Privy, Circle, etc.)
const myWallet: WalletSigner = {
  chain: "evm",
  address: "0xYourAgentWalletAddress",
  signMessage: async (message) => {
    // Must return EIP-191 personal_sign signature (65 bytes hex with 0x prefix)
    return yourWalletProvider.personalSign(message);
  },
};

// ── Step 2: Create SDK client (handles SIWX auth automatically) ──
const client = new ChainStreamClient("", {
  serverUrl: "https://api.chainstream.io",
  walletSigner: myWallet,
});

// ── Step 3: Set up x402 payment (handles 402 → pay → retry) ──
const x402 = new x402Client();
// Base: register with viem-compatible Account (must support signTypedData)
x402.register("eip155:8453", new ExactEvmScheme(yourViemCompatibleAccount));
// OR Solana: register with @solana/kit signer
// x402.register("solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp", new ExactSvmScheme(solanaSigner));
const x402Fetch = wrapFetchWithPayment(fetch, x402);

// ── Step 4: Purchase quota if needed ──
// First API call may return 402 if no subscription exists.
// Option A: Let x402Fetch handle it transparently
await x402Fetch("https://api.chainstream.io/x402/purchase?plan=nano");
// Option B: SDK calls — catch 402, purchase, retry
try {
  const tokens = await client.token.search({ q: "PUMP", chains: ["sol"] });
} catch (err) {
  if (err.message.includes("402") || err.message.includes("PAYMENT_REQUIRED")) {
    await x402Fetch("https://api.chainstream.io/x402/purchase?plan=nano");
    // Retry — now subscription is active
    const tokens = await client.token.search({ q: "PUMP", chains: ["sol"] });
  }
}
```

### Compatible wallet providers

| Provider | signMessage | signTypedData | x402 ready |
|----------|-------------|---------------|------------|
| Privy (server wallets) | `privy.walletApi.ethereum.signMessage()` | `privy.walletApi.ethereum.signTypedData()` | Official `@privy-io/x402` |
| Circle Programmable Wallets | Sign message API | Sign typed data API | Manual integration |
| Crossmint Smart Wallets | Create Signature API | Create Signature API | Manual integration |
| Lit Protocol PKP | `PKPEthersWallet.signMessage()` | `PKPEthersWallet.signTypedData()` | Manual integration |
| ZeroDev Kernel | Kernel account signing | EIP-712 intents | Manual integration |
| Any viem LocalAccount | `account.signMessage()` | `account.signTypedData()` | Native with `ExactEvmScheme` |

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
