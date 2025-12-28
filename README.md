# Intrinsic Value × Entropy × Sentiment Alpha Framework

## Overview

This project is an end-to-end quantitative alpha framework that combines intrinsic valuation, market-implied expectations, entropy-based timing, and news sentiment to produce a single interpretable alpha score for publicly traded equities.

The core idea is simple:

> **Markets do not reward growth — they reward growth relative to expectations, adjusted for timing and uncertainty.**

This system decomposes that idea into measurable components and recombines them into a unified signal.

This project builds on two prior projects I developed:
- **Intrinsic Value & Market-Implied Growth Framework**
- **Entropy-Based Market Uncertainty Model**

Both were refactored and integrated into a single pipeline for alpha generation.

---

## Core Alpha Construction

The final alpha score is constructed as:
```
Alpha = Core Expectation Gap × Valuation Adjustment × Entropy × Sentiment
```
Each term is intentionally modular and interpretable.

---

## Intrinsic Value & Expectation Gap

Financial statements are used to derive:
- ROIC (Return on Invested Capital)
- ROIIC (Return on Incremental Invested Capital)
- Reinvestment rates
- Fundamental growth (`g_fund`)

A DCF-style solver is used to compute market-implied growth (`g_implied`) from the current enterprise value and WACC.

The core signal is the expectation gap:
```
(g_fund − g_implied) / WACC
```

This captures whether the business can sustainably outperform what the market is already pricing in.

---

## Valuation Check

Intrinsic enterprise value is compared to market enterprise value.

The mispricing signal is scaled using a bounded function to prevent extreme outputs. This ensures valuation enhances the signal without dominating it.

---

## Entropy (Timing & Uncertainty)

An entropy metric is computed from price behavior.

- Higher entropy indicates greater dispersion and potential deviation from equilibrium pricing
- Entropy is non-directional
- Acts as a timing and opportunity filter

---

## Sentiment (Short-Term Context)

Recent news headlines are collected and scored.

- Sentiment is time-weighted to emphasize more recent information
- Adjusts the alpha score to reflect near-term narrative pressure

---

## Implementation
## Project Structure

This project is organized as a modular, end-to-end alpha generation pipeline. Each file is responsible for a specific component of the valuation, uncertainty, and signal aggregation process.

### Core Modules

#### `financial_data.py`
- Pulls raw financial statement data from external sources  
- Cleans and standardizes financial statements  
- Provides a unified data interface for downstream models  

#### `calculate_fundamentals.py`
- Computes core fundamental metrics:
  - ROIC (Return on Invested Capital)
  - ROIIC (Return on Incremental Invested Capital)
  - FCFF (Free Cash Flow to the Firm)
  - Reinvestment rates
  - Fundamental growth estimates  
- Translates accounting data into economically meaningful signals  

#### `implied_solver.py`
- Solves for **market-implied growth** using:
  - Enterprise Value
  - WACC  
- Reverse-engineers the growth expectations embedded in the stock price  

#### `intrinsic.py`
- Performs intrinsic valuation  
- Computes expectation gaps between:
  - Fundamental growth capacity
  - Market-implied growth  
- Produces the core valuation signal  

#### `entropy.py`
- Computes market entropy as a proxy for uncertainty and dispersion  
- Normalizes entropy scores across assets  
- Acts as a timing and opportunity filter  

#### `sentiment.py`
- Scrapes recent news related to the asset  
- Scores sentiment with time-decay weighting  
- Aggregates sentiment into a normalized signal  

#### `alpha.py`
- Combines all model components into a single alpha score  
- Integrates:
  - Expectation gap
  - Valuation adjustment
  - Entropy
  - Sentiment  

#### `main.py`
- Orchestrates the full pipeline  
- Executes data ingestion, valuation, entropy, sentiment, and alpha computation  
- Serves as the primary entry point for running the framework  

