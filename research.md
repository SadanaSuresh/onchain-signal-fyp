# Research Log

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

## 2026-09-29 — Problem Statement, Aim, Objectives and Scope finalised

**Problem statement:** Traders who rely only on price charts and market
data miss a category of information that exists before the price actually
moves: on-chain activity. A large amount of crypto moving from a wallet to
an exchange is usually a sign that a sale is coming, but chart-only traders
don't see this happening on the blockchain. By the time the price actually
drops and they notice, the move has already started, so they miss out on
profit or take a bigger loss than if they had seen the signal early. This
isn't a hypothetical gap: institutional analytics platforms (Nansen,
Glassnode, Chainalysis) already sell this exact type of on-chain visibility
to well-funded desks, at prices from roughly $99/month to an estimated
$50,000-200,000/year (see Background Review), which confirms the
information itself is valuable and already in active use, just not
accessible to a trader without that budget. This project investigates
whether on-chain signals carry information about price and volatility that
isn't already reflected in market data, and whether a free-tier tool can
give an individual trader that same early visibility.

**Aim:** develop a web dashboard for traders that surfaces on-chain signals
(exchange inflows/outflows, whale wallet transfers, stablecoin flows, DEX
liquidity activity) across Bitcoin and Ethereum, to evaluate whether these
signals provide predictive value for price movement and volatility beyond
what is visible from market data alone.

**Objectives (7, SMART, covering all 14 original research questions;
target dates taken from the Time Schedule below):**

1. By 20 Nov 2026, build a data pipeline pulling raw on-chain data (Dune)
   and market data (CCXT) for BTC/ETH, at least 3 months of historical plus
   live feed. (Q1, Q10)
2. By 8 Jan 2027, collect and process the 4 signal types (exchange flows,
   whale transfers refined by entity/transfer-type labels, stablecoin
   flows, DEX liquidity) using Dune, Arkham and DefiLlama. (Q2-Q8)
3. By 8 Jan 2027, build anomaly detection using rolling z-scores per
   wallet/token, rather than a fixed threshold. (Q9)
4. By 21 Jan 2027, test whether each signal adds predictive information
   beyond market data alone, using Granger causality and regression
   (p < 0.05). (Q11)
5. By 21 Jan 2027, measure lead-lag relationships via return
   autocorrelation across 5min, 30min, 4hr and 24hr windows. (Q12)
6. By 10 Mar 2027, build a composite score combining all 4 signals and
   compare its accuracy against each individual signal. (Q13)
7. By 10 Mar 2027, apply walk-forward validation across objectives 4-6 to
   guard against overfitting and multiple-testing bias. (Q14)

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
evaluated system. Chalkiadakis et al. (2022) test statistical causality
between on-chain sentiment and price using a single-signal framework, and
Zhu et al. (2026) study what drives DEX liquidity on Uniswap in isolation
from other on-chain signal types; both reinforce the single-signal pattern
this project is built to move past. Bailey et al. (2016) show that testing
multiple signals without proper validation risks false discoveries, a risk
this literature does not consistently address. This project closes that
gap by testing all four signal types together under walk-forward
validation, presented through one dashboard.

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
provide. Kang et al. (2020) show that address clustering can
de-anonymize a meaningful share of Bitcoin wallets using heuristics similar
to the entity clustering Arkham exposes, which both confirms the technical
feasibility of the whale-tracking approach in Objective 2 and is exactly
why Objective 2 and the Ethics section below restrict this project to
labels Arkham already makes public, rather than attempting independent
clustering.

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

**Why this specific combination, not just each tool in isolation:** the
stack is deliberately structured in two layers rather than picked
tool-by-tool. The data-collection layer (Dune, Arkham, DefiLlama, The
Graph, CCXT) was chosen entirely on one criterion, free access to real
raw or indexed data with no infrastructure to self-host, because Georgy's
own tool-mapping confirmed this combination alone covers the large
majority of the 14 research questions. The analysis layer (pandas,
SQLite, statsmodels, scipy) was chosen on a different criterion,
minimising integration overhead, by staying inside one language (Python)
end to end so the pipeline built in Objective 1 feeds directly into the
statistical testing in Objectives 4-7 without a format-conversion step
between them. Streamlit sits on top of both layers rather than being
picked independently: it needs to read directly from the same
pandas/SQLite objects the analysis layer already produces, which is why
Flask (requiring a separate templating/serialization step to display the
same data) was rejected specifically for this project's shape, not on
general merit.

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

**Analytical methods and success metrics (per objective):** the dev
methodology (Agile) governs how the project is built; these are the actual
technical methods and the specific pass/fail metric, each one is judged
against, so "success" is defined in advance rather than decided after
seeing the results, which is itself a safeguard against overfitting
risk raised in Objective 7.

