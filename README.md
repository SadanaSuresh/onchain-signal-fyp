## What this project is about

Traders usually rely on price charts and market data, but this often misses the on-chain
transfer data, such as large amounts of cryptocurrency moving in or out of exchanges.
This kind of activity can signal a price move before it shows up on the chart, which
matters for a trader's P&L.

This project investigates whether on-chain signals (things that can be observed
directly on the blockchain, separate from price and volume) carry information that
isn't already reflected in market data, and whether that information is useful enough
to act on.

## Research questions

Based on scoping discussions with the supervisor, the project investigates the
following (see `research.md` for progress on each):

1. Raw on-chain and market data collection
2. Exchange inflows and outflows
3. Whale wallet tracking over time
4. Distinguishing between types of transfers
5. Entity clustering (grouping wallets belonging to the same real-world actor)
6. Stablecoin flow monitoring
7. Cross-venue capital flow (money moving between exchanges/chains)
8. DEX (decentralised exchange) liquidity pool activity
9. Anomaly detection against normal wallet/token behaviour
10. Timing (when a transfer happens vs when it is confirmed on-chain)
11. Whether on-chain signals add information beyond market data alone
12. Lead-lag relationships between on-chain activity and price
13. How multiple signals interact when combined
14. Guarding against overfitting and multiple-testing bias when testing many signals

## Planned data sources (free tier)

| Source | Used for |
|---|---|
| Dune Analytics | Custom SQL queries on raw chain data (transfers, DEX flows, stablecoins) |
| Arkham Intelligence | Entity clustering, wallet labelling |
| DefiLlama | Cross-chain/cross-venue capital flow, stablecoin TVL |
| Coinglass | Derivatives context (open interest, funding, liquidations) |
| CCXT | Market price/volume data across exchanges |
| The Graph | DEX liquidity pool events (subgraphs) |
| Mempool.space | Bitcoin pre-confirmation timing |

## Repository structure

```
data/         raw and processed datasets
notebooks/    exploratory analysis
src/          data collection, processing and signal-testing code
docs/         project proposal materials, diagrams, written notes
research.md   ongoing research log
```