---
### Example Output:
# Alpha Framework Run — Apple Inc. (AAPL)
```
====================================================================================================
Entropy: 2.7805089175654394
====================================================================================================
====================================================================================================
                                                          2021-09-30         2022-09-30         2023-09-30         2024-09-30         2025-09-30
RAW INPUTS           EBIT                                               119,437,000,000    114,301,000,000    123,216,000,000    133,050,000,000
                     PretaxIncome                                       119,103,000,000    113,736,000,000    123,485,000,000    132,729,000,000
                     TaxProvision                                        19,300,000,000     16,741,000,000     29,749,000,000     20,719,000,000
                     InterestExpense                   2,645,000,000      2,931,000,000      3,933,000,000                                      
                     InvestedCapital                                    170,741,000,000    173,234,000,000    163,579,000,000    172,390,000,000
                     TotalDebt                                          132,480,000,000    111,088,000,000    106,629,000,000     98,657,000,000
                     CashAndCashEquivalents                              23,646,000,000     29,965,000,000     29,943,000,000     35,934,000,000
                     mktCap                        4,057,362,333,696  4,057,362,333,696  4,057,362,333,696  4,057,362,333,696  4,057,362,333,696
                     BETA                                     1.1070             1.1070             1.1070             1.1070             1.1070
                     DepreciationAndAmortization                         11,104,000,000     11,519,000,000     11,445,000,000     11,698,000,000
                     CapitalExpenditure                                 -10,708,000,000    -10,959,000,000     -9,447,000,000    -12,715,000,000
                     ChangeInWorkingCapital                               1,200,000,000     -6,577,000,000      3,651,000,000    -25,000,000,000
DERIVED FUNDAMENTALS taxRate                                                     0.1620             0.1472             0.2409             0.1561
                     NOPAT                                              100,082,877,098     97,476,836,666     93,531,805,288    112,280,891,893
                     ROIC                                                        0.5862             0.5627             0.5718             0.6513
                     dNOPAT                                                                 -2,606,040,432     -3,945,031,378     18,749,086,604
                     dIC                                                                     2,493,000,000     -9,655,000,000      8,811,000,000
                     ROIIC                                                                         -1.0453             0.4086             2.1279
                     FCFF                                               120,694,877,098    126,531,836,666    110,772,805,288    161,693,891,893
                     reinvestment_rate                                                              0.0249            -0.0990             0.0942
                     implied_growth (ROIIC*reinv)                                                  -0.0260            -0.0405             0.2005
COST OF CAPITAL      rf (scalar)                              0.0414             0.0414             0.0414             0.0414             0.0414
                     erp (scalar)                             0.0300             0.0300             0.0300             0.0300             0.0300
                     beta (from data[0])                      1.1070             1.1070             1.1070             1.1070             1.1070
                     CAPM (scalar)                            0.0746             0.0746             0.0746             0.0746             0.0746
                     cost_of_debt                             0.0534             0.0464             0.0464             0.0534             0.0534
                     WACC                                                        0.0735             0.0737             0.0737             0.0739
                     Equity (mktCap scalar)        4,057,362,333,696  4,057,362,333,696  4,057,362,333,696  4,057,362,333,696  4,057,362,333,696
                     Debt                                               132,480,000,000    111,088,000,000    106,629,000,000     98,657,000,000
                     V = Debt + Equity                                4,189,842,333,696  4,168,450,333,696  4,163,991,333,696  4,156,019,333,696
BALANCE SHEET CHECKS NetDebt = Debt - Cash                              108,834,000,000     81,123,000,000     76,686,000,000     62,723,000,000
====================================================================================================
                                                                           Value
Section                     Metric                                              
Fundamental Growth Inputs   Blended ROIIC                                 1.3896
                            Reinvestment Rate                              9.42%
                            Fundamental Growth (g_fund)                   13.09%
Market Expectations         Market-Implied Growth (g_implied)              6.96%
Enterprise Value Comparison Market EV                          4,120,085,333,696
                            Intrinsic EV                       5,336,579,954,349
                            EV / EV_mkt                                   1.2953
Mispricing Signal           EV Mispricing (M_ev)                          29.53%
====================================================================================================

====================================================================================================
SENTIMENT ANALYSIS REPORT: Apple Inc. (AAPL)
====================================================================================================

Analysis Parameters:
  Lookback Period: 90 days
  Time Decay Factor: 0.9
  Articles Analyzed: 28

Sentiment Scores:
  Average Raw Sentiment: 0.0511
  Average Time-Weighted Sentiment: 0.0372
  Overall Sentiment: NEUTRAL

Sentiment Distribution:
  Positive: 13 (46.4%)
  Neutral:  14 (50.0%)
  Negative: 1 (3.6%)

====================================================================================================
TOP 5 MOST POSITIVE ARTICLES
====================================================================================================

1. Score: 0.213 (Raw: 0.292)
   Date: Unknown (3 days ago)
   Title: Apple Faces 2026 iPhone Sales Slump, But Still Best Positioned Vs. Peers, Counterpoint Research Says
   URL: https://www.benzinga.com/markets/tech/25/12/49427767/apple-faces-2026-iphone-sales-slump-but-still-best-positioned-vs-peers-counterpoint-research-says

2. Score: 0.082 (Raw: 0.112)
   Date: Unknown (3 days ago)
   Title: Apple (AAPL) Q3 Earnings and Revenues Top Estimates
   URL: https://www.nasdaq.com/articles/apple-aapl-q3-earnings-and-revenues-top-estimates

3. Score: 0.077 (Raw: 0.106)
   Date: Unknown (3 days ago)
   Title: Apple Plans 8 New iPhones: Report Says Folding Model, 20th Anniversary Edition In The Works
   URL: https://www.benzinga.com/trading-ideas/long-ideas/25/12/49459212/apple-plans-8-new-iphones-report-says-folding-model-20th-anniversary-edition-in-the-works

4. Score: 0.072 (Raw: 0.098)
   Date: Unknown (3 days ago)
   Title: What Makes Apple (AAPL) a Strong Holding?
   URL: https://www.insidermonkey.com/blog/what-makes-apple-aapl-a-strong-holding-1640163/

5. Score: 0.071 (Raw: 0.071)
   Date: 2025-12-28 (0 days ago)
   Title: Apple: Quality Vs. Price
   URL: https://seekingalpha.com/article/4856022-apple-quality-vs-price

====================================================================================================
TOP 5 MOST NEGATIVE ARTICLES
====================================================================================================

1. Score: -0.039 (Raw: -0.053)
   Date: Unknown (3 days ago)
   Title: Apple Inc. (AAPL)’s “Got Game,” Says Jim Cramer
   URL: https://www.insidermonkey.com/blog/apple-inc-aapls-got-game-says-jim-cramer-1572050/

2. Score: -0.025 (Raw: -0.035)
   Date: Unknown (3 days ago)
   Title: AAPL 'Stock Would Soar' If It Buys This, Says Jim Cramer: 'Continued Buybacks Will Do Nothing'
   URL: https://www.benzinga.com/markets/equities/25/07/46439852/aapl-stock-would-soar-if-it-buys-this-says-jim-cramer-continued-buybacks-will-do-nothing

3. Score: 0.002 (Raw: 0.002)
   Date: Unknown (3 days ago)
   Title: Apple Stock Is Rising Wednesday: Here's Why
   URL: https://www.benzinga.com/trading-ideas/movers/25/09/47472072/apple-stock-is-rising-wednesday-heres-why

4. Score: 0.002 (Raw: 0.002)
   Date: Unknown (3 days ago)
   Title: Analysts Have Conflicting Sentiments on These Technology Companies: Cognizant (CTSH) and Apple (AAPL)
   URL: https://www.theglobeandmail.com/investing/markets/stocks/AAPL/pressreleases/36756379/analysts-have-conflicting-sentiments-on-these-technology-companies-cognizant-ctsh-and-apple-aapl/

5. Score: 0.006 (Raw: 0.009)
   Date: Unknown (3 days ago)
   Title: Apple (AAPL) share price
   URL: https://economictimes.indiatimes.com/markets/us-stocks/apple-inc/aapl

====================================================================================================

====================================================================================================
ALPHA REPORT — AAPL
====================================================================================================

FINAL ALPHA
----------------------------------------------------------------------------------------------------
Alpha Score: 0.5346
Verdict: STRONG LONG

STRUCTURAL EXPECTATION GAP
----------------------------------------------------------------------------------------------------
Fundamental Growth (g_fund):      13.09%
Market-Implied Growth (g_implied): 6.96%
WACC:                              7.39%
Core Alpha ((g_fund - g_implied)/WACC): 0.830

VALUATION CHECK
----------------------------------------------------------------------------------------------------
EV Mispricing (M_ev):              29.53%
EV Scaling Factor (tanh):           0.755

TIMING & OPPORTUNITY FILTERS
----------------------------------------------------------------------------------------------------
Entropy (scaled):                  0.794
Sentiment (scaled):                1.074

ALPHA DECOMPOSITION
----------------------------------------------------------------------------------------------------
Alpha = Core × EV × Entropy × Sentiment
      = 0.830 × 0.755 × 0.794 × 1.074
      = 0.5346

INTERPRETATION
----------------------------------------------------------------------------------------------------
The market is pricing growth expectations below what fundamentals can support.
Valuation provides upside, and timing indicators do not materially oppose entry.
====================================================================================================
```

