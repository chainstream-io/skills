# ChainStream Data API Endpoints

Complete reference for all `/v2/*` Data API endpoints. Base URL: `https://api.chainstream.io`

## Blockchain

| Method | Path | Description |
|--------|------|-------------|
| GET | `/v2/blockchain` | List supported blockchains |
| GET | `/v2/blockchain/{chain}/latest_block` | Latest block number and timestamp |

## Token

| Method | Path | Description |
|--------|------|-------------|
| GET | `/v2/token/search` | Search tokens by keyword (name/symbol/address). Params: `keyword`, `chain`, `limit` |
| GET | `/v2/token/{chain}/list` | Paginated token list. Params: `offset`, `limit`, `sort` |
| GET | `/v2/token/{chain}/multi` | Batch get tokens. Params: `addresses` (comma-separated) |
| GET | `/v2/token/{chain}/{tokenAddress}` | Full token detail |
| GET | `/v2/token/{chain}/{tokenAddress}/metadata` | Token metadata (name, symbol, decimals, image) |
| GET | `/v2/token/{chain}/metadata/multi` | Batch metadata |
| GET | `/v2/token/{chain}/{tokenAddress}/marketData` | Market data (price, volume, market cap, price changes) |
| GET | `/v2/token/{chain}/marketData/multi` | Batch market data |
| GET | `/v2/token/{chain}/{tokenAddress}/stats` | Trading stats (buys, sells, volume by time window) |
| GET | `/v2/token/{chain}/stats/multi` | Batch stats |
| GET | `/v2/token/{chain}/{tokenAddress}/price` | Current price. Params: `timestamp` for historical |
| GET | `/v2/token/{chain}/{tokenAddress}/prices` | Price history array |
| GET | `/v2/token/{chain}/{tokenAddress}/candles` | OHLCV candlestick. Params: `resolution` (1m/5m/15m/1h/4h/1d), `from`, `to`, `limit` |
| GET | `/v2/token/{chain}/pair/{pair}/candles` | Candles by trading pair |
| GET | `/v2/token/{chain}/pool/{poolAddress}/candles` | Candles by pool address |
| GET | `/v2/token/{chain}/{tokenAddress}/holders` | Holder list with balance and percentage |
| GET | `/v2/token/{chain}/{tokenAddress}/holders/multi` | Batch holders |
| GET | `/v2/token/{chain}/{tokenAddress}/topHolders` | Top N holders by balance |
| GET | `/v2/token/{chain}/{tokenAddress}/pools` | Liquidity pools containing this token |
| GET | `/v2/token/{chain}/{tokenAddress}/liquiditySnapshots` | Historical liquidity snapshots |
| GET | `/v2/token/{chain}/{tokenAddress}/creation` | Token creation info (deployer, timestamp, initial supply) |
| GET | `/v2/token/{chain}/{tokenAddress}/security` | Security check (honeypot, mint auth, freeze auth, holder concentration) |
| GET | `/v2/token/{chain}/{tokenAddress}/transfers` | Transfer history |
| GET | `/v2/token/{chain}/{tokenAddress}/transfer-total` | Transfer totals |
| GET | `/v2/token/{chain}/{tokenAddress}/mintAndBurn` | Mint and burn events |
| GET | `/v2/token/{chain}/{tokenAddress}/traders/{tag}` | Traders by tag (smart_money, whale, sniper, etc.) |
| GET | `/v2/token/{chain}/dev/{devAddress}/tokens` | Tokens created by a developer address |

## Trade

| Method | Path | Description |
|--------|------|-------------|
| GET | `/v2/trade/{chain}` | Recent trades. Params: `tokenAddress`, `walletAddress`, `limit` |
| GET | `/v2/trade/{chain}/activities` | Trade activities with enriched data |
| GET | `/v2/trade/{chain}/top-traders` | Top traders ranked by PnL or volume |
| GET | `/v2/trade/{chain}/trader-gainers-losers` | Traders sorted by gains/losses |

## Wallet

| Method | Path | Description |
|--------|------|-------------|
| GET | `/v2/wallet/{chain}/first-tx` | Wallet's first transaction timestamp |
| GET | `/v2/wallet/{chain}/net-worth-summary` | Aggregate net worth summary |
| GET | `/v2/wallet/{chain}/pnl-by-wallet` | PnL ranked by wallet |
| GET | `/v2/wallet/{chain}/{walletAddress}/pnl` | Wallet PnL summary |
| POST | `/v2/wallet/{chain}/{walletAddress}/calculate-pnl` | Trigger async PnL calculation job |
| GET | `/v2/wallet/{chain}/{walletAddress}/pnl-details` | Detailed PnL breakdown |
| GET | `/v2/wallet/{chain}/{walletAddress}/pnl-by-token` | PnL per token held |
| GET | `/v2/wallet/{chain}/{walletAddress}/net-worth` | Total net worth |
| GET | `/v2/wallet/{chain}/{walletAddress}/net-worth-chart` | Net worth over time |
| GET | `/v2/wallet/{chain}/{walletAddress}/net-worth-details` | Net worth breakdown |
| GET | `/v2/wallet/{chain}/{walletAddress}/net-worth/tokens` | Net worth by individual tokens |
| GET | `/v2/wallet/{chain}/{walletAddress}/tokens-balance` | Token balances |
| GET | `/v2/wallet/{chain}/{walletAddress}/transfers` | Transfer history |
| GET | `/v2/wallet/{chain}/{walletAddress}/transfer-total` | Transfer totals |
| GET | `/v2/wallet/{chain}/{walletAddress}/balance-updates` | Recent balance changes |