| Objective | Method | Metric for success |
|---|---|---|
| 3 (anomaly detection) | Rolling z-score per wallet/token, not a fixed global threshold | Flags transfers >2 standard deviations from that wallet/token's own recent rolling mean |
| 4 (beyond-market-data test) | Granger causality + regression, on-chain signal vs price/volatility | Statistically significant at p < 0.05; a signal that fails this is reported as a negative result, not discarded |
| 5 (lead-lag) | Return autocorrelation across 5min/30min/4hr/24hr windows | The window with the strongest, earliest significant autocorrelation is reported as the effective lead time |
| 6 (composite score) | Weighted combination of the 4 signals, one score per asset per window | Combined score's predictive accuracy compared directly against each individual signal from Objective 4, same p < 0.05 threshold |
| 7 (walk-forward validation) | Sequential train/test split (e.g. train months 1-2, test month 3, roll forward) | A relationship only counts as confirmed if it holds on the out-of-sample window, not just in-sample |

**Original contribution:** integrating four independently-studied on-chain
signal types into one statistically validated system, a combination not
attempted in any reviewed literature or existing platform, tested under
walk-forward validation to guard against the overfitting risk identified
as the field's primary failure mode. The technical difficulty is not any
single method in isolation (each of the five methods above is individually
standard), it is applying all of them consistently across four
independently-sourced, differently-shaped signal types for two assets
without letting the analysis quietly overfit, which is exactly the failure
mode Bailey et al. (2016) and Georgy both identified as the field's biggest
risk.

**Why Agile specifically fits this technical shape:** each objective in
the table above is a self-contained pass/fail test. If Objective 3
(anomaly detection) turns out noisy or Objective 6 (composite scoring)
doesn't beat the individual signals, that objective can be simplified,
reported as a negative finding, or dropped without stalling the others,
because they don't depend on each other succeeding. A Waterfall plan fixed
in advance would force committing to a composite-scoring design before
knowing whether any individual signal even works, which is the specific
risk Agile's incremental structure avoids here.

## 2026-09-30 — Ethics, Legal, Social, EDI and Sustainability