---

## Tools & Assistance

This project was implemented primarily in Python using a dedicated virtual environment to ensure dependency isolation and reproducibility.

To accelerate development and iteration:
- **ChatGPT** was used for implementation guidance, debugging, and architectural refinement
- **Claude** was used for cross-checking logic, improving structure, and validating assumptions

All modeling decisions, financial logic, and signal construction were ultimately designed and implemented by me. AI tools served strictly as productivity accelerators rather than decision-makers.

---

## Limitations & Caveats

### Small-Cap and Thinly Covered Stocks
Companies with limited or inconsistent financial disclosures can produce unstable or misleading outputs.

### Highly Volatile or Irregular Reporters
Firms with large one-time items, erratic cash flows, or frequent accounting changes can distort ROIIC, reinvestment, and growth calculations.

### Model Sensitivity
Intrinsic valuation and implied growth are sensitive to:
- Terminal assumptions
- WACC estimates
- Financial data quality

### Alpha Is Not a Prediction
The output is a relative signal, not a price target or timing guarantee. It is best used as a ranking and screening tool, not a standalone trading system.

---

## Goal

The goal of this project is not to predict short-term price movements, but to:
- Quantify expectation gaps
- Align valuation with business fundamentals
- Incorporate uncertainty and narrative context
- Produce a transparent, explainable alpha signal

This framework is designed to evolve and serve as a foundation for further research into expectation-driven returns.


