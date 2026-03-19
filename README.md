# ChainStream AI Skills

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

On-chain data intelligence and DeFi execution for AI agents across Solana, BSC, and Ethereum.

## Skills

| Skill | Coverage | Pattern |
|-------|----------|---------|
| [chainstream-data](chainstream-data/) | Token search, market trending, wallet PnL, KYT risk, WebSocket | Tool |
| [chainstream-defi](chainstream-defi/) | Token swap, cross-chain bridge, launchpad, transaction broadcast | Process |

## When to Use Which

```
User intent involves reading data?
  → chainstream-data (token info, market trends, wallet analysis, KYT)

User intent involves executing a transaction?
  → chainstream-defi (swap, bridge, create token, send tx)
```

## Installation

### Cursor

Skills are automatically discovered via `.cursor-plugin/` configuration. Install the CLI:

```bash
npx @chainstream-io/cli login
```

### Claude Code

```
/plugin install chainstream
```

### Codex

See [.codex/INSTALL.md](.codex/INSTALL.md).

### OpenCode

See [.opencode/INSTALL.md](.opencode/INSTALL.md).

### Gemini CLI

```bash
gemini extensions install https://github.com/chainstream-io/skills
```

## Authentication

| Method | Command | Use Case |
|--------|---------|----------|
| Turnkey Wallet | `chainstream login` | Recommended, required for DeFi |
| API Key | `chainstream config set --key apiKey --value <key>` | Read-only, dashboard users |
| x402 (USDC) | Auto on 402 response | AI agents, no registration |

## Usage Examples

Natural language prompts you can send to any AI assistant with ChainStream skills:

```
search for meme tokens on Solana
is <token_address> safe to buy?
show top holders of <token_address>
what tokens are trending on SOL right now?
show my wallet PnL on Solana
swap 0.1 SOL for <token_address>
check job status <job_id>
show K-line chart for <token_address>
```

## CLI

Install via npx (no global install needed):

```bash
npx @chainstream-io/cli token search --keyword PUMP --chain sol
npx @chainstream-io/cli market trending --chain sol --duration 1h
npx @chainstream-io/cli wallet pnl --chain sol --address <addr>
npx @chainstream-io/cli dex quote --chain sol --input-token SOL --output-token <addr> --amount 1000000
```

## MCP Server

```json
{
  "mcpServers": {
    "chainstream": {
      "url": "https://mcp.chainstream.io/mcp",
      "transport": "streamable-http"
    }
  }
}
```

## Supported Chains

| Chain | ID | Data | DeFi | WebSocket |
|-------|----|------|------|-----------|
| Solana | `sol` | Yes | Yes | Yes |
| BSC | `bsc` | Yes | Yes | Yes |
| Ethereum | `eth` | Yes | Yes | Yes |

## Resources

- [Documentation](https://docs.chainstream.io)
- [Dashboard](https://app.chainstream.io)
- [CLI README](https://github.com/chainstream-io/cli)
- [API Reference](https://docs.chainstream.io/api-reference)

## License

[MIT](LICENSE)
