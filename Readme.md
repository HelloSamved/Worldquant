# Quantitative Alpha Research Portfolio

> **Detailed Documentation of 10 Quantitative Alpha Strategies**  
> *Financial Hypotheses, Mathematical Construction, Data Fields, and Quantitative Methodologies*

---

## 📌 Metadata & Overview

| Attribute | Specification |
| :--- | :--- |
| **Research Domain** | Quantitative Finance & Statistical Arbitrage |
| **Specialization** | Equity Alpha Research & Signal Architecture |
| **Target Platform** | WorldQuant BRAIN-style Alpha Expression Language |
| **Strategies Documented** | 10 Distinct Alpha Expressions |
| **Primary Data Universes** | Price-Volume, Fundamentals, Options Surface, Alternative Sentiment |

---

## 📖 Table of Contents

- [Introduction](#-introduction)
- [Summary Matrix of 10 Alpha Strategies](#-summary-matrix-of-10-alpha-strategies)
- [Detailed Strategy Analysis](#-detailed-strategy-analysis)
  - [Alpha 1: Short-Term Price Reversal](#alpha-1-short-term-price-reversal)
  - [Alpha 2: Operating Income to Market Capitalization](#alpha-2-operating-income-to-market-capitalization)
  - [Alpha 3: Call-Put Implied Volatility Spread](#alpha-3-call-put-implied-volatility-spread)
  - [Alpha 4: News-Based Sentiment and Price Reversal](#alpha-4-news-based-sentiment-and-price-reversal)
  - [Alpha 5: Implied Volatility Skew and Volatility Regime](#alpha-5-implied-volatility-skew-and-volatility-regime)
  - [Alpha 6: Put-Call Open Interest and Options-Based Regression](#alpha-6-put-call-open-interest-and-options-based-regression)
  - [Alpha 7: GARP Valuation Ratio and Volatility](#alpha-7-garp-valuation-ratio-and-volatility)
  - [Alpha 8: Multi-Factor Volatility, Mean Reversion, Volume & Sentiment](#alpha-8-multi-factor-volatility-mean-reversion-volume-and-sentiment)
  - [Alpha 9: Realized Variance-Based Signal](#alpha-9-realized-variance-based-signal)
  - [Alpha 10: Analyst Cash Flow Flag and Volume Confirmation](#alpha-10-analyst-cash-flow-flag-and-volume-confirmation)
- [Common Operator Reference](#-common-operator-reference)
  - [Cross-Sectional Rank vs. Time-Series Rank](#cross-sectional-rank-vs-time-series-rank)
- [Overall Research Classification](#-overall-research-classification)
  - [Diversity of Information Sources](#diversity-of-information-sources)
- [Research & Validation Considerations](#-research-and-validation-considerations)
- [Conclusion & Best Practices](#-conclusion)
- [Documentation Disclaimer](#-documentation-disclaimer)

---

## 🔬 Introduction

This repository provides rigorous mathematical documentation and financial intuition for ten quantitative alpha strategies formulated in the WorldQuant BRAIN expression language. 

The core objective of quantitative alpha modeling is to transform raw, noisy market observations—spanning price action, corporate balance sheets, derivatives pricing surfaces, analyst estimates, and alternative crowdsourced sentiment streams—into normalized, cross-sectional predictive signals of relative future equity returns.

```
+---------------------------------------------------------------------------------+
|                                 Raw Data Inputs                                 |
|   Price/Volume  *  Fundamentals  *  Options Greeks  *  Alternative/Sentiment    |
+---------------------------------------------------------------------------------+
                                         |
                                         v
+---------------------------------------------------------------------------------+
|                       Time-Series Operators & Smoothing                         |
|        ts_delta()  *  ts_delay()  *  ts_mean()  *  ts_std_dev()  *  vec_avg()   |
+---------------------------------------------------------------------------------+
                                         |
                                         v
+---------------------------------------------------------------------------------+
|                       Cross-Sectional Relative Normalization                    |
|             rank()  *  ts_rank()  *  trade_when() Filtering                     |
+---------------------------------------------------------------------------------+
                                         |
                                         v
+---------------------------------------------------------------------------------+
|                         Tradable Alpha Signal / Weights                         |
|                 Dollar-Neutral / Industry-Neutral Target Weights                |
+---------------------------------------------------------------------------------+
```

---

## 📊 Summary Matrix of 10 Alpha Strategies

| ID | Strategy Name | Primary Category | Secondary Category | Lookback Horizon | Key Inputs |
| :-: | :--- | :--- | :--- | :--- | :--- |
| **01** | Short-Term Price Reversal | Price Mean Reversion | Short-Term Momentum | 3 Days | `close` |
| **02** | Operating Income to Market Cap | Fundamental Valuation | Historical Normalization | 250 Days (~1 Year) | `operating_income`, `cap` |
| **03** | Call-Put Implied Volatility Spread | Options Market | Implied Volatility Skew | 30-Day Tenor, 5D Backfill | `implied_volatility_call_30`, `implied_volatility_put_30` |
| **04** | News Sentiment & Price Reversal | Alternative Data | Price Mean Reversion | 60D News / 2D Return | `nws12_afterhsz_sl`, `close` |
| **05** | IV Skew & Volatility Regime | Options Market | Volatility Regime | 10-Day Tenor, 20D Moving Avg | `implied_volatility_call_10`, `implied_volatility_put_10` |
| **06** | Put-Call OI & Options Regression | Options Market | Regression-Based Signal | 20D OI, 10D Rolling Window | `pcr_oi_20`, `call_breakeven_20`, `close` |
| **07** | GARP Valuation Ratio & Volatility | Fundamental Valuation | Valuation Stability | 30 Days | `mdl177_garpanalystmodel_qgp_vfpriceratio` |
| **08** | Multi-Factor Composite Alpha | Multi-Factor Strategy | Alternative Data & Microstructure | 2D / 20D / 60D / 126D | `close`, `volume`, `adv20`, `scl12_alltype_buzzvec`, `scl12_sentiment` |
| **09** | Realized Variance-Based Signal | Volatility | Low-Volatility Anomaly | 20 Days (~1 Month) | `returns` |
| **10** | Analyst Cash Flow Flag & Volume | Fundamental / Analyst | Volume Confirmation | 5 Days | `anl4_cff_flag`, `volume` |

---

## 📑 Detailed Strategy Analysis

---

### Alpha 1: Short-Term Price Reversal

#### Expression
```c
alpha = -(ts_delta(close, 3) / ts_delay(close, 3));
```

#### Objective
Captures short-term mean-reverting equity returns over a rolling 3-day holding horizon by taking contrarian positions against aggressive short-term directional dislocations.

#### Mathematical Construction
1. **Three-Day Absolute Price Delta:**
   $$\Delta P_t = P_t - P_{t-3}$$
   where $P_t$ represents the current closing price and $P_{t-3}$ represents the closing price three trading sessions prior.

2. **Normalized Percentage Return:**
   $$R_{3,t} = \frac{\Delta P_t}{P_{t-3}} = \frac{P_t - P_{t-3}}{P_{t-3}}$$
   Normalizing by $P_{t-3}$ enables scale-invariant cross-sectional comparison across low-priced and high-priced equities.

3. **Signal Direction Reversal:**
   $$\alpha_t = -R_{3,t}$$
   Multiplying by $-1$ maps positive historical returns to short positions and negative historical returns to long positions.

#### Data Fields & Operators
| Item | Type | Description |
| :--- | :--- | :--- |
| `close` | Field | Daily unadjusted / split-adjusted closing stock price |
| `ts_delta(x, 3)` | Operator | Time-series difference between current observation and observation 3 days ago |
| `ts_delay(x, 3)` | Operator | Lag operator accessing historical observation at $t - 3$ |

#### Market Hypothesis
Equities undergoing sharp short-term moves often overshoot their intrinsic equilibrium due to temporary liquidity vacuums, institutional block liquidation, or retail overreaction. Once buying/selling pressure subsides, prices tend to mean-revert.

#### Expected Signal Behavior
- **Strong Positive 3-Day Return:** Highly Negative Alpha (Short signal)
- **Strong Negative 3-Day Return:** Highly Positive Alpha (Long signal)
- **Flat 3-Day Price Action:** Alpha near Zero (Neutral signal)

#### Key Limitations
- Vulnerable to persistent momentum trends driven by fundamental information surprises.
- High portfolio turnover leading to transaction cost and slippage erosion.

---

### Alpha 2: Operating Income to Market Capitalization

#### Expression
```c
alpha = ts_rank(operating_income / cap, 250);
```

#### Objective
Identifies equities generating high operating earnings relative to their equity market capitalization, normalized against each firm's historical 1-year valuation distribution.

#### Mathematical Construction
1. **Operating Income Yield:**
   $$\text{OI Ratio}_t = \frac{\text{Operating Income}_t}{\text{Market Capitalization}_t}$$
   Measures core operating profitability (EBIT level) per dollar of equity value.

2. **Time-Series Percentile Ranking:**
   $$\alpha_t = \operatorname{TSRank}(\text{OI Ratio}_t, 250)$$
   Ranks the current observation $\text{OI Ratio}_t$ against all values in the trailing 250 trading days (approximately 1 calendar trading year):
   $$\alpha_t \in [0, 1]$$
   where $1.0$ indicates that the stock's current operating yield is at its 1-year peak (historically undervalued relative to operating earnings).

#### Data Fields & Operators
| Item | Type | Description |
| :--- | :--- | :--- |
| `operating_income` | Field | Operating earnings generated from core corporate activities before interest and taxes |
| `cap` | Field | Current market capitalization ($\text{Shares Outstanding} \times \text{Price}$) |
| `ts_rank(x, 250)` | Operator | Rolling 250-day time-series percentile rank of input $x$ |

#### Market Hypothesis
Stock prices do not immediately reprice corporate operational improvements. When operating earnings rise while market valuation lags—driving the yield to a 250-day high—the equity is primed for positive upward repricing as markets digest financial results.

#### Key Limitations
- Susceptible to "value traps" if market capitalization drops drastically due to deteriorating future solvency.
- Accounting data is updated quarterly, generating step-function plateaus between earnings releases.

---

### Alpha 3: Call-Put Implied Volatility Spread

#### Expression
```c
alpha = rank(
    (
        ts_backfill(implied_volatility_call_30, 5)
        -
        ts_backfill(implied_volatility_put_30, 5)
    )
    /
    ts_backfill(implied_volatility_call_30, 5)
);
```

#### Objective
Extracts forward-looking risk premia and directional sentiment embedded in the options volatility surface by quantifying relative demand for upside call options versus downside put options.

#### Mathematical Construction
1. **5-Day Backfilled Implied Volatilities:**
   Let $IV_{\text{call},30,t}$ and $IV_{\text{put},30,t}$ denote the 5-day backfilled 30-day implied volatilities for call and put options (retrieved via `ts_backfill(implied_volatility_call_30, 5)` and `ts_backfill(implied_volatility_put_30, 5)`).

2. **Normalized Volatility Spread:**
   $$S_t = \frac{IV_{\text{call},30,t} - IV_{\text{put},30,t}}{IV_{\text{call},30,t}}$$
   A positive spread ($S_t > 0$) signifies higher call pricing relative to puts; negative spread indicates elevated put skew.

3. **Cross-Sectional Ranking:**
   $$\alpha_t = \operatorname{Rank}(S_t)$$
   Normalizes the relative spread uniformly across all universe constituents into the $[0, 1]$ domain.

#### Data Fields & Operators
| Item | Type | Description |
| :--- | :--- | :--- |
| `implied_volatility_call_30` | Field | 30-day constant maturity call implied volatility |
| `implied_volatility_put_30` | Field | 30-day constant maturity put implied volatility |
| `ts_backfill(x, 5)` | Operator | Replaces missing (`NaN`) values with the most recent observation within 5 periods |
| `rank(x)` | Operator | Cross-sectional percentile rank across all active assets at time $t$ |

#### Market Hypothesis
Options market participants are frequently informed institutional traders. Elevated call volatility relative to put volatility reflects disproportionate demand for upside leverage, anticipating positive corporate developments or breakout momentum.

#### Key Limitations
- Moneyness conventions and delta-neutral skew differences can create structural biases between calls and puts.
- Illiquid single-stock options contracts can produce stale backfilled readings.

---

### Alpha 4: News-Based Sentiment and Price Reversal

#### Expression
```c
alpha = if_else(
    rank(ts_sum(vec_avg(nws12_afterhsz_sl), 60)) > 0.5,
    1,
    rank(-ts_delta(close, 2))
);
```

#### Objective
Combines long-horizon media sentiment filtering with short-horizon price reversal. High cumulative positive sentiment grants an automatic maximum long score; otherwise, the model deploys a 2-day price reversal strategy.

#### Mathematical Construction
1. **Vector Aggregation & Rolling 60-Day News Accumulation:**
   $$N_t = \sum_{i=0}^{59} \text{News}_{t-i}$$
   where $\\text{News}_t$ aggregates the daily vector mean of sentiment (ec_avg(nws12_afterhsz_sl)) intraday and after-hours news sentiment vectors into a daily scalar score, summed over a trailing 60-day window.

2. **Cross-Sectional Sentiment Ranking:**
   $$\text{NewsRank}_t = \operatorname{Rank}(N_t)$$

3. **Conditional Dual-State Alpha:**
   $$\alpha_{i,t} = \begin{cases} 1, & \text{if } \text{NewsRank}_{i,t} > 0.5 \\ \operatorname{Rank}(-\Delta P_{2,i,t}), & \text{otherwise} \end{cases}$$
   where $\Delta P_{2,i,t} = P_{i,t} - P_{i,t-2}$ represents the 2-day price change (`ts_delta(close, 2)`).

#### Data Fields & Operators
| Item | Type | Description |
| :--- | :--- | :--- |
| `nws12_afterhsz_sl` | Field | Vector-valued after-hours news sentiment dataset |
| `close` | Field | Daily closing stock price |
| `vec_avg(x)` | Operator | Computes the arithmetic mean of all elements within a vector field |
| `ts_sum(x, 60)` | Operator | Rolling 60-day time-series sum |
| `if_else(c, a, b)` | Operator | Conditional branching operator: evaluates expression `a` if condition `c` holds, else `b` |

#### Market Hypothesis
Stocks with sustained high positive media coverage (top 50% percentile) benefit from enduring brand equity and investor attention. Stocks lacking strong positive media flow are subject to noise trader overreaction, where a 2-day contrarian price mean-reversion rule generates superior alpha.

#### Key Limitations
- The top 50% group receives an identical flat score of $1.0$, creating extensive ties that reduce cross-sectional differentiation.
- News sentiment classification models can misinterpret nuance, sarcasm, or earnings guidance language.

---

### Alpha 5: Implied Volatility Skew and Volatility Regime

#### Expression
```c
B = (
    ts_backfill(implied_volatility_call_10, 5)
    -
    ts_backfill(implied_volatility_put_10, 5)
)
/
ts_backfill(implied_volatility_call_10, 5);

C = (
    ts_backfill(implied_volatility_call_10, 5)
    +
    ts_backfill(implied_volatility_put_10, 5)
) / 2;

D = rank(C / ts_mean(C, 20));

alpha = rank(B) + D;
```

#### Objective
Synthesizes short-term (10-day tenor) options implied volatility skew with an implied volatility regime indicator to reward assets experiencing simultaneous upside option demand and volatility expansion.

#### Mathematical Construction
1. **Component B (Relative 10-Day Call-Put Skew):**
   $$B_t = \frac{IV_{\text{call},10,t} - IV_{\text{put},10,t}}{IV_{\text{call},10,t}}$$

2. **Component C (Average Implied Volatility Level):**
   $$C_t = \frac{IV_{\text{call},10,t} + IV_{\text{put},10,t}}{2}$$

3. **Component D (Normalized Volatility Expansion):**
   $$\text{MA}_{20}(C_t) = \frac{1}{20}\sum_{i=0}^{19} C_{t-i}, \quad V_t = \frac{C_t}{\text{MA}_{20}(C_t)}$$
   $$D_t = \operatorname{Rank}(V_t)$$

4. **Composite Multi-Factor Synthesis:**
   $$\alpha_t = \operatorname{Rank}(B_t) + D_t = \operatorname{Rank}(B_t) + \operatorname{Rank}(V_t)$$
   Produces a composite score bounded within $[0, 2]$.

#### Data Fields & Operators
| Item | Type | Description |
| :--- | :--- | :--- |
| `implied_volatility_call_10` | Field | 10-day option tenor call implied volatility |
| `implied_volatility_put_10` | Field | 10-day option tenor put implied volatility |
| `ts_mean(x, 20)` | Operator | Rolling 20-day historical moving average |

#### Market Hypothesis
Volatility regimes are two-dimensional: directional skew ($B$) identifies whether market participants favor call options over put options, while volatility acceleration ($D$) confirms that market attention and pricing dispersion are expanding.

#### Key Limitations
- Short 10-day expiration horizons exhibit high gamma and delta sensitivity around discrete corporate earnings announcements.
- Adding two ranked variables equally assumes orthogonal information without orthogonalization.

---

### Alpha 6: Put-Call Open Interest and Options-Based Regression

#### Expression
```c
alpha = trade_when(
    pcr_oi_20 < 1,
    rank(
        -1 * ts_regression(
            close,
            ts_backfill(call_breakeven_20, 30),
            10
        )
    ),
    -1
);
```

#### Objective
Employs an options open-interest positioning filter to trigger a 10-day rolling linear regression signal between equity prices and call breakeven strike levels.

#### Mathematical Construction
1. **Entry Gatekeeper Condition:**
   $$\text{Condition}_t: \quad \text{PCR}_{\text{OI},20} < 1.0 \implies \frac{\text{Put Open Interest}_{20}}{\text{Call Open Interest}_{20}} < 1.0$$
   Only activates trades when call open interest exceeds put open interest (bullish structural positioning).

2. **Rolling 10-Day Time-Series Regression:**
   $$\text{Close}_t = \beta_0 + \beta_1 X_t + \epsilon_t$$
   where $X_t$ is the 30-day backfilled call breakeven level (`ts_backfill(call_breakeven_20, 30)`).

3. **Negative Ranked Output & Trade Execution:**
   $$\text{Signal}_t = \operatorname{Rank}\Big(-1 \times \beta_{10,t}\Big)$$
   Trade execution is controlled via 	rade_when(pcr_oi_20 < 1, Signal, -1) with continuous holding until re-evaluated.
   The exit parameter `-1` indicates an infinite holding state (no programmatic forced exit).

#### Data Fields & Operators
| Item | Type | Description |
| :--- | :--- | :--- |
| `pcr_oi_20` | Field | 20-day put-call ratio based on total open interest |
| `call_breakeven_20` | Field | Strike price + call premium for 20-day options |
| `ts_regression(y, x, 10)` | Operator | Rolling 10-day linear regression slope/statistic between $y$ and $x$ |
| `trade_when(cond, expr, exit)`| Operator | State-dependent execution engine: triggers `expr` when `cond` is met; exits on `exit` |

#### Market Hypothesis
When call open interest dominates ($\text{PCR} < 1$), institutional traders hold bullish inventory. The regression measures price co-movement against call breakeven barriers, capturing mean-reverting deviations from option strike anchors.

#### Key Limitations
- Open interest can be driven by covered call writers (short volatility/capped upside) rather than outright bullish directional buyers.
- 10-day sample size in linear regression is susceptible to statistical noise and leverage points.

---

### Alpha 7: GARP Valuation Ratio and Volatility

#### Expression
```c
alpha =
    -1 * mdl177_garpanalystmodel_qgp_vfpriceratio
    * ts_std_dev(
        mdl177_garpanalystmodel_qgp_vfpriceratio,
        30
    );
```

#### Objective
Identifies companies with attractive valuation characteristics while penalizing securities exhibiting unstable, fluctuating valuation ratios.

#### Mathematical Construction
1. **Valuation Series Notation:**
   $$X_t = \text{ValuationRatio}_t = \text{GARP Price Ratio}_t$$

2. **Rolling 30-Day Historical Volatility of Valuation:**
   $$\sigma_{30,t} = \sqrt{\frac{1}{29} \sum_{i=0}^{29} \Big(X_{t-i} - \bar{X}_t\Big)^2}$$

3. **Risk-Penalized Alpha Construction:**
   $$\alpha_t = -1 \times \Big(X_t \cdot \sigma_{30,t}\Big)$$
   Assuming $X_t > 0$, an elevated valuation ratio combined with large estimation swings produces an aggressive short penalty.

#### Data Fields & Operators
| Item | Type | Description |
| :--- | :--- | :--- |
| `mdl177_garpanalystmodel_qgp_vfpriceratio` | Field | Proprietary Growth-at-a-Reasonable-Price (GARP) valuation ratio from analyst consensus |
| `ts_std_dev(x, 30)` | Operator | Rolling 30-day sample standard deviation |

#### Market Hypothesis
Under the GARP framework, high valuation multiples represent elevated downside risk unless offset by predictable stability. Severe volatility in analyst valuation models suggests model disagreement, earnings forecast uncertainty, or operational instability.

#### Expected Signal Behavior
| Valuation Level ($X_t$) | Valuation Volatility ($\sigma_{30,t}$) | Resulting Alpha ($\alpha_t$) | Positioning |
| :--- | :--- | :--- | :--- |
| **High** | **High** | Strongly Negative | Heavy Underweight / Short |
| **High** | **Low** | Moderately Negative | Mild Underweight |
| **Low** | **High** | Mildly Negative | Neutral |
| **Low** | **Low** | Least Negative / Near 0 | Relative Long Overweight |

#### Key Limitations
- Multiplication amplifies outliers in either variable.
- Requires precise confirmation of whether higher $X_t$ indicates overvaluation or undervaluation.

---

### Alpha 8: Multi-Factor Volatility, Mean Reversion, Volume and Sentiment

#### Expression
```c
return = (close / ts_delay(close, 1)) - 1;

volatility = ts_std_dev(return, 60);

when = (ts_rank(volatility, 126) > 0.65);

mr_component =
    -rank(
        (close - ts_delay(close, 2))
        / ts_delay(close, 2)
        / ts_std_dev(return, 20)
    );

vol_component = rank(volume / adv20);

buzz_avg = -vec_avg(scl12_alltype_buzzvec);

sentiment_component = rank(scl12_sentiment);

alpha = (
    0.5 * mr_component +
    0.25 * vol_component +
    0.125 * buzz_avg +
    0.125 * sentiment_component
);

trade_when(
    when,
    alpha,
    -1
);
```

#### Objective
A regime-switching multi-factor alpha that triggers strictly in high-volatility environments ($\ge 65\text{th}$ percentile over 6 months), blending volatility-adjusted mean reversion, volume confirmation, contrarian buzz filtering, and positive social sentiment.

#### Mathematical Construction
1. **Daily Returns & 60-Day Realized Volatility:**
   $$R_t = \frac{P_t}{P_{t-1}} - 1, \quad \sigma_{60,t} = \text{StdDev}(R_{t-59}, \dots, R_t)$$

2. **Regime Activation Gatekeeper:**
   $$\text{when}_t = \mathbb{I}\Big[\operatorname{TSRank}(\sigma_{60,t}, 126) > 0.65\Big]$$

3. **Sub-Component Formulations:**
   - **Volatility-Adjusted 2-Day Mean Reversion ($S_{\text{MR},t}$, 50% weight):**
     $$S_{\text{MR},t} = -\operatorname{Rank}\left(\frac{(P_t - P_{t-2})/P_{t-2}}{\sigma_{20,t}}\right)$$
     where $\sigma_{20,t} = \operatorname{StdDev}_{20}(R)$.
   - **Relative Volume Acceleration ($S_{\text{Vol},t}$, 25% weight):**
     $$S_{\text{Vol},t} = \operatorname{Rank}\left(\frac{\text{Volume}_t}{\text{ADV20}_t}\right)$$
   - **Contrarian Social Buzz Penalty ($S_{\text{Buzz},t}$, 12.5% weight):**
     $$S_{\text{Buzz},t} = -\overline{\text{Buzz}}_t$$
     where $\overline{\text{Buzz}}_t$ represents the vector mean of discussion activity (`vec_avg(scl12_alltype_buzzvec)`).
   - **Social Sentiment Ranking ($S_{\text{Sent},t}$, 12.5% weight):**
     $$S_{\text{Sent},t} = \operatorname{Rank}(\text{Sentiment}_t)$$
     where $\text{Sentiment}_t$ is the NLP sentiment score (`scl12_sentiment`).

4. **Composite Linear Factor Weighting:**
   $$\alpha_t = 0.50 \cdot S_{\text{MR},t} + 0.25 \cdot S_{\text{Vol},t} + 0.125 \cdot S_{\text{Buzz},t} + 0.125 \cdot S_{\text{Sent},t}$$

```
Factor Weight Allocation:
+-------------------------------------------------------------+
| [■■■■■■■■■■■■■■■■■■■■] Mean Reversion: 50.0%               |
| [■■■■■■■■■■] Relative Volume: 25.0%                         |
| [■■■■■] Social Buzz (Contrarian): 12.5%                     |
| [■■■■■] Social Sentiment: 12.5%                             |
+-------------------------------------------------------------+
```

#### Data Fields & Operators
| Item | Type | Description |
| :--- | :--- | :--- |
| `close`, `volume` | Fields | Daily closing price and share trading volume |
| `adv20` | Field | 20-day average daily trading volume |
| `scl12_alltype_buzzvec` | Field | Vector containing discussion buzz across social channels |
| `scl12_sentiment` | Field | Aggregated NLP polarity score for company discussions |

#### Market Hypothesis
Short-term mean reversion works best during turbulent market conditions when forced liquidations and emotional panic dominate trading. During these regimes, high volume confirms order-flow exhaustion, unranked excessive retail buzz marks speculative frenzy (penalized), and positive sentiment cushions downside risk.

#### Key Limitations
- `buzz_avg` is not cross-sectionally ranked, meaning its raw numerical magnitude may disproportionately dominate or be eclipsed by the ranked components.

---

### Alpha 9: Realized Variance-Based Signal

#### Expression
```c
alpha = -ts_sum(power(returns, 2), 20);
```

#### Objective
Systematically exploits the empirical **low-volatility anomaly** by calculating a 20-day proxy for realized variance and tilting the portfolio in favor of low-variance assets.

#### Mathematical Construction
1. **Squared Daily Returns:**
   $$R_t = \text{returns}_t = \frac{P_t}{P_{t-1}} - 1 \implies R_t^2 = \text{power}(R_t, 2)$$
   Squaring strips directional sign and heavily penalizes large daily jumps:
   $$(0.04)^2 = 0.0016 = 4 \times (0.02)^2$$

2. **20-Day Realized Variance Proxy:**
   $$\text{RV}_{20,t} = \sum_{i=0}^{19} R_{t-i}^2$$

3. **Inverted Alpha Signal:**
   $$\alpha_t = -\text{RV}_{20,t}$$
   Assets with small accumulated variance receive higher (less negative) alpha allocations.

#### Data Fields & Operators
| Item | Type | Description |
| :--- | :--- | :--- |
| `returns` | Field | Daily percentage equity return |
| `power(x, 2)` | Operator | Computes mathematical square $x^2$ |
| `ts_sum(x, 20)` | Operator | Rolling 20-day time-series sum |

#### Market Hypothesis
The **Low-Volatility Anomaly** demonstrates that low-volatility equities frequently achieve higher risk-adjusted returns (higher Sharpe ratios) than high-volatility equities. Behavioral drivers include investor lottery-ticket seeking behavior, retail preference for high-beta stocks, and institutional mandate leverage constraints.

#### Key Limitations
- Does not separate upside (desirable) volatility from downside (detrimental) variance.
- Tends to accumulate structural defensive factor bets (utilities, consumer staples) with significant interest-rate duration risk.

---

### Alpha 10: Analyst Cash Flow Flag and Volume Confirmation

#### Expression
```c
when = volume > ts_mean(volume, 5);

alpha = rank(ts_mean(anl4_cff_flag, 5));

trade_when(when, alpha, -1);
```

#### Objective
Synchronizes fundamental analyst revisions in operating cash flow with institutional market participation, executing trades only when current trading volume validates the fundamental signal.

#### Mathematical Construction
1. **Five-Day Moving Average Volume Benchmark:**
   $$\text{ADV5}_t = \frac{1}{5}\sum_{i=0}^4 \text{Volume}_{t-i}$$

2. **Execution Trigger Condition:**
   $$\text{when}_t = \text{Volume}_t > \text{ADV5}_t$$

3. **Smoothed Fundamental Signal & Ranking:**
   $$\overline{\text{CFF}}_t = \frac{1}{5}\sum_{i=0}^4 \text{CFF}_{t-i}$$
   $$\alpha_t = \operatorname{Rank}\left(\overline{\text{CFF}}_t\right)$$
   where $\text{CFF}_t$ represents the daily analyst cash flow flag (`anl4_cff_flag`), smoothed over 5 trading days.

4. **Conditional Trade Execution:**
   The strategy executes entries via `trade_when(when, alpha, -1)` when current trading volume exceeds its 5-day moving average, maintaining active exposure until re-evaluated.

#### Data Fields & Operators
| Item | Type | Description |
| :--- | :--- | :--- |
| `volume` | Field | Current session total trading volume |
| `anl4_cff_flag` | Field | Proprietary analyst indicator flagging cash-flow condition changes |
| `ts_mean(x, 5)` | Operator | 5-day rolling arithmetic mean |

#### Market Hypothesis
Fundamental analyst revisions often go unnoticed until institutional trading volume surges, signaling that "smart money" is actively accumulating or distributing shares based on the underlying cash flow revision.

#### Key Limitations
- High trading volume can stem from adverse events (earnings misses, lawsuits, management departures) rather than positive cash flow surprises.
- Binary indicator flags may yield discrete clumps of tied rankings.

---

## 🛠 Common Operator Reference

| Operator | Signature | Mathematical Definition | Typical Research Application |
| :--- | :--- | :--- | :--- |
| `ts_delta` | `ts_delta(x, d)` | $x_t - x_{t-d}$ | Short-term momentum, price change, trend shocks |
| `ts_delay` | `ts_delay(x, d)` | $x_{t-d}$ | Historical price retrieval, lagged ratio denominator |
| `ts_rank` | `ts_rank(x, d)` | $\frac{\text{Rank of } x_t \text{ among } [x_{t-d+1}, \dots, x_t]}{d}$ | Historical percentile normalization (internal valuation) |
| `rank` | `rank(x)` | $\frac{\text{Rank of } x_{i,t} \text{ among all assets in universe } N_t}{N_t}$ | Cross-sectional percentile ranking across universe |
| `ts_mean` | `ts_mean(x, d)` | $\frac{1}{d} \sum_{i=0}^{d-1} x_{t-i}$ | Rolling trendline smoothing, baseline averaging |
| `ts_sum` | `ts_sum(x, d)` | $\sum_{i=0}^{d-1} x_{t-i}$ | Cumulative accumulation (realized variance, sentiment sums) |
| `ts_std_dev` | `ts_std_dev(x, d)` | $\sqrt{\frac{1}{d-1}\sum_{i=0}^{d-1}(x_{t-i}-\bar{x})^2}$ | Historical volatility estimation, signal risk scaling |
| `ts_backfill` | `ts_backfill(x, d)` | Returns most recent non-NaN value within window $d$ | Handling sparse data (options surfaces, analyst flags) |
| `vec_avg` | `vec_avg(x)` | $\frac{1}{K}\sum_{k=1}^K x[k]$ | Compressing vector-valued alternative data into a scalar |
| `power` | `power(x, n)` | $x^n$ | Non-linear signal amplification, squared returns ($R^2$) |
| `ts_regression`| `ts_regression(y, x, d)` | Returns linear regression coefficient ($\beta$ or residual) | Rolling factor sensitivity and co-movement analysis |
| `if_else` | `if_else(cond, a, b)` | Returns $a$ if $\text{cond} = \text{true}$, else $b$ | Non-linear regime switching and conditional branching |
| `trade_when` | `trade_when(c, a, exit)` | Holds position $a$ while $c$ is met, exits on $exit$ | State-dependent execution engine with entry/exit logic |

---

### Cross-Sectional Rank vs. Time-Series Rank

Understanding the mathematical distinction between `rank()` and `ts_rank()` is fundamental to quantitative portfolio construction:

```
Cross-Sectional Rank: rank(x)                    Time-Series Rank: ts_rank(x, d)
Evaluates across ALL STOCKS at time t            Evaluates ONE STOCK across TIME [t-d, t]

       Stock A  Stock B  Stock C                       Stock A Historical Values
t=0  [  0.92  |  0.15  |  0.54  ]                 t=0   | 102.5 | <- Current (Rank = 1.0)
          |        |        |                    t=-1  |  98.2 |
          v        v        v                    t=-2  |  95.0 |
     rank(x) evaluates relative                  t=-3  |  91.4 |
     universe position today.                    ts_rank(x, 4) evaluates where today sits
                                                 relative to its own recent past.
```

- **`rank(x)` (Cross-Sectional):** Answers *"Is Stock A more attractive than Stock B today?"* Guarantees dollar-neutral, uniform percentile distribution across the investment universe at each timestamp.
- **`ts_rank(x, d)` (Time-Series):** Answers *"Is Stock A unusually cheap or volatile relative to its own recent history?"* Does not compare across companies; isolates internal cyclical extremes.

---

## 🧭 Overall Research Classification

### Categorical Classification

```
+-----------------------------------------------------------------------------------------+
|                               ALPHA RESEARCH SPECTRUM                                   |
+-----------------------------------------------------------------------------------------+
| [Technical / Microstructure]          [Derivatives / Options]         [Fundamental / NLP]|
| - Alpha 1: 3D Price Reversal          - Alpha 3: Call-Put IV Spread   - Alpha 2: OI Yield|
| - Alpha 9: 20D Realized Variance      - Alpha 5: IV Skew & Regime     - Alpha 4: News NLP|
| - Alpha 8: Multi-Factor Composite     - Alpha 6: Open Interest Reg    - Alpha 7: GARP    |
|                                                                       - Alpha 10: Cash Fl|
+-----------------------------------------------------------------------------------------+
```

### Diversity of Information Sources

Modern quantitative strategies must combine independent, orthogonal data sources to mitigate crowding and factor decay:

1. **Price Data:** Daily closing prices and percentage returns (`close`, `returns`).
2. **Fundamental Accounting Data:** Balance-sheet and income-statement fields (`operating_income`, `cap`).
3. **Derivatives & Options Surface:** Call/put implied volatilities across tenors, open interest distributions, and strike breakevens (`implied_volatility_call_30`, `implied_volatility_put_30`, `implied_volatility_call_10`, `implied_volatility_put_10`, `pcr_oi_20`, `call_breakeven_20`).
4. **Alternative & NLP Data:** Vector-embedded after-hours news feeds, social discussion frequency, and sentiment polarity (`nws12_afterhsz_sl`, `scl12_alltype_buzzvec`, `scl12_sentiment`).
5. **Analyst Consensus Models:** Proprietary Growth-at-a-Reasonable-Price valuation metrics and forward cash flow revision flags (`mdl177_garpanalystmodel_qgp_vfpriceratio`, `anl4_cff_flag`).
6. **Market Microstructure:** Session trading volume and multi-day volume moving averages (`volume`, `adv20`).

---

## 🛡 Research and Validation Considerations

Before deploying any mathematical alpha into simulated paper trading or live capital allocation, quantitative researchers must test against standard biases and structural headwinds:

### 1. Data Availability & Semantic Definitions
Proprietary alpha fields (e.g., `mdl177_...`, `scl12_...`, `anl4_...`) should have their exact units, point-in-time calculation rules, and updating frequencies verified in the platform's Data Dictionary to avoid misinterpreting sign conventions.

### 2. Look-Ahead Bias (Point-in-Time Data)
Financial and fundamental data must be strictly point-in-time. A historical backtest must not utilize financial statement data before the official public filing date (SEC 10-Q/10-K timestamp).

### 3. Survivorship Bias
Ensure backtest universes encompass historical delistings, bankruptcies, and corporate acquisitions. Evaluating strategies only on surviving companies artificially inflates Sharpe ratios and annual returns.

### 4. Transaction Costs & Turnover Decay
High-frequency reversal alphas (e.g., Alpha 1, Alpha 4, Alpha 8) generate significant portfolio turnover. Slippage models incorporating market impact, bid-ask spreads, and borrow costs for short legs must be applied:
$$\text{Net Return}_t = \text{Gross Return}_t - \text{Turnover}_t \times \Big(\text{Fee} + \text{Slippage}\Big)$$

### 5. Factor Risk Neutralization
Raw alphas can inadvertently take on passive exposures to systemic risk factors (e.g., Size, Value, Momentum, Industry, Market Beta). True alpha generation requires neutralizing signals against Barra-style factor risk models:
$$\alpha_{\text{pure}} = \alpha_{\text{raw}} - \mathbf{X}(\mathbf{X}^T\mathbf{X})^{-1}\mathbf{X}^T \alpha_{\text{raw}}$$

### 6. Robustness & Parameter Sensitivity Testing
Robust alphas should exhibit stability across parameter neighborhoods rather than isolated peaks of overfitted performance. Parameter sweeps should be executed:
- **Lookback Windows:** Test 3-day vs. 5-day reversals; 20-day vs. 30-day realized variance.
- **Threshold Ranges:** Test percentile boundaries (e.g., 0.60, 0.65, 0.70 in Alpha 8).
- **Sub-Period Splits:** Verify out-of-sample (OOS) consistency across distinct economic cycles (e.g., bull trends, bear markets, high-rate regimes).

---

## 🎯 Conclusion

The ten strategies in this portfolio demonstrate the transition from simple price-based heuristics to modern, multi-dimensional quantitative alphas. 

The true frontier of quantitative alpha generation lies not merely in compounding mathematical complexity, but in **identifying economically sound market hypotheses, verifying point-in-time data integrity, and synthesizing orthogonal information channels** into resilient, risk-neutralized investment signals.

---

## ⚠️ Documentation Disclaimer

*These reports and mathematical breakdowns document the submitted WorldQuant BRAIN-style expressions as written. They do not represent financial advice, nor do they guarantee future out-of-sample trading performance or commercial profitability. Readers and researchers must conduct exhaustive backtesting, transaction-cost modeling, and robustness checks before committing real-world investment capital.*
