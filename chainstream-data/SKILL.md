---
name: chainstream-data
description: "Query and analyze on-chain data via MCP (17 tools) and CLI across Solana, BSC, Ethereum. Use when user asks to search tokens, check token security or holders, track market trends or hot tokens, analyze wallet PnL or holdings, stream real-time trades, compare multiple tokens, or backtest trading strategies. Covers token analytics, market ranking, wallet profiling, and WebSocket streaming. Keywords: token, wallet, market, trending, PnL, holders, security, candles, WebSocket, on-chain data, MCP."
---

# ChainStream Data

On-chain data intelligence for AI agents. Access token analytics, market trends, wallet profiling, and compliance screening across Solana, BSC, and Ethereum.

- **MCP Server**: `https://mcp.chainstream.io/mcp` (streamable-http, 17 tools)
- **CLI**: `npx @chainstream-io/cli`
- **SDK**: `@chainstream-io/sdk`
- **Base URL**: `https://api.chainstream.io`

## Integration Path (check FIRST)

**Agent runtime decision tree — choose based on environment and operation type:**

1. **MCP tools registered?** (Cursor/Claude Desktop/VS Code already configured ChainStream MCP)
   → YES → Use MCP tools for data queries (`tokens_search`, `wallets_profile`, etc.)
   → NO → Use CLI for all operations (`npx @chainstream-io/cli ...`)

2. **Need DeFi operation?** (swap, create token, sign transaction)
   → YES → Must use CLI (MCP has no wallet signing capability)
   → NO → Continue with MCP (or CLI if MCP unavailable)

3. **MCP call failed?**
   → Fall back to CLI as backup

