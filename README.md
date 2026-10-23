# On-Chain Signal Analysis for Crypto Trading

Final year project (6COSC023W), University of Westminster. Supervisor: Dr Georgy Urumov.

## What this project is about

Traders usually rely on price charts and market data, but that misses what's
happening on the blockchain itself, like large amounts of crypto moving in or
out of exchanges. That kind of activity can show up before a price move
appears on the chart, which matters for a trader's P&L.

This project tests whether on-chain signals (things you can see directly on
the blockchain, separate from price and volume) carry information that isn't
already in market data, and whether it's useful enough to act on.

## Research questions

These came out of my scoping discussions with my supervisor (progress on each
is in `research.md`):

1. Collecting raw on-chain and market data
2. Exchange inflows and outflows
3. Tracking whale wallets over time
4. Telling different types of transfers apart
5. Entity clustering (grouping wallets that belong to the same owner)
6. Stablecoin flows
7. Money moving between exchanges and chains (through BigQuery: transfers
   between exchange wallets, and transfers into and out of bridge contracts)
8. DEX (decentralised exchange) liquidity pools
9. Spotting unusual activity compared with a wallet's or token's normal behaviour
10. Timing (when a transfer is sent vs when it's confirmed on-chain)
11. Whether on-chain signals add anything beyond market data
12. How far on-chain activity leads price (lead-lag)
13. What happens when the signals are combined
14. Avoiding overfitting and false positives when testing lots of signals

## Data sources (all free tier)

| Source | What I use it for |
|---|---|
| Google BigQuery public datasets | Main on-chain source: SQL over Bitcoin and Ethereum history (exchange flows, whale transfers, stablecoin flows, cross-chain bridge flows) |
| Etherscan | Backup for address and transaction lookups |
| Arkham Intelligence | Wallet labels, if API access is approved (backup: wallet addresses exchanges publish themselves) |
| The Graph | DEX liquidity pool data |
| CCXT | Price and volume data from exchanges |
| Alchemy / Infura | Live Ethereum feed (BigQuery runs behind the live chain) |
| Mempool.space | Live Bitcoin data and transfer timing |

**Rest of the stack:** Python, pandas and SQLite for storage, statsmodels and scipy
for the tests, Streamlit for the dashboard.

## Status

- **Project proposal:** slides in `docs/FYP_PP_w21162895.pdf`, updated after my
  supervisor's feedback on 22 Oct. Final submission 2 Nov 2026.
- **Data check:** BigQuery queries tested on 13 Oct (`src/checks/`).
- **Next:** the data pipeline (Objective 1), target 20 Nov 2026, then the
  requirements spec by 4 Dec.

Full plan, decisions and supervisor feedback are in `research.md`.

## Folders

```
data/         raw and processed data
notebooks/    exploration
src/          data collection, processing and testing code
docs/         proposal material, diagrams and notes
research.md   my research log
```
