# ChainStream GraphQL Query Patterns

Ready-to-use query templates for common blockchain analytics scenarios. Replace placeholder values (`TOKEN_ADDRESS`, `WALLET_ADDRESS`, etc.) with actual addresses.

All queries use `network: sol` (Solana) by default. Change to `eth` or `bsc` as needed.

## Token Discovery & Analytics

### Trending Tokens by Volume (TokenTradeStats)

```graphql
query {
  TokenTradeStats(
    network: sol
    limit: {count: 50}
    orderBy: VolumeUSD_DESC
    where: {Block: {Time: {since: "2026-03-30 00:00:00"}}}
  ) {
    Token { Address }
    VolumeUSD TradeCount BuyCount SellCount
    UniqueBuyers UniqueSellers
    joinToken { Token { Name Symbol ImageUrl } }
  }
}
```

### Latest Price (via OHLC)

```graphql
query {
  OHLC(network: sol, limit: {count: 1}, tokenAddress: {is: "So11111111111111111111111111111111111111112"}) {
    TimeMinute
    Token { Address }
    Price { Close }
    VolumeUSD
  }
}
```

### Token Search (metadata + stats)

```graphql
query {
  TokenSearch(network: sol, tokenAddress: {is: "TOKEN_ADDRESS"}, limit: {count: 1}) {
    Token {
      Address Decimals PriceUSD MarketCapUSD
      TotalTradeCount TotalVolumeUSD HolderCount PoolCount
    }
  }
}
```

## DEX Trades

### Latest Trades

```graphql
query {
  DEXTrades(network: sol, limit: {count: 25}, orderBy: Block_Time_DESC) {
    Block { Time }
    Trade {
      Buy { Currency { MintAddress Symbol Name } Amount PriceInUSD }
      Sell { Currency { MintAddress Symbol Name } Amount }
      Dex { ProtocolName ProtocolFamily }
    }
  }
}
```

### Trades for a Specific Token

```graphql
query {
  DEXTrades(network: sol, limit: {count: 25}, tokenAddress: {is: "TOKEN_ADDRESS"}, orderBy: Block_Time_DESC) {
    Block { Time }
    Trade {
      Buy { Amount PriceInUSD Account { Owner } }
      Sell { Currency { MintAddress } Amount }
      Dex { ProtocolName }
    }
    Pool { Address }
  }
}
```

### Trades with Token Images (joinBuyToken)

Symbol and Name are now inline in `Currency`. Use `joinBuyToken`/`joinSellToken` only when you need extra TokenSearch fields like `ImageUrl`.

```graphql
query {
  DEXTrades(network: sol, limit: {count: 25}, orderBy: Block_Time_DESC) {
    Block { Time }
    Trade {
      Buy { Currency { MintAddress Symbol Name } Amount PriceInUSD }
      Sell { Currency { MintAddress Symbol Name } Amount }
    }
    joinBuyToken { Token { ImageUrl } }
  }
}
```

### Top Traders by Volume

```graphql
query {
  DEXTrades(
    network: sol
    limit: {count: 100}
    tokenAddress: {is: "TOKEN_ADDRESS"}
    where: {IsSuspect: {eq: false}}
  ) {
    Trade { Buy { Account { Owner } Amount PriceInUSD } }
    count
    sum(of: Trade_Buy_Amount)
  }
}
```

### pump.fun Trades (Last 7 Days)

```graphql
query {
  DEXTrades(
    network: sol
    where: {
      Block: {Time: {since: "2026-03-24 00:00:00"}}
      Trade: {Dex: {ProgramAddress: {is: "6EF8rrecthR5Dkzon8Nwu78hRvfCKubJ14M5uBEwF6P"}}}
    }
    limit: {count: 1000}
    orderBy: Block_Time_DESC
  ) {
    Block { Time }
    Trade {
      Buy { Currency { MintAddress } Amount }
      Sell { Currency { MintAddress } Amount }
      Dex { ProtocolName }
    }
  }
}
```

### Trade Count (Aggregation Only)

```graphql
query {
  DEXTrades(network: sol, where: {Block: {Time: {since: "2026-03-24 00:00:00"}}}) {
    count
  }
}
```

## OHLCV / K-Line

### K-Line Data for a Token

```graphql
query {
  OHLC(network: sol, limit: {count: 24}, tokenAddress: {is: "TOKEN_ADDRESS"}) {
    TimeMinute
    Token { Address }
    Price { Open High Low Close }
    VolumeUSD
    TradeCount
  }
}
```

### Trade Statistics (Buy/Sell breakdown)

```graphql
query {
  TokenTradeStats(network: sol, limit: {count: 24}, tokenAddress: {is: "TOKEN_ADDRESS"}) {
    TimeMinute
    Token { Address }
    TradeCount BuyCount SellCount
    VolumeUSD UniqueBuyers UniqueSellers
  }
}
```

