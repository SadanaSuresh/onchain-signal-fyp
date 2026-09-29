# Research Log

This file tracks the research and scoping decisions behind the project, so the
supervisor can follow progress without needing a separate update each time.

---

## 2026-09-29 — Data source scoping

Reviewed the free/freemium data landscape for each of the 14 research questions,
based on supervisor's mapping. Summary of what was checked and confirmed:

- **Dune Analytics / Flipside Crypto** — free SQL access to raw Ethereum, Bitcoin
  and Solana chain data. Covers custom queries on transfers, DEX flows and
  stablecoin movements (Q1, Q6, Q7, Q8).
- **The Graph** — free indexed subgraph queries, used for DEX liquidity pool
  events in close to real time (Q8).
- **Etherscan / Basescan** — free tier (approximately 100k calls/day), raw
  transaction data and address labels (Q1, Q4).
- **DefiLlama** — fully free API, TVL and cross-chain/cross-venue capital
  migration data (Q7, Q8).
- **Glassnode / CryptoQuant** — limited free tiers, cover exchange inflow/outflow
  at a basic level (Q2, Q6); full granularity is paywalled.
- **Coinglass** — free, open interest / funding rates / liquidations, useful
  derivatives context (Q11).
- **Whale Alert** — free tier flags large transfers with basic labels, but no
  historical data on the free tier (Q3).
- **Arkham Intelligence** — free tier with entity clustering and wallet history
  (Q3, Q4, Q5). Best free option for tracking whale behaviour over time.
- **Mempool.space** — free, Bitcoin mempool data, relevant to the pre-confirmation
  timing question (Q10).
- **Alchemy / Infura** — free tier Ethereum node access, for direct on-chain
  indexing if needed (Q1, Q9, Q10).
- **CCXT** — free Python library for market price/volume data across exchanges,
  used alongside the on-chain sources for comparison (Q1, Q11).

**Confirmed stack:** Dune + Arkham + DefiLlama + Coinglass + CCXT covers the
large majority of the data needed for the 14 questions without cost. The
remaining gap is real-time anomaly detection (Q9) and large-scale entity
clustering (Q5), which are only available at usable depth through paid tools
(Nansen, Chainalysis) and will instead be approached as a DIY build on top of
the free data.

## Questions flagged as requiring custom analysis (no off-the-shelf tool)

The following are not solved by any data source directly and require building
the analysis: anomaly detection against dynamic thresholds (Q9), testing whether
on-chain data adds information beyond market data via statistical methods such
as Granger causality (Q11), measuring lead-lag relationships between on-chain
events and price at different time windows (Q12), and combining multiple
signals into a composite score (Q13).

## Methodology note: overfitting and multiple testing (Q14)

Flagged by the supervisor as the biggest risk in this kind of research, since
testing many possible on-chain variables against price increases the chance of
finding a "signal" that is really just noise. The approach going forward:

- Form hypotheses first, using questions 2 to 8 as priors, rather than testing
  everything blindly
- Test only a small, pre-defined set of signals
- Use walk-forward validation rather than in-sample testing

## Next steps

- Finalise Project Proposal scope and SMART objectives
- Begin drafting the methodology and tools sections against the PP rubric
- Start building the data collection pipeline (Dune + CCXT) once the proposal
  scope is confirmed
