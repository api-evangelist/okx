# OKX Exchange GraphQL Schema

## Overview

This document describes a conceptual GraphQL schema for the OKX cryptocurrency exchange and Web3 platform. OKX provides a comprehensive REST and WebSocket API covering spot and derivatives trading, account management, market data, funding operations, sub-accounts, financial products, and on-chain wallet services.

The schema below maps OKX's REST API surface — documented at https://www.okx.com/docs-v5/en/ — onto a GraphQL type system. It is intended as a design reference for teams building GraphQL gateways, federated graphs, or API abstraction layers on top of OKX.

## Source API

- Documentation: https://www.okx.com/docs-v5/en/
- GitHub: https://github.com/okx
- Base URL: https://openapi.okx.com

## Schema File

See `okx-schema.graphql` in this directory for the full type definitions.

## Domain Coverage

### Account & Balances

Types covering the OKX unified account model, trading account balances, funding account balances, and currency-level detail including available equity, unrealized PnL, and margin ratios.

- `Account`, `TradingAccount`, `FundingAccount`, `UnifiedAccount`
- `Balance`, `CurrencyBalance`, `MarginBalance`
- `Equity`, `RiskExposure`, `Portfolio`

### Positions & Margin

Types representing open positions across all instrument types, position summaries, margin mode configuration, leverage settings, and cross/isolated margin states.

- `Position`, `PositionSummary`
- `MarginMode`, `CrossMargin`, `IsolatedMargin`, `LeverageInfo`

### Orders

Full order lifecycle types covering all OKX order categories: standard spot and derivatives orders, algorithmic orders (TWAP, grid, conditional), copy trading, and order status/side/type enumerations.

- `Order`, `SpotOrder`, `MarginOrder`, `FuturesOrder`, `OptionOrder`
- `AlgoOrder`, `ConditionalOrder`, `GridOrder`, `StopOrder`, `TWAPOrder`
- `CopyTrade`
- Enums: `OrderStatus`, `OrderSide`, `OrderType`, `PosSide`, `TDMode`

### Instruments

Instrument descriptors for every asset class traded on OKX, plus option chain aggregation.

- `Instrument`, `SpotInstrument`, `FutureInstrument`, `PerpetualInstrument`
- `OptionInstrument`, `OptionChain`

### Market Data

Real-time and historical market data types including tickers, order books, candlesticks, individual trades, index values, funding rates, open interest, and liquidation events.

- `MarketTicker`, `OrderBook`, `OrderBookEntry`
- `Candle`, `Trade`
- `IndexData`, `FundingRate`, `OpenInterest`
- `LiquidationOrder`, `Mark`
- `TakerVolume`, `LongShortRatio`

### Asset Management

Currency and asset metadata types used across trading, funding, and on-chain wallet operations.

- `Asset`

### Authentication & Webhooks

Types supporting API key management, OAuth token flows, and webhook event delivery.

- `APIKey`, `OAuthToken`
- `Webhook`, `WebhookEvent`

## Key Queries

```graphql
# Account
account(uid: ID!): Account
tradingAccount: TradingAccount
fundingAccount: FundingAccount

# Market Data
ticker(instId: String!): MarketTicker
orderBook(instId: String!, depth: Int): OrderBook
candles(instId: String!, bar: String, after: String, before: String, limit: Int): [Candle!]!
trades(instId: String!, limit: Int): [Trade!]!
fundingRate(instId: String!): FundingRate
openInterest(instType: String!): [OpenInterest!]!

# Instruments
instruments(instType: String!): [Instrument!]!
instrument(instId: String!): Instrument
optionChain(underlying: String!, expDate: String): OptionChain

# Orders
order(ordId: ID!, instId: String!): Order
openOrders(instType: String): [Order!]!
orderHistory(instType: String, after: String, limit: Int): [Order!]!
algoOrders(algoOrdType: String!): [AlgoOrder!]!

# Positions
positions(instType: String): [Position!]!
positionSummary: PositionSummary
```

## Key Mutations

```graphql
# Orders
placeOrder(input: PlaceOrderInput!): Order
cancelOrder(ordId: ID!, instId: String!): Order
amendOrder(input: AmendOrderInput!): Order
placeAlgoOrder(input: PlaceAlgoOrderInput!): AlgoOrder
cancelAlgoOrder(algoOrdId: ID!, instId: String!): AlgoOrder

# Account
setLeverage(instId: String!, lever: String!, mgnMode: String!): LeverageInfo
setMarginMode(mgnMode: String!): Account
transferFunds(input: TransferInput!): Transfer
withdraw(input: WithdrawInput!): Withdrawal

# API Keys
createAPIKey(input: APIKeyInput!): APIKey
deleteAPIKey(apiKey: String!): Boolean
```

## Authentication

OKX uses HMAC-SHA256 signed API keys. Each private request requires:

- `OK-ACCESS-KEY`: The API key
- `OK-ACCESS-SIGN`: HMAC-SHA256 signature
- `OK-ACCESS-TIMESTAMP`: ISO 8601 UTC timestamp
- `OK-ACCESS-PASSPHRASE`: Passphrase set at key creation

In a GraphQL gateway these credentials are passed as HTTP headers.

## Rate Limiting

OKX enforces per-endpoint rate limits, typically 20–60 requests per 2 seconds for trading endpoints and higher for market data. A GraphQL layer should implement per-field rate limit accounting mapped to the underlying REST call.

See `rate-limits/okx-rate-limits.yml` for detailed limit specifications.

## WebSocket Subscriptions

OKX's WebSocket API supports push channels for real-time order book updates, ticker streams, order status notifications, and position updates. A GraphQL subscription layer can map these to:

```graphql
subscription {
  tickerUpdated(instId: String!): MarketTicker
  orderBookUpdated(instId: String!, depth: String): OrderBook
  orderUpdated(instId: String): Order
  positionUpdated: Position
  accountUpdated: TradingAccount
}
```
