# Research Log

My working notes for the project, newest entries at the bottom.

## 29 Sep 2026 - Data sources

I went through the free and freemium data sources for each of the 14
questions from my supervisor and checked what each one actually gives you:

- **Dune / Flipside** - free SQL on raw Ethereum, Bitcoin and Solana data,
  so I can write my own queries on transfers, DEX flows and stablecoins
  (Q1, Q6, Q7, Q8).
- **The Graph** - free subgraph queries for DEX liquidity pool events (Q8).
- **Etherscan / Basescan** - free tier (about 100k calls a day) for raw
  transactions and address labels (Q1, Q4).
- **DefiLlama** - free API for TVL and money moving between chains and
  venues (Q7, Q8).
- **Glassnode / CryptoQuant** - basic exchange inflow/outflow on the free
  tier (Q2, Q6), but the detailed data is paid.
- **Coinglass** - free open interest, funding rates and liquidations, which
  is useful context (Q11).
- **Whale Alert** - flags large transfers for free, but there's no history
  on the free tier (Q3).
- **Arkham Intelligence** - entity clustering and wallet history, the best
  free option I found for following whales over time (Q3, Q4, Q5).
- **Mempool.space** - free Bitcoin mempool data, for the timing question
  (Q10).
- **Alchemy / Infura** - free Ethereum node access if I need to read the
  chain directly (Q1, Q9, Q10).
- **CCXT** - free Python library for price and volume data from exchanges
  (Q1, Q11).

**Stack I went with:** Dune + Arkham + DefiLlama + Coinglass + CCXT covers
most of what the 14 questions need without paying anything. The parts that
aren't really covered are real-time anomaly detection (Q9) and large-scale
entity clustering (Q5). Those are only properly available through paid tools
like Nansen and Chainalysis, so I'll try building my own versions on top of
the free data.

*Update 13 Oct: I've swapped Dune for Google BigQuery's public datasets as
the main on-chain source (see the 13 Oct entry). I also dropped DefiLlama
(BigQuery and The Graph cover the same flows) and Coinglass (open interest
and funding are derivatives data, not one of my 4 on-chain signals). Arkham
needs API approval, so the backup is the wallet addresses exchanges publish
themselves.*

## Questions that no tool answers directly

These need me to build the analysis myself rather than just pull data:
anomaly detection against thresholds that change over time (Q9), testing
whether on-chain data adds anything beyond market data, e.g. with Granger
causality (Q11), measuring lead-lag between on-chain events and price at
different windows (Q12), and combining signals into one score (Q13).

## Note on overfitting (Q14)

This is the biggest risk in the whole project. If I test lots of on-chain
variables against price, something will look like a signal just by chance.
So:

- I'll set my hypotheses first, using questions 2 to 8, instead of testing
  everything and seeing what sticks
- I'll only test a small set of signals that I pick in advance
- I'll use walk-forward validation instead of testing on the same data I fit on

## 29 Sep 2026 - Problem, aim, objectives and scope

**Problem:** traders who only use price charts and market data miss a whole
type of information that shows up before the price moves: on-chain
activity. When a lot of crypto moves from a wallet to an exchange, it can
mean a sale is coming, but a chart-only trader won't see that. By the time
the price drops and they notice, the move has already started, so they miss
out or lose more than they would have if they'd seen it earlier. Platforms
like Nansen, Glassnode and Chainalysis already sell this kind of visibility
to well-funded desks, from around $49 a month up to an estimated
$50,000-200,000 a year, so the information clearly has value. It just isn't
available to someone without that budget. My project looks at whether
on-chain signals tell you something about price and volatility that market
data doesn't already, and whether a free tool can give an individual trader
that early view.

**Aim:** build a web dashboard for traders that shows on-chain signals
(exchange inflows/outflows, whale transfers, stablecoin flows and DEX
liquidity) for Bitcoin and Ethereum, and use it to test whether they predict
price movement and volatility beyond what market data shows.

**Objectives** (7, covering all 14 questions; dates come from my timeline):

1. By 20 Nov 2026: a data pipeline pulling on-chain data (BigQuery) and
   market data (CCXT) for BTC and ETH, with 3 months of history at each
   source's finest resolution (5-minute for prices), under 5% missing
   intervals, plus a live feed (Alchemy/Infura for ETH, Mempool.space for
   BTC). (Q1, Q10)