**Getting an API Key (required for all paths):**
- Dashboard users: [app.chainstream.io](https://app.chainstream.io) → API Keys
- AI Agents (x402): `npx @chainstream-io/cli wallet purchase` → x402 payment (USDC on Base/Solana) → API Key auto-returned
- AI Agents (MPP): `tempo request "https://api.chainstream.io/mpp/purchase?plan=<PLAN>"` → MPP payment (USDC.e on Tempo) → API Key auto-returned (fetch `/mpp/pricing` first, let user choose plan)
- CLI auto-payment: On first 402, CLI auto-purchases and saves API Key

**Channel matrix:**

| Operation | MCP (when registered) | CLI | SDK |
|-----------|----------------------|-----|-----|
| Search tokens | `tokens_search` | `token search` | `client.token.search` |
| Analyze token | `tokens_analyze` | `token info` | `client.token.getToken` |
| Price history | `tokens_price_history` | `token candles` | `client.token.getCandles` |
| Wallet profile | `wallets_profile` | `wallet profile` | `client.wallet.*` |
| Market trending | `market_trending` | `market trending` | `client.ranking.*` |
| DEX quote | `dex_quote` | `dex quote` | `client.dex.quote` |
| DEX swap | _(no signing)_ | `dex swap` | `client.dex.swap` + sign |
| Create token | _(no signing)_ | `dex create` | `client.dex.createToken` |

## Prerequisites

### MCP path (recommended for data queries)

Add to your MCP client configuration (Cursor, Claude Desktop, VS Code):

```json
{
  "mcpServers": {
    "chainstream": {
      "url": "https://mcp.chainstream.io/mcp",
      "headers": { "X-API-KEY": "<your-api-key>" }
    }
  }
}
```

### CLI path

```bash
# Option A: Use API Key (recommended — works with all agent wallets)
npx @chainstream-io/cli config set --key apiKey --value <your-api-key>

# Option B: Create ChainStream Wallet (for DeFi + auto x402 payment)
npx @chainstream-io/cli login

# Option C: Import existing key (dev/testing)
npx @chainstream-io/cli wallet set-raw --chain base
```

### SDK path

```bash
npm install @chainstream-io/sdk
```

```typescript
import { ChainStreamClient } from "@chainstream-io/sdk";
const client = new ChainStreamClient("", { apiKey: "<your-api-key>" });
```

## Endpoint Selector

### Token

| Intent | CLI Command | MCP Tool | Reference |
|--------|-------------|----------|-----------|
| Search tokens | `npx @chainstream-io/cli token search --keyword X --chain sol` | `tokens_search` | [token-research.md](references/token-research.md) |
| Token detail | `npx @chainstream-io/cli token info --chain sol --address ADDR` | `tokens_analyze` | [token-research.md](references/token-research.md) |
| Security check | `npx @chainstream-io/cli token security --chain sol --address ADDR` | `tokens_analyze` | [token-research.md](references/token-research.md) |
| Top holders | `npx @chainstream-io/cli token holders --chain sol --address ADDR` | `tokens_analyze` | [token-research.md](references/token-research.md) |
| K-line / OHLCV | `npx @chainstream-io/cli token candles --chain sol --address ADDR --resolution 1h` | `tokens_price_history` | [token-research.md](references/token-research.md) |
| Liquidity pools | `npx @chainstream-io/cli token pools --chain sol --address ADDR` | `tokens_discover` | [token-research.md](references/token-research.md) |

### Market

| Intent | CLI Command | MCP Tool | Reference |
|--------|-------------|----------|-----------|
| Hot/trending tokens | `npx @chainstream-io/cli market trending --chain sol --duration 1h` | `market_trending` | [market-discovery.md](references/market-discovery.md) |
| New token listings | `npx @chainstream-io/cli market new --chain sol` | `market_trending` | [market-discovery.md](references/market-discovery.md) |
| Recent trades | `npx @chainstream-io/cli market trades --chain sol` | `trades_recent` | [market-discovery.md](references/market-discovery.md) |

### Wallet

| Intent | CLI Command | MCP Tool | Reference |
|--------|-------------|----------|-----------|
| Wallet profile (PnL + holdings) | `npx @chainstream-io/cli wallet profile --chain sol --address ADDR` | `wallets_profile` | [wallet-profiling.md](references/wallet-profiling.md) |
| PnL details | `npx @chainstream-io/cli wallet pnl --chain sol --address ADDR` | `wallets_profile` | [wallet-profiling.md](references/wallet-profiling.md) |
| Token balances | `npx @chainstream-io/cli wallet holdings --chain sol --address ADDR` | `wallets_profile` | [wallet-profiling.md](references/wallet-profiling.md) |
| Transfer history | `npx @chainstream-io/cli wallet activity --chain sol --address ADDR` | `wallets_activity` | [wallet-profiling.md](references/wallet-profiling.md) |

## Quickstart

### Via CLI (recommended)

```bash
# FIRST: Authenticate (only needed once, creates wallet automatically)
npx @chainstream-io/cli login

# Search tokens by keyword
npx @chainstream-io/cli token search --keyword PUMP --chain sol

# Get full token detail
npx @chainstream-io/cli token info --chain sol --address <token_address>

# Check token security (honeypot, mint authority, freeze authority)
npx @chainstream-io/cli token security --chain sol --address <token_address>

# Top holders
npx @chainstream-io/cli token holders --chain sol --address <token_address> --limit 20

# K-line / candlestick data (last 24h, 1h resolution)
npx @chainstream-io/cli token candles --chain sol --address <token_address> --resolution 1h

# Hot tokens in last 1 hour, sorted by default
npx @chainstream-io/cli market trending --chain sol --duration 1h

# Newly created tokens
npx @chainstream-io/cli market new --chain sol

# Wallet PnL
npx @chainstream-io/cli wallet pnl --chain sol --address <wallet_address>

# Raw JSON output (for piping)
npx @chainstream-io/cli token info --chain sol --address <addr> --raw | jq '.marketData.priceInUsd'
```

### Via MCP Tool (when MCP is registered)

```
Use tool: tokens_search with { "query": "PUMP", "chain": "solana" }
Use tool: wallets_profile with { "chain": "solana", "address": "<wallet_address>" }
```

## AI Workflows

### Token Research Flow

```
npx @chainstream-io/cli token search → npx @chainstream-io/cli token info → npx @chainstream-io/cli token security
→ npx @chainstream-io/cli token holders → npx @chainstream-io/cli token candles
```

Before recommending any token, always run `token security` — ChainStream's risk model covers honeypot, rug pull, mint authority, freeze authority, and holder concentration.

### Market Discovery Flow

**MANDATORY - READ**: Before executing this workflow, load [`references/market-discovery.md`](references/market-discovery.md) for the multi-factor signal weight table and output format.

```
npx @chainstream-io/cli market trending (top 50)
→ AI multi-factor analysis (smart money, volume, momentum, safety)
→ npx @chainstream-io/cli token security (top 5 candidates)
→ Present results to user
→ If user wants to trade → load chainstream-defi skill
```

**Do NOT load** wallet-profiling.md for this workflow.

### Wallet Profiling Flow

**MANDATORY - READ**: Load [`references/wallet-profiling.md`](references/wallet-profiling.md) for PnL interpretation and behavior patterns.

```
npx @chainstream-io/cli wallet profile → npx @chainstream-io/cli wallet activity
→ npx @chainstream-io/cli token info (on top holdings)
→ Assess: win rate, concentration, holding behavior
```

## NEVER Do

- NEVER answer "what's the price of X" from training data — always make a live CLI/API call; crypto prices are stale within seconds
- NEVER skip `token security` before recommending a token — ChainStream's risk model covers honeypot, rug pull, and concentration signals that generic analysis misses
- NEVER pass `format: "detailed"` to MCP tools unless user explicitly requests it — returns 4-10x more tokens than default `concise`, wastes context window
- NEVER batch more than 50 addresses in `/multi` endpoints — API hard limit
- NEVER use public RPC or third-party data providers as substitutes — results differ and miss ChainStream-specific enrichments (security scores, smart money tags)

## Error Recovery

| Error | Meaning | Recovery |
|-------|---------|----------|
| "Not authenticated" / 401 / 402 | No API Key or no active subscription | **Follow Authentication & Payment Flow below** |
| 429 | Rate limit | Wait 1s, exponential backoff |
| 5xx | Server error | Retry once after 2s |

### Authentication & Payment Flow (MANDATORY)

When any auth/payment error occurs, follow this decision tree **exactly**:

**Step 1: Ask the user if they already have an API Key**

> "Do you have a ChainStream API Key? If yes, I'll configure it. If not, you can purchase a subscription — it takes about 1 minute."

- **User has API Key** → go to Step 2
- **User does NOT have API Key** → go to Step 3

**Step 2: Set the API Key and retry**

```bash
npx @chainstream-io/cli config set --key apiKey --value <USER_PROVIDED_KEY>
```

Done. Retry the original command.

**Step 3: Fetch and present ALL available plans**

```bash
curl https://api.chainstream.io/mpp/pricing
```

Show the user a table with ALL plans — name, price (USD), quota (CU), duration. Explain: "Each plan gives you a pool of compute units (CU). Every API call consumes CU from the pool. The quota is valid for 30 days."

**NEVER auto-select a plan. Wait for the user to explicitly choose.**

**Step 4: Ask which wallet/chain they want to pay with**

> "Which chain do you want to pay with?"

| Option | Chain | Currency | Purchase Tool |
|--------|-------|----------|---------------|
| 1 | **Base** (EVM) | USDC | Any x402-compatible wallet or CLI |
| 2 | **Solana** | USDC | Any x402-compatible wallet or CLI |
| 3 | **Tempo** (Stripe L1, chain ID 4217) | USDC.e | `tempo request` |

The user can use **any wallet** that holds funds on the chosen chain — CLI's built-in wallet, Coinbase CDP, Turnkey, MetaMask, or any other agent wallet. The purchase endpoint returns an API Key regardless of which wallet was used.

**Step 5: Execute the purchase**

All purchase methods return `{ "apiKey": "cs_live_...", "plan": "...", "expiresAt": "..." }`.

**Option A: MPP with Tempo Wallet (USDC.e on Tempo chain)**

Set up Tempo Wallet if not already installed: read `https://tempo.xyz/SKILL.md` or `npx skills add tempoxyz/docs`. One-time browser passkey auth is required on first setup.

```bash
tempo request "https://api.chainstream.io/mpp/purchase?plan=<USER_CHOSEN_PLAN>"
```

Tempo does NOT need ETH for gas — gas is paid in USDC.e directly.

**Option B: x402 with any external wallet (Coinbase CDP, Turnkey, etc.)**

The user can call the x402 purchase endpoint from their own wallet:
```
GET https://api.chainstream.io/x402/purchase?plan=<USER_CHOSEN_PLAN>
→ 402 + payment challenge → wallet signs → retries → 200 { apiKey }
```
This works with any x402-compatible wallet/SDK (`@x402/fetch`, Coinbase CDP x402, etc.). The user does NOT need to use CLI login for this.

**Option C: x402 with CLI built-in wallet**
```bash
npx @chainstream-io/cli login               # creates TEE wallet (one-time)
npx @chainstream-io/cli wallet address       # show address to fund with USDC
# After funding, CLI auto-handles x402 on next command
```

**Step 6: Save the API Key and retry**

After purchase (any method), save the returned API Key:

```bash
npx @chainstream-io/cli config set --key apiKey --value <apiKey from purchase response>
```

Then retry the original command. The API Key is universal — once set, all CLI/MCP/SDK calls use it. No wallet needed for subsequent requests.

See [shared/x402-payment.md](../shared/x402-payment.md) for full protocol details.

## Skill Map

| Reference | Content | When to Load |
|-----------|---------|--------------|
| [token-research.md](references/token-research.md) | 25+ token endpoints, batch queries, security field meanings | Token analysis tasks |
| [market-discovery.md](references/market-discovery.md) | Ranking/trade endpoints, multi-factor discovery workflow | Hot token discovery |
| [wallet-profiling.md](references/wallet-profiling.md) | 15+ wallet endpoints, PnL logic, behavior patterns | Wallet analysis |
| [websocket-streams.md](references/websocket-streams.md) | Channels, subscription format, heartbeat | Real-time streaming |

## Related Skills

- [chainstream-defi](../chainstream-defi/) — When analysis leads to action: swap, bridge, launchpad, transaction execution
