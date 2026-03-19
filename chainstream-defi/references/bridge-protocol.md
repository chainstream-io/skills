# Bridge Protocol Reference

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| POST | `/v2/bridge/quote` | Get bridge quote (source → destination) |
| POST | `/v2/bridge/execute` | Build unsigned bridge transaction |
| GET | `/v2/bridge/status/:txHash` | Bridge transaction status |

## Supported Bridge Paths

| Source → Destination | Protocol |
|---------------------|----------|
| Solana ↔ Base | LiFi, FlowBridge |
| Solana ↔ BSC | LiFi |
| Solana ↔ Ethereum | LiFi |
| BSC ↔ Base | LiFi |
| Ethereum ↔ Base | LiFi |
| Ethereum ↔ BSC | LiFi |

## Bridge Flow

Same four-phase protocol as swap:
1. Quote: `POST /v2/bridge/quote` → show fees, estimated time, route
2. Confirm: user approval required
3. Sign: unsigned tx → local signing → send
4. Poll: bridge completion (may take 1-30 minutes)

Bridge transactions typically take longer than swaps due to cross-chain finality.