2. By 8 Jan 2027: collect the 4 signals (exchange flows, whale transfers
   cleaned up with labels and transfer types, stablecoin flows, DEX
   liquidity) using BigQuery, exchange wallet labels and The Graph. (Q2-Q8; Q7 through BigQuery,
   see 23 Oct)
3. By 8 Jan 2027: anomaly detection with rolling z-scores per wallet/token
   instead of one fixed threshold, flagging transfers over 2 SD. It passes
   if, in the test weeks, flagged hours are followed by bigger price moves
   than other hours (Mann-Whitney U, Holm over 2 tests: BTC and ETH). (Q9)
4. By 21 Jan 2027: compare a market-only model (the target's own last 24
   hours plus the last 24 hours of trading volume) against the same model
   plus each signal, on a walk-forward split. It passes if out-of-sample
   RMSE is lower for next-hour log returns and realised volatility (both
   chosen in advance), and the difference holds up on a Diebold-Mariano
   test after Holm correction (16 tests: 4 signals x 2 coins x 2 targets).
   Granger is only supporting evidence. (Q11)
5. By 21 Jan 2027: lead-lag, measured as the correlation between each
   signal and later returns and volatility at 30min, 1hr, 4hr and 24hr. The
   shortest window that is still significant after Holm (64 tests) and
   holds out-of-sample is the lead time. If none does, I don't claim a
   lead. (Q12)
6. By 10 Mar 2027: a composite score from all 4 signals (fixed equal
   weights), tested against the best single signal out-of-sample with
   Diebold-Mariano after Holm correction (4 tests: 2 coins x 2 targets).
   (Q13)
7. By 10 Mar 2027: walk-forward validation across objectives 3-6 so I'm
   not fooled by overfitting or by running lots of tests. (Q14)