| Area | Issue specific to this project | Safeguard |
|---|---|---|
| Legal | Each data source (Dune, Arkham, DefiLlama, CCXT) has its own free-tier ToS | Strict compliance per API: no scraping beyond limits, no reselling data, no exceeding rate limits |
| Privacy | Blockchain wallets are pseudonymous, but entity clustering (Objective 2) can link them to real identities | Only use identity labels already public via Arkham; never attempt independent deanonymization |
| Social/financial harm | Dashboard's prediction flags could be misread as guaranteed trading advice | Mandatory disclaimer (academic purpose only); display confidence ranges, not binary buy/sell signals |
| EDI | Stablecoins are disproportionately used by people in unstable economies as a survival tool, not speculation (~90% of Venezuela's Binance P2P volume is USDT) | Keep framing analytical/academic, not "get rich" marketing language |
| Sustainability | No new blockchain computation/mining created, only querying existing data | Avoid redundant/repeated API queries to minimize load on free-tier infrastructure |
| Systemic risk (beyond individual harm) | If signal-based tools like this become widely adopted, coordinated reaction to the same on-chain events could amplify volatility rather than reduce information asymmetry | Acknowledged as a scaling limitation beyond this project's scope to test, but not left unaddressed: the dashboard will display confidence ranges and historical hit-rate rather than an instant real-time alert, deliberately introducing a short interpretation delay before a user can act, which reduces the chance of many users reacting to the identical signal in the same instant |
| Legal boundary (GDPR) | Wallet addresses are pseudonymous, not directly tied to verified identity by this project, so likely outside GDPR's personal data definition; however entity clustering makes this boundary not absolute | Will not attempt independent deanonymization beyond labels already public through Arkham |

**SDG engagement:** SDG 10 (Reduced Inequalities) is the primary, directly-tied
SDG. Existing institutional-grade on-chain analytics tools are priced for
institutions (Nansen ~$99-499/month, Chainalysis an estimated
$50,000-200,000/year). This project deliberately uses only free-tier data
sources to build equivalent analytical capability, directly addressing the
information gap between institutional and retail market participants
identified in the Background Review.

## 2026-09-30 — Time Schedule

Full plan spans the whole FYP (not just the PP), aligned to the real module
timeline, with personal target dates set roughly 2 weeks ahead of each
official deadline as a deliberate risk-management buffer:

| Phase | My target | Official deadline | Main task | Key sub-tasks | Risk + mitigation |
|---|---|---|---|---|---|
| 1. PP finalization | 20 Oct 2026 | 2 Nov 2026 | Complete and rehearse PP | Finish remaining slides, assemble deck, rehearse Q&A | 2-week buffer for rehearsal and unexpected issues |
| 2. Data pipeline (Obj 1) | 20 Nov 2026 | Feeds SRS, 4 Dec 2026 | Build data collection pipeline | Dune API setup, CCXT integration, SQLite schema, 3-month historical backfill | API/rate limit issues → start with shorter historical window, expand once confirmed working |
| 3. Signal collection + anomaly detection (Obj 2, 3) | 8 Jan 2027 | First code demo, 22 Jan 2027 | Collect 4 signal types, build anomaly detection | Exchange flows first, then whale/entity refinement, stablecoin flows, DEX liquidity, rolling z-score | Entity clustering more complex than expected → build simplest signal first, add refinements incrementally |
| 4. Statistical testing + dashboard MVP (Obj 4, 5) | 21 Jan 2027 | IPD, 4 Feb 2027 | Granger causality, lead-lag testing, dashboard MVP | Implement statsmodels tests, build first dashboard view | No significant result found → still a valid research finding, reframe rather than treat as failure |
| 5. Composite scoring + validation (Obj 6, 7) | 10 Mar 2027 | — | Build composite score, apply walk-forward validation | Equal-weighted scoring first, then optimize, rolling train/test splits | Backtesting computational time → start simple before optimizing |
| 6. Full dashboard + report | 8 Apr 2027 | Final Report/Software/Video, 22 Apr 2027 | Finalize dashboard, write report, record demo | Polish UI, write methodology/results, edit video | Report-writing time crunch → draft sections as each objective completes |
| 7. Viva prep | 20 Apr 2027 | Viva, 29 Apr-14 May 2027 | Rehearse full project defense | Mock Q&A on all 7 objectives and findings | — |

The consistent ~2-week buffer across every phase is itself the stated risk
management strategy for the Time Schedule slide.

## Next steps

- Format the reference list in Harvard style
- Assemble all content into actual slides
- Start building the data collection pipeline (Dune + CCXT)

## 2026-09-30 — References (Harvard style)

Bailey, D.H., Borwein, J.M., Salehipour, A., Lopez de Prado, M. and Zhu, Q.
(2016) *Backtest overfitting in financial markets*. [Working paper].
Available at: https://www.davidhbailey.com/dhbpapers/overfit-tools-at.pdf
(Accessed: 30 September 2026).

Chalkiadakis, I., Zaremba, A., Peters, G.W. and Chantler, M.J. (2022)
'On-chain analytics for sentiment-driven statistical causality in
cryptocurrencies', Blockchain: Research and Applications, 3(2).

Chi, Y., Chu, Q. and Hao, W. (2024) Return and volatility forecasting
using on-chain flows in cryptocurrency markets. arXiv:2411.06327
[Preprint]. Available at: https://arxiv.org/abs/2411.06327
(Accessed: 30 September 2026).

Drakopoulou, V. (2026) Stablecoin liquidity as a crypto-native regime
signal: state-dependent density forecasts for BTC, ETH, and SOL.
[Preprint]. Research Square. Available at:
https://www.researchsquare.com/article/rs-9715935/v1
(Accessed: 30 September 2026).

Grobys, K., Nasman, S. and Sandretto, D. (2026) 'Using on-chain data to
predict Bitcoin cycles', Research in International Business and Finance,
89.

Kang, C., Lee, C., Ko, K., Woo, J. and Hong, J.W.K. (2020)
'De-anonymization of the Bitcoin network using address clustering', in
Zheng, Z., Dai, H.N., Fu, X. and Chen, B. (eds.) Blockchain and
Trustworthy Systems: BlockSys 2020, Communications in Computer and
Information Science, vol. 1267. Singapore: Springer, pp. 489-501.

Urumov, G. and Chountas, P. (2022) 'Clustering stock price volatility
using intuitionistic fuzzy sets', Notes on Intuitionistic Fuzzy Sets,
28(3), pp. 343-352.

Zhu, B., Liu, D., Wan, X., Liao, G., Moallemi, C. and Bachu, B. (2026)
'What drives liquidity on decentralized exchanges? Evidence from the
Uniswap protocol', in Financial Cryptography and Data Security: FC 2025
International Workshops. Cham: Springer.

Verification update (30 Sept 2026): both flags now closed. Bailey et al.
confirmed as a working paper, not a journal article — the source document
(dated Feb 2016) has no journal or conference listed anywhere in it, so
the citation above is correctly formatted as [Working paper]. Kang et al.
confirmed via the Springer chapter page directly: pp. 489-501, in
Communications in Computer and Information Science vol. 1267, edited by
Zheng, Dai, Fu and Chen, published 12 November 2020, DOI
10.1007/978-981-15-9213-3_38. Reference list above updated accordingly.
All 8 references are now fully verified with no outstanding flags.

## Next steps

- Assemble all content into actual slides (14-slide structure)
- Start building the data collection pipeline (Dune + CCXT)
