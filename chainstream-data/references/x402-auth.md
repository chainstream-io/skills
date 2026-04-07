# x402 Wallet Authentication

Keyless access to ChainStream APIs using USDC on Base or Solana. No account registration needed.

> Full details: [shared/authentication.md](../../shared/authentication.md) and [shared/x402-payment.md](../../shared/x402-payment.md)

## How It Works

1. Purchase a USDC plan once — get a pool of Compute Units (CU)
2. API calls consume CU from the pool (different endpoints cost different CU)
3. Quota is valid for 30 days from purchase

Only the initial plan purchase involves a blockchain transaction. Subsequent API calls just need SIWX wallet signature (no on-chain payment per request).

## Plans

Plans are dynamic. Query the latest:

```bash
npx @chainstream-io/cli wallet pricing
# or: curl https://api.chainstream.io/x402/pricing
```

CLI always shows all plans and prompts the user to choose — there is no default plan.

## CLI: x402 Payment Flow

CLI automates the x402 signing flow, but this is a **real USDC payment** — an EIP-3009 `signTypedData` that authorizes fund transfer from the user's wallet. Agents MUST present plan options and get user confirmation before any purchase.

**⚠️ CLI auto-purchase only works in interactive terminals** (human user at keyboard). In agent/pipe mode, the interactive plan selection prompt will hang. Agents must explicitly: check `plan status` → run `wallet pricing` → present plans to user → let user choose → have user run purchase in their terminal.

```bash
# In an interactive terminal (human present):
npx @chainstream-io/cli token search --keyword BTC --chain eth

# If no subscription exists, CLI prompts the user to choose a plan:
# [chainstream] No active subscription. Available plans:
# [chainstream]   1. nano   - $X/mo (Y CU)
# [chainstream]   2. micro  - $X/mo (Y CU)
# [chainstream]   ...
# [chainstream] Select a plan: _
# [chainstream] Signing x402 payment (USDC transfer)...
# [chainstream] Subscription activated (expires: 2026-04-19T...)
# { data: [...] }
```

## SDK: `@x402/fetch`

When using the SDK with your own wallet:

```typescript
import { x402Client } from "@x402/core/client";
import { wrapFetchWithPayment } from "@x402/fetch";
import { ExactEvmScheme } from "@x402/evm/exact/client";
import { ExactSvmScheme } from "@x402/svm/exact/client";

const x402 = new x402Client();

// Register ONE payment scheme based on your wallet's chain:
// Base (USDC)
x402.register("eip155:8453", new ExactEvmScheme(yourViemAccount));
// Solana (USDC)
x402.register("solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp", new ExactSvmScheme(solanaSigner));

const x402Fetch = wrapFetchWithPayment(fetch, x402);

// Purchase — plan name MUST come from user's explicit selection
const resp = await x402Fetch("https://api.chainstream.io/x402/purchase?plan=<USER_CHOSEN_PLAN>");
```

Base payment requires `signTypedData` (EIP-712). Solana payment requires a `@solana/kit` signer. Both involve **real USDC transfer** — always confirm with user first.

## Authentication After Purchase

After purchasing a plan, all API calls use **SIWX** (Sign-In-With-X) authentication:

```
Authorization: SIWX <base64(message)>.<signature>
```

The SDK/CLI handles SIWX token creation and caching automatically. See [authentication.md](../../shared/authentication.md) for details.

## Check Current Subscription

```bash
# CLI (auto-detects wallet from config)
npx @chainstream-io/cli plan status

# API
curl "https://api.chainstream.io/x402/status?chain=evm&address=0x..."
```

Returns plan name, quota usage, expiry, and active status. See [x402-payment.md](../../shared/x402-payment.md#check-current-subscription) for response format.

## Supported Networks

| Network | Chain ID | USDC Contract |
|---------|----------|---------------|
| Base | `eip155:8453` | `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913` |
| Solana | `solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp` | `EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v` |
