# x402 Payment Protocol

When the API returns `402 Payment Required`, a quota plan must be purchased with USDC. Plans provide a **compute unit (CU) quota** — each API call consumes CU based on the endpoint and data volume returned, NOT a fixed per-call cost.

## IMPORTANT: Use CLI or @x402/fetch — Do NOT manually construct payments

x402 uses **EIP-3009 `transferWithAuthorization`** (off-chain signed authorization), NOT a simple USDC transfer. Manual curl construction will fail. You MUST use one of these methods:

### Method 1: CLI (recommended)

CLI handles x402 transparently — no manual payment steps:

```bash
# Just call any command. If 402, CLI auto-purchases and retries.
npx @chainstream-io/cli token search --keyword PUMP --chain sol
```

The CLI internally uses `@x402/fetch` to:
1. Detect 402 response
2. Sign EIP-3009 typed data with wallet (requires `signTypedData`, not `signMessage`)
3. Retry with `Payment-Signature` header
4. Return the API response — agent never sees the 402

### Method 2: Standard x402 GET (any x402-compatible wallet)

The purchase endpoint follows the x402 standard — compatible with any x402-compliant client:

```
GET https://api.chainstream.io/x402/purchase?plan=nano
→ 402 + Payment-Required header → client signs → retries with Payment-Signature → 200
```

### Method 3: @x402/fetch (programmatic)

For agents that need programmatic access without CLI:

```javascript
import { x402Client } from "@x402/core/client";
import { wrapFetchWithPayment } from "@x402/fetch";
import { ExactEvmScheme } from "@x402/evm/exact/client";
import { ExactSvmScheme } from "@x402/svm/exact/client";

const client = new x402Client();

// Base (EVM) — register with viem account
client.register("eip155:8453", new ExactEvmScheme(viemAccount));

// OR Solana — register with @solana/kit signer
client.register("solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp", new ExactSvmScheme(solanaSigner));

const x402Fetch = wrapFetchWithPayment(fetch, client);

// Standard GET — transparent: 402 → sign → retry → success
const resp = await x402Fetch("https://api.chainstream.io/x402/purchase?plan=nano");
```

Required packages: `@x402/core`, `@x402/fetch`, `@x402/evm` (for Base), `@x402/svm` (for Solana)

### Why manual curl does NOT work

The `Payment-Signature` header requires an EIP-712 typed data signature (`signTypedData`) over a `transferWithAuthorization` struct — not a raw message signature. This involves:
- EIP-712 domain separator (USDC contract name, version, chainId, verifyingContract)
- Typed struct: `TransferWithAuthorization(address from, address to, uint256 value, uint256 validAfter, uint256 validBefore, bytes32 nonce)`
- The facilitator submits the signed authorization on-chain (gasless for the user)

None of this can be done with `curl` or basic `signMessage`.

## Plans

Plans and pricing are dynamic. **Always fetch the latest from the API — do NOT hardcode plan names or prices.**

```bash
# CLI: view all available plans
npx @chainstream-io/cli wallet pricing

# API (no auth required)
curl https://api.chainstream.io/x402/pricing
```

### Agent behavior when payment is needed

When a user needs to purchase a subscription (wallet has no active plan or insufficient USDC for auto-pay):

1. **Fetch available plans**: run `npx @chainstream-io/cli wallet pricing` or call `GET /x402/pricing`
2. **Present ALL plans to the user** with name, price, quota, and duration — let them choose
3. **Explain what it means**: "A subscription gives you a pool of compute units (CU). Each API call consumes CU from the pool — the amount varies by endpoint and response size. The quota is valid for 30 days."
4. **After user chooses**: ensure wallet has enough USDC on Base or Solana, then run the CLI command — payment happens automatically on retry

Do NOT just say "you need the nano plan". Always show all options and let the user decide.

The CLI always prompts for plan selection — there is no default plan. Every purchase requires the user to choose.

## Supported Payment Networks

- **Base**: Base mainnet — USDC (`0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913`)
- **Solana**: Solana mainnet — USDC (`EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v`)

CLI auto-payment supports both Base and Solana. The default payment chain is Base; set `walletChain: "sol"` in config to use Solana.

## Error Recovery

- If payment fails (insufficient USDC): tell user how much USDC is needed and on which network (Base or Solana), show wallet address for funding
- If 402 persists after payment: wait 5s for settlement, then retry
- To check available plans: `npx @chainstream-io/cli wallet pricing` or `GET /x402/pricing` (no auth required)
