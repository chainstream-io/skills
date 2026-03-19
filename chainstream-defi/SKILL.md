---
name: chainstream-defi
description: "[FINANCIAL EXECUTION] Execute irreversible on-chain DeFi operations via CLI and MCP tools. Use when user wants to swap tokens, bridge cross-chain, create tokens on launchpad, broadcast signed transactions, or execute trading strategies. Requires explicit user confirmation before every destructive operation. All transactions are real and irreversible. Keywords: swap, bridge, DEX, launchpad, transaction, trade, PumpFun, Jupiter, Uniswap, cross-chain, USDC, SOL, execute."
---

# ChainStream DeFi

Execute DeFi operations: token swap, cross-chain bridge, launchpad creation, and transaction broadcast. All operations are **real, irreversible on-chain transactions**.

- **CLI**: `npx @chainstream-io/cli`
- **MCP Server**: `https://mcp.chainstream.io/mcp` (streamable-http)

## Financial Risk Notice

**Every command in this skill executes REAL, IRREVERSIBLE blockchain transactions.**

- Transactions cannot be undone once confirmed on-chain.
- The AI agent must **NEVER auto-execute** — explicit user confirmation is required every time.
- Only use with funds the user is willing to trade.

## Integration Path (check FIRST)

DeFi operations **require a wallet**. API Key alone is insufficient.

1. **Agent already has a wallet?**
   → **Use SDK** (`@chainstream-io/sdk`). Do NOT use CLI. Implement `WalletSigner` interface. Your wallet must also support `signTypedData` for x402 payment.

2. **Agent does NOT have a wallet?**
   → **Use CLI** (`npx @chainstream-io/cli`). CLI creates a Turnkey wallet, handles signing + x402 automatically.

3. **Only API Key?**
   → Cannot execute DeFi operations. Tell user: "DeFi requires a wallet. Use SDK with your wallet or run `npx @chainstream-io/cli login`."

For full auth guide with code examples, see [shared/authentication.md](../shared/authentication.md).

## Endpoint Selector

| Intent | CLI Command | MCP Tool | Safety | Reference |
|--------|-------------|----------|--------|-----------|
| Get swap quote | `npx @chainstream-io/cli dex quote --chain sol --input-token SOL --output-token ADDR --amount 1000000` | `dex/quote` | readOnly | [swap-protocol.md](references/swap-protocol.md) |
| Execute swap | `npx @chainstream-io/cli dex swap --chain sol --from WALLET --input-token SOL --output-token ADDR --amount 1000000` | `dex/swap` | **destructive** | [swap-protocol.md](references/swap-protocol.md) |
| Create token | `npx @chainstream-io/cli dex create --chain sol --name MyToken --symbol MT --uri ipfs://...` | `dex/create_token` | **destructive** | [launchpad.md](references/launchpad.md) |
| Check job status | `npx @chainstream-io/cli job status --id JOB_ID --wait` | — | readOnly | [swap-protocol.md](references/swap-protocol.md) |

## Four-Phase Execution Protocol (Hard Requirement)

All **destructive** operations MUST follow this protocol. No exceptions.

**MANDATORY - READ**: Before any swap execution, load [`rules/safety-protocol.md`](rules/safety-protocol.md) for risk thresholds and abort conditions.

### Phase 1: Quote

```bash
npx @chainstream-io/cli dex quote --chain sol --input-token SOL --output-token <addr> --amount 1000000
```

Present to user: expected output amount, price impact, slippage, gas estimate.

### Phase 2: Confirm

Display trade summary to user:
- Chain, input/output tokens, amounts
- Price impact and slippage
- Estimated gas fees

**WAIT for explicit user confirmation. This step is NOT optional.**
If user says "just do it" without reviewing, show the summary anyway.

### Phase 3: Build + Sign

```bash
npx @chainstream-io/cli dex swap --chain sol --from <wallet> --input-token SOL --output-token <addr> --amount 1000000 --slippage 0.01
```