## DexPool

| Method | Path | Description |
|--------|------|-------------|
| GET | `/v2/dexpools/{chain}/{poolAddress}` | Pool details (tokens, reserves, TVL, fees) |
| GET | `/v2/dexpools/{chain}/{poolAddress}/snapshots` | Historical pool snapshots |

## Dex

| Method | Path | Description |
|--------|------|-------------|
| GET | `/v2/dex` | List DEXes. Params: `chains`, `dexProgram` |

## Ranking

| Method | Path | Description |
|--------|------|-------------|
| GET | `/v2/ranking/{chain}/hotTokens/{duration}` | Hot tokens. Duration: `1h`, `6h`, `24h` |
| GET | `/v2/ranking/{chain}/newTokens` | Newly created tokens |
| GET | `/v2/ranking/{chain}/finalStretch` | Tokens about to graduate (bonding curve near completion) |
| GET | `/v2/ranking/{chain}/migrated` | Tokens that migrated to DEX from launchpad |
| GET | `/v2/ranking/{chain}/stocks` | Stock-type tokens (real-world asset backed) |

## KYT (Know Your Transaction)

| Method | Path | Description |
|--------|------|-------------|
| POST | `/v2/kyt/address` | Register address for risk assessment |
| GET | `/v2/kyt/addresses/{address}/risk` | Get address risk score and categories |
| POST | `/v2/kyt/transfer` | Register transfer for compliance screening |
| GET | `/v2/kyt/transfers/{transferId}/summary` | Transfer risk summary |
| GET | `/v2/kyt/transfers/{transferId}/alerts` | Transfer compliance alerts |
| GET | `/v2/kyt/transfers/{transferId}/exposures/direct` | Direct exposure to risky entities |
| GET | `/v2/kyt/transfers/{transferId}/network-identifications` | Network identification results |
| POST | `/v2/kyt/withdrawal` | Register withdrawal for screening |
| GET | `/v2/kyt/withdrawal/{withdrawalId}/summary` | Withdrawal risk summary |
| GET | `/v2/kyt/withdrawal/{withdrawalId}/alerts` | Withdrawal alerts |
| GET | `/v2/kyt/withdrawal/{withdrawalId}/exposures/direct` | Withdrawal direct exposure |
| GET | `/v2/kyt/withdrawal/{withdrawalId}/address-identifications` | Address identification results |
| GET | `/v2/kyt/withdrawal/{withdrawalId}/network-identifications` | Network identification results |
| GET | `/v2/kyt/withdrawal/{withdrawalId}/fraud-assessment` | Fraud assessment score |

## Webhook

| Method | Path | Description |
|--------|------|-------------|
| GET | `/v2/webhook/endpoint` | List webhook endpoints |
| POST | `/v2/webhook/endpoint` | Create endpoint. Body: `url`, `events`, `description` |
| PATCH | `/v2/webhook/endpoint` | Update endpoint |
| GET | `/v2/webhook/endpoint/{id}` | Get endpoint details |
| DELETE | `/v2/webhook/endpoint/{id}` | Delete endpoint |
| GET | `/v2/webhook/endpoint/{id}/secret` | Get endpoint signing secret |
| POST | `/v2/webhook/endpoint/{id}/secret/rotate` | Rotate signing secret |

## RedPacket

| Method | Path | Description |
|--------|------|-------------|
| POST | `/v2/redpacket/{chain}/create` | Create a red packet |
| POST | `/v2/redpacket/{chain}/claim` | Claim a red packet |
| POST | `/v2/redpacket/{chain}/send` | Send a red packet |
| GET | `/v2/redpacket` | List red packets |
| GET | `/v2/redpacket/{id}` | Red packet details |
| GET | `/v2/redpacket/{id}/claims` | Claim records for a red packet |
| GET | `/v2/redpacket/wallet/{address}/claims` | Claims by wallet |
| GET | `/v2/redpacket/wallet/{address}/redpackets` | Red packets by wallet |

## IPFS

| Method | Path | Description |
|--------|------|-------------|
| POST | `/v2/ipfs/presign` | Get presigned URL for IPFS upload (Pinata) |

## Watchlist

| Method | Path | Description |
|--------|------|-------------|
| POST | `/v2/watchlist/{chain}/{walletAddress}` | Add wallet to watchlist |
