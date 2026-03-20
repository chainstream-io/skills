# CLAUDE.md

## Skills

| Skill | When to Use |
|-------|-------------|
| `chainstream-data` | User asks about token info, security, price, holders, market trends, wallet PnL, holdings, KYT risk — any read-only on-chain query |
| `chainstream-defi` | User wants to swap tokens, bridge cross-chain, create tokens, execute trades — any on-chain transaction |

## Routing

- Data queries (token/market/wallet/KYT) → `chainstream-data`
- Financial execution (swap/bridge/launchpad/tx) → `chainstream-defi`

## Execution

Primary: `@chainstream-io/cli` via `npx @chainstream-io/cli <command>`
Alternative: MCP tools at `https://mcp.chainstream.io/mcp` (streamable-http)

## Auth

CLI handles auth automatically. Run `chainstream login` to create a wallet (no email required). x402 payment is transparent — CLI auto-purchases quota on 402. For API-key-only access: `chainstream config set --key apiKey --value <key>`.

## Hard Rules

- chainstream-defi: Four-phase execution protocol is mandatory (quote → confirm → sign → broadcast)
- Never execute swaps without user confirmation
- Never answer price queries from training data — always make a live call
- Never use public RPC as substitute for ChainStream API