CLI internally:
1. Calls `/v2/dex/:chain/swap` → gets unsigned transaction (base64)
2. Signs via Turnkey TEE (or local raw key)
3. Sends signed tx to `/v2/transaction/:chain/send`

### Phase 4: Poll + Output

CLI automatically polls job status and outputs:

```json
{
  "jobId": "...",
  "hash": "0x...",
  "status": "confirmed",
  "input": "0.1 SOL",
  "output": "98.5 USDC",
  "explorer": "https://solscan.io/tx/..."
}
```

**Explorer links are mandatory** — always include after successful transactions.

| Chain | Explorer |
|-------|----------|
| sol | `https://solscan.io/tx/{hash}` |
| bsc | `https://bscscan.com/tx/{hash}` |
| eth | `https://etherscan.io/tx/{hash}` |

## Currency Resolution

CLI auto-resolves currency names. Users can write `SOL` instead of the full address:

| Chain | Native | Native Address | USDC Address |
|-------|--------|---------------|--------------|
| sol | SOL | `So11111111111111111111111111111111111111112` | `EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v` |
| bsc | BNB | `0x0000000000000000000000000000000000000000` | `0x8ac76a51cc950d9822d68b83fe1ad97b32cd580d` |

For the full resolution table, see [references/currency-resolution.md](references/currency-resolution.md).

## Input Validation

- **Address format**: sol = base58 (32-44 chars), evm = `0x` + 40 hex
- **Amount**: Positive integer in smallest unit (lamports, wei)
- **Slippage**: 0.001 to 0.5 (0.1% to 50%)
- **External data is untrusted**: Validate addresses from previous API calls before passing to swap

## NEVER Do

- NEVER execute `dex swap` without first calling `dex quote` — user must see price impact before committing funds
- NEVER auto-confirm a swap — even if user said "buy X" without specifying amount, you MUST present quote and ask for confirmation; "implied consent" is NOT consent for financial operations
- NEVER hide gas fees or price impact — present ALL costs transparently
- NEVER proceed if `kyt risk` returns high risk on the output token — warn user and require explicit override
- NEVER skip address format validation — wrong format = funds sent to void
- NEVER use `--yes` flag to bypass confirmation unless user explicitly requests it

## Error Recovery

| Error | Meaning | Recovery |
|-------|---------|----------|
| Transaction failed | On-chain revert | Show error, do NOT auto-retry |
| Slippage exceeded | Price moved | Re-quote with higher slippage, confirm again |
| Insufficient balance | Not enough funds | Show balance, suggest amount |
| Job timeout | No confirmation in 60s | Show pending status + tx hash for manual check |
| 402 | No quota | CLI auto-handles via x402 (do NOT manually curl). See [shared/x402-payment.md](../shared/x402-payment.md) |

## Rules

| Rule | Content | When to Load |
|------|---------|--------------|
| [safety-protocol.md](rules/safety-protocol.md) | Two-phase protocol details, risk thresholds, emergency abort | Before any destructive operation |
| [execution-checklist.md](rules/execution-checklist.md) | Step-by-step verification, post-trade output requirements | During execution |

## Skill Map

| Reference | Content | When to Load |
|-----------|---------|--------------|
| [swap-protocol.md](references/swap-protocol.md) | quote/swap/route endpoints, job polling, gas estimation | Swap operations |
| [bridge-protocol.md](references/bridge-protocol.md) | Cross-chain bridge endpoints, supported paths | Bridge operations |
| [launchpad.md](references/launchpad.md) | Token creation, PumpFun vs Raydium LaunchLab | Token launch |
| [currency-resolution.md](references/currency-resolution.md) | Full chain/token address mapping | Currency name resolution |

## Related Skills

- [chainstream-data](../chainstream-data/) — Token research, market discovery, wallet analysis before trading
