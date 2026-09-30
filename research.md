# Research Log

This log is to help me track the research and scoping decisions for the project, and to also
tell the supervisor to follow the progress.

---

## 2026-09-29 — Data source scoping

Reviewed the free/freemium data landscape for each of the 14 research questions,
based on the supervisor's mapping. Summary of what was checked and confirmed:

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
- **Coinglass** — free, open interest/funding rates/liquidations, useful
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
a large majority of the data needed for the 14 questions, at no cost. The
remaining gap is real-time anomaly detection (Q9) and large-scale entity
clustering (Q5), which is only available at usable depth through paid tools
(Nansen, Chainalysis) and will instead be approached as a DIY build on top of
the free data.

## Questions flagged as requiring custom analysis (no off-the-shelf tool)

The following are not solved by any data source directly and require building
the analysis: anomaly detection against dynamic thresholds (Q9), testing whether
on-chain data adds information beyond market data via statistical methods such
as Granger causality (Q11), measuring lead-lag relationships between on-chain
events and prices at different time windows (Q12), and combining multiple
signals into a composite score (Q13).

## Methodology note: overfitting and multiple testing (Q14)

Flagged by the supervisor as the biggest risk in this kind of research, since
Testing many possible on-chain variables against price increases the chance of
finding a "signal" that is really just noise. The approach going forward:

- Form hypotheses first, using questions 2 to 8 as priors, rather than testing
  everything blindly
- Test only a small, pre-defined set of signals
- Use walk-forward validation rather than in-sample testing

## 2026-09-29 — Aim, Objectives and Scope finalised

**Aim:** develop a web dashboard for traders that surfaces on-chain signals
(exchange inflows/outflows, whale wallet transfers, stablecoin flows, DEX
liquidity activity) across Bitcoin and Ethereum, to evaluate whether these
signals provide predictive value for price movement and volatility beyond
what is visible from market data alone.

**Objectives (7, SMART, covering all 14 original research questions):**

1. Build a data pipeline pulling raw on-chain data (Dune) and market data
   (CCXT) for BTC/ETH, at least 3 months of historical plus live feed. (Q1, Q10)
2. Collect and process the 4 signal types (exchange flows, whale transfers
   refined by entity/transfer-type labels, stablecoin flows, DEX liquidity)
   using Dune, Arkham and DefiLlama. (Q2-Q8)
3. Build anomaly detection using rolling z-scores per wallet/token, rather
   than a fixed threshold. (Q9)
4. Test whether each signal adds predictive information beyond market data
   alone, using Granger causality and regression (p < 0.05). (Q11)
5. Measure lead-lag relationships via return autocorrelation across 5min,
   30min, 4hr and 24hr windows. (Q12)
6. Build a composite score combining all 4 signals and compare its accuracy
   against each individual signal. (Q13)
7. Apply walk-forward validation across objectives 4-6 to guard against
   overfitting and multiple-testing bias. (Q14)

**Scope:**
- Included: everything in objectives 1-7, delivered as a web dashboard
- Excluded: cryptocurrencies beyond Bitcoin and Ethereum, live/real-money
  trading, a mobile app (web only), and any on-chain signals beyond the 4
  chosen types

## 2026-09-30 — Background Review: literature and existing solutions

Found and verified 8 real academic sources (checked directly against the
publisher/arXiv/Springer pages, not taken on trust):

1. Chi, Y., Chu, Q. and Hao, W., 'Return and Volatility Forecasting Using
   On-Chain Flows in Cryptocurrency Markets', arXiv 2411.06327
2. Grobys, K., Näsman, S. and Sandretto, D., 'Using on-chain data to predict
   Bitcoin cycles', Research in International Business and Finance, vol. 89,
   2026