## Wallet Analysis

### Wallet PnL by Token

```graphql
query {
  WalletTokenPnL(network: sol, limit: {count: 20}, walletAddress: {is: "WALLET_ADDRESS"}) {
    Wallet { Address }
    Token { Address }
    BuyVolumeUSD SellVolumeUSD
    BuyCount SellCount
    FirstTrade LastTrade
  }
}
```

> **Token Holders**: Top-holder data is available via the REST API (`chainstream-data` skill), not through GraphQL.

## Transfers

### Latest Transfers

```graphql
query {
  Transfers(network: sol, limit: {count: 20}, orderBy: Block_Time_DESC) {
    Block { Time }
    Transaction { Hash }
    Transfer {
      Currency { MintAddress }
      Sender { Address }
      Receiver { Address }
      Amount AmountInUSD
    }
  }
}
```

### Wallet Outgoing Transfers

```graphql
query {
  Transfers(network: sol, limit: {count: 20}, senderAddress: {is: "WALLET_ADDRESS"}, orderBy: Block_Time_DESC) {
    Block { Time }
    Transfer {
      Currency { MintAddress }
      Receiver { Address }
      Amount AmountInUSD
    }
  }
}
```

## Transactions (with Joins)

### Solana Transaction with Full Details

```graphql
query {
  SolTransactions(network: sol, limit: {count: 1}, orderBy: Block_Time_DESC) {
    Block { Time Slot Height Hash }
    Signature TxIndex Success Fee FeeInNative FeePayer Signer
    joinInstructions {
      Transaction { Signature Index InnerIndex NestingLevel Program { Id Name MethodName } }
    }
    joinDEXTrades {
      Trade { Buy { Currency { MintAddress } Amount PriceInUSD } Sell { Currency { MintAddress } Amount } }
    }
    joinTransfers {
      Transfer { Currency { MintAddress } Amount AmountInUSD }
    }
  }
}
```

### EVM Transaction with Logs and Traces

```graphql
query {
  EVMTransactions(network: eth, limit: {count: 1}, orderBy: Block_Time_DESC) {
    Block { Time Height Hash }
    Hash From To Success Gas { Used Price }
    joinLogs { LogIndex Address EventName Topics { Topic0 Topic1 } Data }
    joinTraces { TraceIndex CallType FromAddress ToAddress ValueInNative GasUsed }
  }
}
```

## Pools & Liquidity

### DEX Pools for a Token

```graphql
query {
  DEXPools(network: sol, limit: {count: 20}, tokenAddress: {is: "TOKEN_ADDRESS"}) {
    Pool { Address ProgramAddress }
    TokenA { Address }
    TokenB { Address }
    Dex { ProtocolName }
  }
}
```

## DEXTradeByTokens (Token-Centric)

### All Trades for a Specific Token

Token-centric view: both buy and sell sides appear as rows with the queried token as the primary `Currency`.

```graphql
query {
  DEXTradeByTokens(
    network: sol
    tokenAddress: {is: "TOKEN_ADDRESS"}
    limit: {count: 50}
    orderBy: Block_Time_DESC
  ) {
    Block { Time }
    Trade {
      Currency { MintAddress Symbol Name }
      Amount AmountInUSD
      Side { Type Currency { MintAddress Symbol } Amount AmountInUSD }
      Dex { ProtocolName ProtocolFamily }
    }
  }
}
```

### Buy vs Sell Volume Analysis

```graphql
query {
  DEXTradeByTokens(
    network: sol
    tokenAddress: {is: "TOKEN_ADDRESS"}
    where: {Block: {Time: {since: "2026-03-24 00:00:00"}}}
  ) {
    Trade { Side { Type } }
    count
    sum(of: Trade_AmountInUSD)
  }
}
```

## Blocks

### Latest Blocks

```graphql
query {
  Blocks(network: sol, limit: {count: 10}, orderBy: Block_Time_DESC) {
    Block {
      Time Slot Number Hash TxCount TotalFee
    }
  }
}
```

## Pattern Reference: Data Shape → Chart Type

When analyzing query results, use this mapping to choose appropriate visualization:

| Data Pattern | Recommended Chart | Example Query |
|-------------|------------------|---------------|
| Time series (price, volume over time) | Line / Candlestick | OHLC, TokenTradeStats |
| Ranking (top tokens by volume) | Bar chart / Leaderboard | TokenTradeStats, DEXTradeByTokens |
| Distribution (asset allocation) | Pie / Treemap | WalletTokenPnL grouped |
| Buy vs Sell breakdown | Stacked bar / Pie | DEXTradeByTokens grouped by Side.Type |
| Comparison (cross-chain, cross-protocol) | Grouped bar | DEXTrades grouped by Dex |
| Real-time feed (trade stream) | Table with scrolling | DEXTrades latest |
| Single metric (total count, total volume) | Metric card | Aggregation-only queries |