**Scope:**
- In: everything in objectives 1-7, delivered as one web dashboard
- Out: coins other than BTC and ETH, live or real-money trading and any
  profit claims (I'm not modelling fees or slippage), claims beyond this
  3-month sample, a mobile app, and any signals beyond the 4 above

## 30 Sep 2026 - Literature and existing tools

I found 8 academic sources and checked each one against the publisher,
arXiv or Springer page myself (3 more added in October, numbers 9-11, so
the slides review 11):

1. Chi, Chu and Hao - return and volatility forecasting using on-chain
   flows (arXiv 2411.06327)
2. Grobys, Näsman and Sandretto - using on-chain data to predict Bitcoin
   cycles (Research in International Business and Finance, 2026)
3. Urumov and Chountas - clustering stock price volatility with
   intuitionistic fuzzy sets (2022), my supervisor's paper
4. Chalkiadakis et al. - on-chain analytics and statistical causality
   (Blockchain: Research and Applications, 2022)
5. Bailey et al. - backtest overfitting in financial markets (2016)
6. Kang et al. - de-anonymising the Bitcoin network with address
   clustering (BlockSys 2020)
7. Drakopoulou - stablecoin liquidity as a regime signal (Research Square
   preprint, 2026)
8. Zhu et al. - what drives liquidity on Uniswap (arXiv 2410.19107). I
   couldn't confirm a Springer version, so I cite the arXiv preprint.
9. Ante, Fiedler and Strehle - stablecoin issuance and abnormal crypto
   returns (Finance Research Letters, 2021)
10. Granger - the original causality test, used only as supporting
   evidence in Objective 4 (Econometrica, 1969)
11. Harvey, Liu and Zhu - multiple-testing bias in finance research
   (Review of Financial Studies, 2016)

**The gap:** each study looks at one type of signal on its own, with its
own method and time window. None of them puts exchange flows, whale
activity, stablecoin flows and DEX liquidity together in one tested system.
Chalkiadakis et al. test one sentiment signal against price, and Zhu et al.
look at DEX liquidity by itself. Bailey et al. and Harvey, Liu and Zhu
show that testing lots of
signals without proper validation leads to false discoveries, and most of
this literature doesn't really deal with that. So my project tests all
four together, with walk-forward validation, in one dashboard.

**Where the sources disagree:** Grobys et al. get strong backtested
results from on-chain indicators, but that's exactly the kind of result
Bailey et al. warn about, which is why I'm using walk-forward validation.
Chi et al. treat stablecoin flows as a simple linear predictor, while
Drakopoulou finds they work more like a signal for switching between market
regimes. Using my supervisor's fuzzy clustering method to define the regimes is an
idea for later, not part of the current plan.

**Existing tools:** Nansen (wallet labels, free tier or Pro from $49/month), Glassnode
(on-chain metrics, paid plans from $49/month), Arkham (entity labels; my label source only
if its API access is approved, otherwise exchange-published addresses) and Chainalysis (compliance forensics, ~$50k-200k/year, third-party
estimate). They all show on-chain data, but none of them tests whether it
actually predicts price, and none combines several signals into one tested
output. That's what my project adds. It's not new data, it's the testing
layer on top. Kang et al. show address clustering can de-anonymise a lot of
Bitcoin wallets, which is why I'm sticking to labels that are already
public and not doing my own clustering.

**Background numbers:** crypto market cap over $3.9 trillion (Aug 2025),
daily volume around $144 billion, Bitcoin dominance around 57%, and retail
transaction volume of $979 billion in Q1 2026, down 11% from about $1.1
trillion a year earlier (TRM Labs).

## 30 Sep 2026 - Tools

**Dashboard:** I picked Streamlit over Flask (which I used for LetBrick)
because it has charts and live updates built in, so I can spend more time
on the testing and less on building a front end.

What I'm using for each part, and what I decided against:

- **On-chain history:** BigQuery public datasets (crypto_bitcoin,
  crypto_ethereum) with SQL. The first 1 TiB of queries a month is free, so
  I filter by date and column instead of scanning whole tables. Dune's free
  plan is view-only now, so I'd only use it to sketch a query. Etherscan
  stays as a backup for lookups, since it isn't built for chain-wide
  queries. BigQuery runs a bit behind the live chain, so it's for history
  only.
- **Live feed:** Alchemy/Infura for Ethereum and Mempool.space for Bitcoin,
  both free. BigQuery is too far behind for this.
- **Whale/entity labels:** Arkham, but only if its API access is approved
  (see 15 Oct). If not, the wallet addresses exchanges publish themselves.
  Nansen has better labels, but Pro costs
  from $49 a month. Whale Alert has no history on the free tier, so it can't
  cover 3 months, but it could help with the live feed.
- **Cross-venue:** I dropped DefiLlama on 13 Oct. Stablecoin flows now come
  from BigQuery token transfers and DEX liquidity from The Graph.
- **DEX liquidity:** The Graph. I could query Uniswap contracts directly
  through Alchemy, but the existing subgraphs already have what I need.
- **Market data:** CCXT, one library for many exchanges. Going to each
  exchange's API directly is slightly faster but means a separate
  integration each time, and speed doesn't matter here.
- **Storage:** Python, pandas and SQLite. PostgreSQL would scale better,
  but it's just me using it.
- **Stats:** statsmodels and scipy. R would work too, but then my code
  would be split across two languages.
- **Version control:** Git and GitHub, which my supervisor asked for so he
  can follow progress through README.md and this log.

**How it fits together:** the data side (BigQuery, Arkham if approved, The Graph, CCXT,
Alchemy/Infura, Mempool.space) was chosen because it's all free and needs no
servers. My supervisor's tool list covered most of the 14 questions, and on
13 Oct he agreed BigQuery should replace Dune. The analysis side (pandas,
SQLite, statsmodels, scipy) is all Python, so the pipeline from Objective 1
feeds straight into the tests in Objectives 4-7. Streamlit can read the same
pandas data directly, which is the main reason I didn't go with Flask.

## 30 Sep 2026 - Methodology

**Approach: Agile, adapted for working on my own.**

The 7 objectives are incremental and can be tested independently and
this is exploratory research, so, it isn't known in advance which of the 4
signal types will actually be proven. That uncertainty is what
Agile is built for, rather than Waterfall, which assumes fixed requirements
upfront. It also fits my plan to try all 14 of my supervisor's questions and
drop whatever doesn't work, rather than cutting scope before I start.

**Strength:** if an objective underperforms during testing (e.g.
anomaly detection or composite scoring), the scope can be adapted without the whole
project collapsing, since each objective is individually tested.

**Limitation:** true Agile assumes a team with sprint reviews and
flexible deadlines, but as a student working on a fixed-deadline project
(PP, IPD, Final Report), this is Agile's iterative cycle adapted for a
solo academic project, not full Scrum, which needs defined roles and
team ceremonies.

**Where my requirements come from:**
- My supervisor: his 14 questions and tool list shaped the 7 objectives
- The literature: the gap (no tool combines several signals with proper
  testing) became the main requirement

**Methods and how I'll judge each objective** (decided now, before I see
any results, so I can't move the goalposts later):

| Objective | Method | Pass if |
|---|---|---|
| 3 (anomaly detection) | Rolling z-score per wallet/token, not one fixed threshold | It flags transfers more than 2 standard deviations from that wallet/token's recent average |
| 4 (beyond market data) | Baseline: OLS on the target's own last 24 hourly values plus the last 24 hourly trading volumes (log), refitted each training window. Signal model: the same plus one signal's last 24 hourly values. Walk-forward split; ADF test first; Granger as support only | Lower out-of-sample RMSE for next-hour log returns and realised volatility, and the gap is significant on Diebold-Mariano after Holm (16 tests). A signal that fails is reported, not hidden |
| 5 (lead-lag) | Correlation between each signal and later returns and volatility at 30min, 1hr, 4hr and 24hr, with Newey-West errors because the longer windows overlap | The shortest window that's significant after Holm (64 tests) and holds out-of-sample is the lead time |
| 6 (composite) | Equal weights on the 4 z-scored signals, fixed in advance; fitted weights only as a side check | Beats the best single signal (picked on training data only) out-of-sample on Diebold-Mariano, after Holm (4 tests: 2 coins x 2 targets) |
| 7 (walk-forward) | Expanding window: train on weeks 1-4, test week 5, then keep adding weeks and roll forward weekly to week 13 (9 test weeks). Scaling is fitted inside each training window and signals only use data known at that hour | RMSE pooled over the 9 test weeks (about 1,500 hours). Winning individual weeks is just a stability note. Nothing counts as confirmed unless it holds out-of-sample |

**What's original:** each method on its own is standard. What's new is
using them together on four signal types from different sources, for two
coins, with walk-forward validation, without letting the analysis quietly
overfit. Bailey et al. and my supervisor both called overfitting the biggest
risk in this area.

**Why Agile suits this:** each objective is its own pass/fail test. If
anomaly detection turns out noisy, or the composite doesn't beat the single
signals, I can simplify it, report it as a negative result or drop it, and
the rest still works. With Waterfall I'd have to commit to a composite
design before knowing if any single signal works at all.

## 30 Sep 2026 - Ethics, legal, social, EDI and sustainability

| Area | Issue | What I'll do |
|---|---|---|
| Legal | Every source (BigQuery, Arkham, The Graph, CCXT, Alchemy/Infura, Mempool.space) has its own terms | Stick to each API's terms: no scraping, no reselling, stay within rate limits |
| Privacy | Wallets are pseudonymous, but clustering can link them to real people | Only use labels that are already public (exchanges, funds), never try to de-anonymise anyone |
| Financial harm | People might read a prediction as a sure thing | Disclaimer, and show ranges and hit rates instead of buy/sell signals |
| EDI | In unstable economies stablecoins are often a way to protect savings, not speculation (for example where the local currency loses value quickly) | Keep the wording plain and analytical, no "get rich" language |
| Sustainability | No mining or new transactions, just querying data that already exists | Avoid repeat queries, filter BigQuery scans, store data locally |
| Crowd effects | If lots of people used tools like this, they could all react to the same event at once and make prices swing more | Out of scope to test, but the dashboard shows ranges and a historical hit rate rather than instant alerts, so people aren't all pushed to act at the same second |
| GDPR | Wallet addresses aren't tied to names by this project, so probably not personal data, but clustering makes that line blurry | Same as privacy: public labels only, no de-anonymising |

**SDGs:** SDG 10 (reduced inequalities) is the main one. The good on-chain
tools are priced for institutions (Nansen Pro from $49 a month, Chainalysis an
estimated $50k-200k a year). I'm only using free data, so an individual
trader gets the same kind of analysis without the budget.

## 30 Sep 2026 - Timeline

The plan covers the whole FYP, not just the proposal. I've set my own
targets about 2 weeks before each official deadline as a buffer:

| Phase | My target | Official deadline | Main task | Sub-tasks | Risk and fix |
|---|---|---|---|---|---|
| 1. Proposal | 22 Oct 2026 (formative draft) | 2 Nov 2026 | Finish and rehearse the PP | Finish slides, supervisor fixes, ethics form, rehearse Q&A | Away 2-23 Nov during the talk window: talk requested online, rehearsal done before leaving |
| 2. Data pipeline (Obj 1) | 20 Nov 2026 | Feeds SRS, 4 Dec 2026 | Build the pipeline | BigQuery queries (filtered by date and column), live feed, CCXT, SQLite, 3-month backfill | BigQuery quota and rate limits: filter queries, start with a shorter window, then expand |
| 3. Signals + anomalies (Obj 2, 3) | 8 Jan 2027 | First code demo, 22 Jan 2027 | Collect the 4 signals, build anomaly detection | Exchange flows first, then whales, stablecoins, DEX liquidity, rolling z-score | Labels harder than expected: simplest signal first, add the rest step by step |
| 4. Testing + first dashboard (Obj 4, 5) | 21 Jan 2027 | IPD, 4 Feb 2027 | Forecast tests and lead-lag, first dashboard | Baseline and signal models, Holm, first dashboard view | No significant result: that's still a valid finding, report it. The ethics declaration must be accepted by IPD, or the IPD mark is 0 |
| 5. Composite + validation (Obj 6, 7) | 10 Mar 2027 | - | Composite score, walk-forward | Fixed equal weights (fitted weights only as a side check), train/test splits | Slow backtests: start simple |
| 6. Dashboard + report | 8 Apr 2027 | Report/software/video, 22 Apr 2027 | Finish the dashboard, write up, record demo | Polish UI, write results, edit video | Writing crunch: draft each section as each objective finishes |
| 7. Viva prep | 20 Apr 2027 | Viva, 29 Apr-14 May 2027 | Rehearse | Mock Q&A on all 7 objectives | - |

The 2-week buffer on every phase is my main way of managing risk.

## 30 Sep 2026 - References (Harvard)

The full, up-to-date list is in the proposal slides. These are the 8 I
started with:

Bailey, D.H., Borwein, J.M., Salehipour, A., López de Prado, M. and Zhu, Q.
(2016) *Backtest overfitting in financial markets*. [Working paper].
Available at: https://www.davidhbailey.com/dhbpapers/overfit-tools-at.pdf
(Accessed: 30 September 2026).

Chalkiadakis, I., Zaremba, A., Peters, G.W. and Chantler, M.J. (2022)
'On-chain analytics for sentiment-driven statistical causality in
cryptocurrencies', Blockchain: Research and Applications, 3(2), Article
100063.

Chi, Y., Chu, Q. and Hao, W. (2024) Return and volatility forecasting
using on-chain flows in cryptocurrency markets. arXiv:2411.06327
[Preprint]. Available at: https://arxiv.org/abs/2411.06327
(Accessed: 30 September 2026).

Drakopoulou, V. (2026) Stablecoin liquidity as a crypto-native regime
signal: state-dependent density forecasts for BTC, ETH, and SOL.
[Preprint]. Research Square. Available at:
https://www.researchsquare.com/article/rs-9715935/v1
(Accessed: 30 September 2026).

Grobys, K., Näsman, S. and Sandretto, D. (2026) 'Using on-chain data to
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

Zhu, B., Liu, D., Wan, X., Liao, G., Moallemi, C. and Bachu, B. (2024)
'What drives liquidity on decentralized exchanges? Evidence from the
Uniswap protocol'. arXiv:2410.19107 [Preprint]. Available at:
https://arxiv.org/abs/2410.19107 (Accessed: 30 September 2026).

Checking notes: Bailey et al. is a working paper (dated Feb 2016, no
journal). Kang et al. page numbers and editors come from the Springer
chapter page (DOI 10.1007/978-981-15-9213-3_38). Chalkiadakis et al. uses
an article number because that journal doesn't use page ranges. Zhu et al.
is cited as the arXiv preprint because I couldn't find the Springer
version. I also fixed the accents in López de Prado and Näsman.

## 13 Oct 2026 - Supervisor feedback: data and tests

What we agreed, now in the slides as well:

- **Data:** BigQuery public datasets replace Dune as the main on-chain
  source, so everything stays free. Etherscan is a backup for lookups.
  BigQuery is behind the live chain, so the live feed uses Alchemy/Infura
  (ETH) and Mempool.space (BTC). The Graph for DEX liquidity, CCXT for
  prices.
- **History:** 3 months for BTC and ETH, which fits in the free 1 TiB a
  month as long as I filter queries by date and column.
- **Objective 4:** lagged-returns model vs the same model plus each signal,
  on a walk-forward split, compared by out-of-sample RMSE. The difference
  is tested with Diebold-Mariano, so "better forecast" is an actual result,
  not just a smaller number. Granger is only support, because a signal can
  pass Granger and still lose on the forecast.
- **Volatility:** added as a second target, decided in advance, so it's
  part of the Holm correction too.
- **Holm:** at 0.05 for each group. Objective 4 has 16 tests (4 signals x 2
  coins x 2 targets) and Objective 5 has 64. (Objective 6 adds 4 more, see
  22 Oct.) With only 3 months of data, a
  null result is likely and that's fine.
- **Objective 5:** fixed the test. It's the correlation between each
  signal and later returns/volatility, not return autocorrelation, which
  doesn't even use the on-chain data.
- **Lead-lag windows:** 30min, 1hr, 4hr, 24hr. I dropped 5 minutes because
  it's shorter than a Bitcoin block, so it's not a fair place to claim a
  lead.
- **Walk-forward:** first test in week 5, rolled weekly, so 9 test weeks
  out of 3 months.

## 13 Oct 2026 - Supervisor feedback: walk-forward

- **Expanding window:** fold 1 trains on weeks 1-4 and tests week 5, and
  each fold after that keeps all the earlier weeks, so the last one trains
  on about 12 weeks. Four weeks is short for rare events like big whale
  transfers, and a sliding window would throw away data I can't spare.
- **First test stays at week 5:** starting later would just mean fewer
  test weeks.
- **One pooled test:** the 9 test weeks (168 hours each, about 1,500 hours)
  are scored as one RMSE. Winning, say, 5 of 9 weeks is a stability note,
  not 9 separate chances to pass.
- **No look-ahead:** any scaling is fitted on the training data only, and
  each signal only uses data that was known at that hour.
- **Why this matters:** the model for each test week is fixed before that
  week starts, so it can't use anything it wouldn't have known at the time.

## 13 Oct 2026 - Data check and full test design

**BigQuery check (free sandbox, 13 Oct, around 22:34 UTC):**
- Newest Bitcoin block in `crypto_bitcoin.blocks`: 22:06 UTC. Newest
  Ethereum block in `crypto_ethereum.blocks`: 22:33 UTC. So both tables
  are up to date, about 30 minutes and 1 minute behind.
- Pulling 3 months of `crypto_ethereum.token_transfers` (13 Jul to 13 Oct,
  five columns, filtered by date) would scan 46.93 GB, under 5% of the free
  1 TiB a month. I'll pull it once and keep it locally.

**Where each signal comes from, and the backup plan:**

| Data | Source | Resolution | If it fails |
|---|---|---|---|
| Exchange flows | BigQuery BTC and ETH tables, plus exchange labels | Every block | Wallet addresses exchanges publish themselves |
| Whale transfers | BigQuery transfers, plus labels | Every block | Size threshold only, no labels |
| Stablecoin flows | BigQuery Ethereum token transfers (USDT, USDC) | Every block | Etherscan lookups |
| DEX liquidity | The Graph (Uniswap subgraph) | Hourly | Read pool events through Alchemy |
| Prices (targets) | CCXT (Binance) | 5-minute candles | Another exchange through CCXT |
| Live feed | Alchemy/Infura (ETH), Mempool.space (BTC) | Seconds | BigQuery, with its lag |

Go/no-go: I'll test every source in week 1 of the pipeline phase. If one
still doesn't work, I drop that signal, say so in the report, and carry on
with the rest.

**Test design, fixed before I run anything:**
- Targets: next-hour log return, and next-hour realised volatility (square
  root of the sum of squared 5-minute returns in that hour).
- Baseline (Obj 4): OLS on the target's own last 24 hourly values,
  refitted in each training window. (Trading volume added on 15 Oct, see
  below.)
- Signal model (Obj 4): the same baseline plus one signal's last 24 hourly
  values.
- Decision (Obj 4): lower pooled out-of-sample RMSE, confirmed by
  Diebold-Mariano with Holm over 16 tests.
- Lead-lag (Obj 5): Newey-West errors because the 4hr and 24hr windows
  overlap, Holm over 64 tests.
- Timing: a transfer counts in the hour its block is mined and is only used
  from the next hour. A daily signal uses the last full day.
- Composite (Obj 6): equal weights on z-scored signals. Fitted weights are
  only a side check. Compared with the best single signal by
  Diebold-Mariano, Holm over 4 tests (2 coins x 2 targets).
- What a pass means: the signal predicts something for BTC and ETH in this
  3-month sample. It doesn't mean profit, since I'm not modelling fees or
  slippage.
- Asked my supervisor: should trading volume go into the baseline?

## 15 Oct 2026 - Fixes after review

- **Wallet labels:** Arkham's API needs approval, and its terms only allow
  internal use and don't allow scraping or republishing their data. So
  Arkham is now something I check in week 1, not something I'm relying on.
  Backup: the wallet addresses exchanges publish themselves (Binance
  published theirs in Nov 2022). Those only cover main wallets, so the
  exchange flows would be partial.
- **Composite (Obj 6):** I pick the best single signal using training data
  only, then compare it with the composite on the same test weeks. That
  way the test weeks can't influence which signal counts as "best".
- **Hit rate:** for returns, it's the share of test hours where the
  predicted direction (up or down) was right. Volatility doesn't have a
  direction, so for that I show the forecast error compared with the
  baseline.
- **Background stat:** reworded to match TRM Labs: retail transaction
  volume of $979B in Q1 2026, down 11% from $1.1T a year earlier.

## 15 Oct 2026 - Trading volume added to the baseline

My supervisor said to add trading volume to the baseline. So the baseline is
now OLS on the target's own last 24 hourly values plus the last 24 hourly
trading volumes (log, from Binance through CCXT), refitted in each training
window. The signal model is the same plus one signal. This makes the test
harder to pass: a signal now has to add something beyond both past returns
and volume.

## 22 Oct 2026 - Supervisor corrections after the formative draft

My supervisor went through the slides and this repo and found some places
that didn't match. What I changed:

- **Exchange-flow chart:** it said Objective 5 tests the inflow idea. It's
  Objective 4 (does the signal improve the forecast). Objective 5 is only
  the lead time.
- **Literature trend line:** it said I fix "one horizon". I actually fix two
  targets (next-hour return and realised volatility), four lead-lag windows
  and one walk-forward design, all in advance.
- **Composite:** the methodology table said "weighted mix" and "equal weights
  first". It's fixed equal weights, with fitted weights only as a side
  check, which is what the test design slide already said.
- **Holm counts:** Objective 6 is also a Diebold-Mariano test, so it gets its
  own Holm group of 4 tests (2 coins x 2 targets). The groups are now 16
  (Obj 4), 64 (Obj 5) and 4 (Obj 6).
- **Sources:** the slides review 11 sources. Ante et al., Granger and
  Harvey et al. are now listed in the 30 Sep literature section too.
- **Arkham:** the 30 Sep tools notes now say Arkham is only the label source
  if its API is approved, with exchange-published addresses as the backup.
- **Tool prices:** older entries had Nansen at $99-499 and Glassnode at
  $29-999 a month. Updated to the prices on the slides (Nansen Pro and
  Glassnode paid plans from $49 a month, checked in October).
- **Question 7:** with DefiLlama dropped, cross-chain flows are out of
  scope. Exchange flows still show money moving in and out of exchanges, so
  Q7 is only covered in part. The README, the scope slide and the
  methodology slide now say this.

## 22 Oct 2026 - Time schedule and references tidy-up

- Updated the Gantt chart in Excel: bigger bold text, real dates, every
  module milestone marked (22 Oct, 2 Nov, 4 Dec, 22 Jan, 4 Feb, 22 Apr,
  29 Apr) and the composite as fixed equal weights.
- Marked my trip (2-23 Nov) as a planned risk: the talk is requested online.
- Added the new ethics rule: the declaration must be accepted by IPD.
- Made the reference list consistent (full stops after italic titles, one
  preprint format, Journal of Business & Economic Statistics).

## 23 Oct 2026 - Checked the proposal against the module lectures

Went through the W2, W3, W5 and requirements lectures and added what the
proposal was missing:

- **Risk scoring (W3):** each phase risk now has probability x impact
  (1-5 each): 15+ high, 6-14 medium, under 6 low. Highest are Arkham
  labels, report-writing and hard statistics questions (all 12, medium).
- **Skills (W3 PP checklist):** existing are Python, SQL, Git and Flask. To
  learn: BigQuery, The Graph queries, time-series testing (Diebold-Mariano,
  Newey-West) and Streamlit.
- **Named UK laws (W5):** API access only as approved (Computer Misuse Act
  1990), no republishing data (Copyright, Designs and Patents Act 1988), and
  under UK GDPR / Data Protection Act 2018 wallet addresses may count as
  "online identifiers", so only public entity labels are used.
- **Storage:** data backed up to University OneDrive, as the ethics lecture
  recommends.
- **To check:** the W2 supervisor-meeting table says the final report is due
  15 Apr and the viva is 19 Apr-4 May, but the module handbook says 22 Apr
  and 29 Apr-14 May. Checking which is right before the final PP.

## 23 Oct 2026 - Supervisor: Q7 through BigQuery

My supervisor said I can use BigQuery instead of DefiLlama for Q7, so it's
back in full scope:

- **Cross-exchange:** transfers between exchange wallets, from the same
  labelled exchange addresses as the exchange-flow signal.
- **Cross-chain:** Ethereum transfers into and out of bridge contracts, plus
  WBTC mints and burns for Bitcoin moving onto Ethereum. All of this is in the
  BigQuery Ethereum tables, so no new source is needed.
- These flows are collected under Objective 2 and shown on the dashboard.
  They aren't a fifth tested signal, so the Holm counts stay 16, 64 and 4.
- Updated the README, the scope, data and methodology slides, and removed
  the "out of scope" notes from 22 Oct.

## 26 Oct 2026 - Closing three design gaps

- **Objective 3 test:** "flags transfers over 2 SD" was true by definition, so
  it didn't test anything. Now it passes only if flagged hours are followed by
  bigger price moves than other hours in the test weeks, checked with a
  Mann-Whitney U test (Mann and Whitney, 1947), which suits fat-tailed crypto
  returns. Holm over 2 tests (BTC and ETH). Holm groups are now 2, 16, 64
  and 4.
- **30-minute lead-lag window:** the hourly timing rule doesn't fit a
  30-minute window, so for lead-lag each window starts at the first 5-minute
  price bar after the block's timestamp. The hourly rule stays for the
  forecast tests in Objective 4.
- **Signals for Bitcoin:** stablecoin flows are one market-wide series used
  for both BTC and ETH, and DEX liquidity for BTC comes from WBTC pools on
  Uniswap.
- **Series count:** with one market-wide stablecoin series, Objective 2 has
  7 series (exchange flows, whale transfers and DEX liquidity for each coin,
  plus stablecoins), not 8. The Holm counts don't change, because the
  stablecoin series is still tested against both coins.
- **Other fixes from a full check against the mark scheme:** the problem slide
  now names the deeper problem (platforms sell on-chain alerts without
  published out-of-sample tests); the literature slide links sources
  (stablecoins have the most support across studies, so they're the signal
  most likely to pass; exchange labels use the same clustering that Kang et
  al. show can de-anonymise users); the tools slide compares Alchemy, and
  PostgreSQL and Firestore, more critically; Objective 7 now covers
  objectives 3-6; the Q7 flows are on the pipeline and dashboard slides; and
  the three UK Acts are in the reference list (28 sources).

## 26 Oct 2026 - Supervisor: rules for the Q7 flows

My supervisor confirmed that collecting the Q7 flows and showing them on the
dashboard doesn't add a test, so the signal families stay at 16, 64 and 4.
The separate group of 2 is the Objective 3 anomaly check. Two rules keep that
true, now on the test design and dashboard slides:

- The exchange labels and the bridge-contract list are fixed before I look
  at any prices.
- The flows stay out of the composite and are never shown next to a p-value.
  On the dashboard they describe where coins moved; they aren't a fifth
  forecast.

We're going through the ethics declaration on Teams on Friday 30 Oct at 11am.