3. Urumov, G. and Chountas, P., 'Clustering stock price volatility using
   intuitionistic fuzzy sets', Notes on Intuitionistic Fuzzy Sets, 28(3),
   2022 (my supervisor's own published paper)
4. Chalkiadakis, I., Zaremba, A., Peters, G.W. and Chantler, M.J., 'On-chain
   analytics for sentiment-driven statistical causality in cryptocurrencies',
   Blockchain: Research and Applications, 3(2), 2022
5. Bailey, D.H., Borwein, J.M., Salehipour, A., López de Prado, M. and
   Zhu, Q., 'Backtest overfitting in financial markets', 2016
6. Kang, C., Lee, C., Ko, K., Woo, J. and Hong, J.W.K., 'De-Anonymization of
   the Bitcoin Network Using Address Clustering', BlockSys 2020, Springer
7. Drakopoulou, V., 'Stablecoin Liquidity as a Crypto-Native Regime Signal:
   State-Dependent Density Forecasts for BTC, ETH, and SOL', Research
   Square preprint, 2026
8. Zhu, B., Liu, D., Wan, X., Liao, G., Moallemi, C. and Bachu, B., 'What
   Drives Liquidity on Decentralized Exchanges? Evidence from the Uniswap
   Protocol', Financial Cryptography and Data Security FC 2025 Workshops,
   Springer, 2026

**Gap analysis:** existing research treats these signals in isolation, each
study tests one on-chain signal type against price using its own
methodology and time window, with no single study combining exchange
flows, whale activity, stablecoin flows and DEX liquidity into one
evaluated system. Bailey et al. (2016) show that testing multiple signals
without proper validation risks false discoveries, a risk this literature
does not consistently address. This project closes that gap by testing all
four signal types together under walk-forward validation, presented
through one dashboard.

**Cross-source synthesis:** these sources do not fully agree with each
other. Grobys et al. (2026) report strong backtested outperformance from
on-chain indicators, but Bailey et al. (2016) show that this kind of result
is exactly the pattern most vulnerable to false discovery from untested
strategy variation, directly motivating Objective 7's walk-forward
validation rather than treating it as a generic precaution. Similarly, Chi
et al. model stablecoin flows as linear predictors of returns, while
Drakopoulou finds stablecoin liquidity instead signals non-linear regime
shifts, an open disagreement addressed by applying Urumov and Chountas's
(2022) fuzzy clustering approach, originally developed for non-separable
volatility data, to define these regimes within Objective 6's composite
scoring.

**Existing solutions reviewed:** Nansen (wallet labelling, ~$99-499/month),
Glassnode (macro on-chain metrics, ~$29-999/month), Arkham Intelligence
(entity labelling, ~$55/month, the tool I'm using), Chainalysis (blockchain
forensics for compliance, ~$50k-200k/year, third-party estimate). All four
surface on-chain data but none statistically test whether it predicts
price, and none combine multiple signal types into one evaluated output.
This project's contribution isn't new data collection (same free-tier
sources), it's the testing and integration layer these platforms don't
provide.

**Background stats (industry context):** global crypto market cap over
$3.9 trillion (Aug 2025), daily trading volume approximately $144 billion,
Bitcoin dominance approximately 57%, global retail crypto trading activity
$979 billion in Q1 2026 (TRM Labs, down 11% year-on-year).

## 2026-09-30 — Tools & Technologies

Dashboard framework decided: Streamlit over Flask (used previously on a
personal project), because Streamlit has built-in charting and live-update
components suited to a multi-signal dashboard, saving development time for
the more heavily weighted Methodology and Q&A preparation.

Full tools table with justification and alternatives-considered trade-off
analysis for each phase:

**Data collection (on-chain):** Dune Analytics, free custom SQL access to
raw chain data. Alternative: Alchemy/Infura, more flexible raw blockchain
access but requires manually building query/node infrastructure Dune
already abstracts; kept as a fallback if a signal proves impossible to
query through Dune.

**Data collection (entity/whale):** Arkham Intelligence, free entity
clustering and wallet history. Alternative: Nansen, more mature labelling
but $99-499/month; Whale Alert, real-time flags but no historical data on
the free tier, ruling it out for the 3-month historical requirement, though
it could supplement Arkham for the live feed specifically.

**Data collection (cross-venue):** DefiLlama, fully free, no rate limits.
Alternative: Glassnode, more curated/analyst-ready metrics but limited free
tier ($29-999/month for full granularity); DefiLlama's zero cost wins at
student-project scale despite requiring more manual aggregation.

**Data collection (DEX liquidity):** The Graph, free indexed subgraph
queries. Alternative: direct Alchemy/Infura querying, more granular control
but requires manually writing smart contract query logic; unnecessary since
existing Uniswap subgraphs already expose the needed data.

**Data collection (market data):** CCXT, free library unifying market data
across exchanges. Alternative: direct exchange APIs, marginally lower
latency but require separate integration per exchange; not worth it since
this project isn't latency-sensitive.

**Data storage/processing:** Python (pandas) + SQLite, free, no server
setup. Alternative: PostgreSQL, better concurrent-write scalability, but
unused capability for a single-user academic project.

**Statistical analysis:** Python (statsmodels, scipy). Alternative: R,
comparable statistical depth but would fragment the codebase between
collection (Python) and analysis (R) for no analytical benefit.

**Dashboard/visualization:** Streamlit, built-in charting/live-update.
Alternative: Flask, more UI control but requires manually building
charting functionality Streamlit provides natively.

**Version control/documentation:** Git + GitHub, specifically requested by
the supervisor for tracking progress via README.md and research.md.

## 2026-09-30 — Methodology

**Development methodology: Agile (iterative, solo-adapted).**

The 7 objectives are naturally incremental and independently testable, and
this is exploratory research, it isn't known in advance which of the 4
signal types will actually prove predictive. That uncertainty is what
Agile is built for, rather than Waterfall, which assumes fixed requirements
upfront. This also matches the earlier decision to attempt all 14 of the
supervisor's original questions and drop whichever parts don't work out
rather than negotiate scope down in advance, iterate then descope based on
what is learned.

**Strength:** if an objective underperforms in early testing (e.g.
anomaly detection or composite scoring), scope can adapt without the whole
project collapsing, since each objective is a separately testable
increment.

**Limitation:** true Agile assumes a team with sprint reviews and flexible
deadlines. As a solo student against fixed hard deadlines (PP, IPD, Final
Report), this is Agile's iterative spirit adapted for a solo academic
project, not full Scrum with defined roles and ceremonies.

**Requirement elicitation methods:**
- Supervisor consultation: Georgy's 14-question roadmap and tool-mapping
  emails functioned as structured stakeholder elicitation, directly
  shaping the 7 objectives
- Literature review: the gap analysis from the Background Review elicited
  the core functional requirement, no existing tool combines multiple
  signal types with statistical validation, which became this project's
  central requirement

**Original contribution:** integrating four independently-studied on-chain
signal types into one statistically validated system, a combination not
attempted in any reviewed literature or existing platform, tested under
walk-forward validation to guard against the overfitting risk identified
as the field's primary failure mode.

## Next steps

- Draft Ethics, legal, social, EDI and sustainability considerations (10%)
- Build the Time Schedule (Gantt chart) with dates for all 7 objectives
- Format the reference list in Harvard style
- Start building the data collection pipeline (Dune + CCXT) once the proposal
  scope is confirmed
