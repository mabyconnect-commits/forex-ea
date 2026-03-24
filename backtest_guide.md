# 4rexbot — Backtesting Guide

> How to backtest the ICT Smart Money EA in MT5 Strategy Tester  
> Estimated time: 3–5 days for a thorough multi-symbol test

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Day 1 — Setup & First Test Run](#day-1)
3. [Day 2 — Multi-Symbol Testing](#day-2)
4. [Day 3 — Parameter Optimisation](#day-3)
5. [Day 4 — Stress Testing & Edge Cases](#day-4)
6. [Day 5 — Analysis & Forward Testing Prep](#day-5)
7. [Reading the Results](#reading-results)
8. [Common Issues](#common-issues)

---

## Prerequisites

Before starting:

- [ ] MT5 installed and logged in to a demo or real account
- [ ] `4rexbot.mq5` compiled successfully (0 errors)
- [ ] Sufficient historical data downloaded (see below)
- [ ] Broker provides tick data (not just M1 data)

### Download Historical Data

Good backtest quality requires real tick data:

1. In MT5, open **Tools → History Center**
2. For each symbol (XAUUSD, BTCUSD, USOIL, GBPUSD):
   - Select the symbol
   - Click **Download**
   - Wait for completion (can take several minutes per symbol)
3. Alternatively, use **Tick Data Suite** (third-party tool) for higher-quality tick data

---

## Day 1 — Setup & First Test Run {#day-1}

### Goal: Confirm the EA runs without errors on a single symbol

### Step 1 — Open Strategy Tester

1. In MT5: **View → Strategy Tester** (or press Ctrl+R)
2. The Strategy Tester panel opens at the bottom

### Step 2 — Configure Test Settings

| Setting | Value |
|---------|-------|
| Expert Advisor | 4rexbot |
| Symbol | XAUUSD |
| Period | M15 (15-minute chart) |
| Model | **Every tick based on real ticks** (best quality) |
| Date From | 6 months ago |
| Date To | Today |
| Optimization | Disabled |
| Deposit | $10,000 (or your actual account size) |
| Currency | USD |
| Leverage | 1:100 (match your broker) |

> ⚠️ If "Every tick" data is not available, use **OHLC on M1** as fallback. Avoid "Open prices only" — it will give unrealistic results for this strategy.

### Step 3 — Set EA Input Parameters

Click the **Properties** button to open EA inputs:

```
AccountType        = PROP
PropRiskPercent    = 1.0
TradeXAUUSD        = true
TradeBTCUSD        = false  ← disable others for Day 1 test
TradeUSOIL         = false
TradeGBPUSD        = false
MaxPairsOpen       = 1
MagicNumber        = 202401
Lookback4H         = 10
LookbackEntry      = 20
FVGMinPips         = 5.0
OBBufferPips       = 2.0
BreakevenRR        = 2.0
PartialCloseRR     = 3.0
TelegramBotToken   = (leave empty for backtesting)
TelegramChatID     = (leave empty for backtesting)
```

### Step 4 — Run the Test

1. Click **Start**
2. Watch the **Graph** tab for equity curve
3. Watch the **Journal** tab for trade entries and any errors
4. A typical 6-month test on M15 takes 5–30 minutes depending on hardware

### Step 5 — Check for Errors

In the **Journal** tab, look for:
- ✅ No `TRADE_RETCODE_REJECT` errors
- ✅ Trades are opening and closing
- ✅ No infinite loops or crashes
- ⚠️ If you see "symbol not found" errors, the symbol name may differ on your broker (e.g., `XAUUSD.` or `XAUUSDm`)

### Day 1 Deliverable

- EA runs without crashes
- At least some trades are opened over 6 months
- Note the basic results: total trades, win rate, profit factor

---

## Day 2 — Multi-Symbol Testing {#day-2}

### Goal: Test all 4 symbols and verify pair limits work correctly

### Method: Multi-Currency Backtest

MT5's Strategy Tester can test on one symbol at a time in standard mode. For multi-symbol backtesting:

**Option A — MT5 Built-in (simulated)**  
Run the test on XAUUSD with all 4 symbols enabled. MT5 will load historical data for the other symbols as sub-data. Results may be approximate.

**Option B — Separate Tests**  
Run 4 separate backtests, one per symbol, then compare results:

| Symbol | Period | Notes |
|--------|--------|-------|
| XAUUSD | M15 | Most liquid, highest spread during news |
| BTCUSD | M15 | Highly volatile, check lot size clamps |
| USOIL | M15 | Sensitive to inventory data (Wednesdays) |
| GBPUSD | M15 | Classic forex, tightest spreads |

### Testing Procedure per Symbol

1. Open Strategy Tester
2. Change Symbol to each instrument
3. Adjust deposit to be realistic:
   - XAUUSD: $5,000+
   - BTCUSD: $10,000+ (requires larger margin)
   - USOIL: $3,000+
   - GBPUSD: $2,000+
4. Run 6-month backtest
5. Export results (right-click on Results tab → Save as Report)

### What to Look For

| Metric | Acceptable | Good | Excellent |
|--------|-----------|------|-----------|
| Win Rate | >45% | >55% | >65% |
| Profit Factor | >1.2 | >1.5 | >2.0 |
| Max Drawdown | <20% | <10% | <5% |
| Avg R:R | >1.5 | >2.0 | >3.0 |

### Day 2 Deliverable

- Results for all 4 symbols
- Identify which symbols perform best
- Identify any symbols to disable or tune separately

---

## Day 3 — Parameter Optimisation {#day-3}

### Goal: Find the best combination of key parameters

### Parameters Worth Optimising

| Parameter | Test Range | Step |
|-----------|-----------|------|
| `Lookback4H` | 5–20 | 5 |
| `LookbackEntry` | 10–30 | 5 |
| `FVGMinPips` | 3–10 | 1 |
| `OBBufferPips` | 1–5 | 1 |
| `BreakevenRR` | 1.5–3.0 | 0.5 |
| `PartialCloseRR` | 2.5–5.0 | 0.5 |

### How to Run Optimisation

1. In Strategy Tester, change **Optimization** to:  
   `Slow complete algorithm (best result)`
2. Click the **Inputs** tab in Strategy Tester
3. For each parameter you want to optimise:
   - Tick the checkbox on the left
   - Set Start, Step, Stop values
4. Change **Optimization criterion** to:
   - **Balance max + Profit Factor** (recommended)
   - Or **Balance drawdown** if you prioritise safety
5. Click **Start**

> ⚠️ Optimisation can take **hours to days**. Start with only 2–3 parameters at a time.

### Avoiding Overfitting

- Test on a **different time period** than what you optimised on
- Example: optimise on Jan–Jun, validate on Jul–Dec
- If results drop significantly on out-of-sample data, the parameters are overfit
- Prefer parameters that perform consistently across different periods over those that excel in one period

### Day 3 Deliverable

- Optimised parameter set for top 2 symbols
- Out-of-sample validation results
- Document your best parameter combination

---

## Day 4 — Stress Testing & Edge Cases {#day-4}

### Goal: Test EA behaviour in extreme market conditions

### Test Scenarios

#### 1. High Volatility Periods

Re-run your backtest specifically during known volatile periods:
- **COVID crash:** Feb–Mar 2020
- **Russia-Ukraine:** Feb 2022
- **2022 rate hikes:** Apr–Dec 2022

Set Strategy Tester dates accordingly and observe:
- Does the lot sizing stay sane?
- Are SLs hit before TPs excessively?
- Does the news filter help reduce losses?

#### 2. Low Spread vs High Spread Conditions

In Strategy Tester **Properties** tab:
- Change **Spread**: try 10 (tight) vs 50 (wide) vs 100 (crisis-level)
- Compare results — if profitability collapses with wider spreads, the strategy has insufficient edge

#### 3. Breakeven + Partial Close Logic

To verify the trail/partial logic:
1. Enable **Visualization** in Strategy Tester
2. Watch individual trades in real-time (slower, but visual)
3. Confirm:
   - SL moves to entry when trade reaches 2:1 R:R
   - 50% close happens at 3:1 R:R
   - Remainder stays open to TP

#### 4. Session Filter Verification

In the Journal tab, verify no trades open:
- On Saturday or Sunday
- Between 22:00–07:00 UTC

Filter the journal by time to check this manually.

#### 5. Max Pairs / Max Trades Per Pair

Run a 1-month test with high risk and verify the EA never has:
- More than `MaxPairsOpen` pairs simultaneously open
- More than `MaxTradesPerPair` trades on one symbol

### Day 4 Deliverable

- EA handles volatility without blowing account
- Spread sensitivity analysis complete
- Logic for breakeven/partial close verified visually
- Session filter confirmed working

---

## Day 5 — Analysis & Forward Testing Prep {#day-5}

### Goal: Interpret full results and prepare for live/demo forward testing

### Generate Final Backtest Report

1. Run final optimised test on the full available history (1–3 years)
2. In **Results** tab, right-click → **Save as Report** → HTML
3. Review all metrics

### Key Metrics to Document

```
Symbol:          ___________
Period Tested:   ___ to ___
Total Trades:    ___
Profit Trades:   ___ (___%)
Loss Trades:     ___ (___%)
Profit Factor:   ___
Expected Payoff: ___ per trade
Max Drawdown:    ___% ($___) 
Sharpe Ratio:    ___
Net Profit:      $___
```

### Comparing Strategy Across Symbols

Create a comparison table from your Day 2 results:

| Symbol | Trades | Win% | PF | Max DD | Net Profit |
|--------|--------|------|----|--------|-----------|
| XAUUSD | | | | | |
| BTCUSD | | | | | |
| USOIL | | | | | |
| GBPUSD | | | | | |

**Recommendation:** Run live only the symbols with PF > 1.3 and Max DD < 15%.

### Forward Testing on Demo Account

Before going live:
1. Attach EA to a **demo account** with realistic balance
2. Run for **minimum 4 weeks** (1 full month)
3. Compare demo results to backtest expectations:
   - Win rate within ±10% of backtest?
   - Drawdown within range?
   - Trades occurring at expected frequency?
4. Only move to live funding when demo results are consistent

### Go-Live Checklist

- [ ] Backtest completed for all symbols
- [ ] Parameters optimised and validated out-of-sample
- [ ] EA running on demo for 4+ weeks
- [ ] Demo results match backtest expectations
- [ ] Telegram notifications tested and working
- [ ] VPS configured and stable
- [ ] Risk % set conservatively for first 30 live trades
- [ ] Drawdown limit agreed with yourself (e.g. stop at -15%)

---

## Reading the Results {#reading-results}

### Key Metrics Explained

| Metric | What It Means |
|--------|--------------|
| **Profit Factor** | Gross profit / Gross loss. >1.5 is good, >2.0 is excellent |
| **Expected Payoff** | Average profit per trade. Must be positive |
| **Max Drawdown** | Largest peak-to-trough drop. Keep under 20% for prop firms |
| **Sharpe Ratio** | Risk-adjusted return. >1.0 is acceptable |
| **Recovery Factor** | Net profit / Max drawdown. >2.0 means strategy recovers well |

### Equity Curve Shapes

- **Smooth upward slope** ✅ — Consistent, low variance strategy
- **Jagged but trending up** ⚠️ — High variance, expect emotional difficulty live
- **Flat then spike** ⚠️ — Likely curve-fitted to a specific market regime
- **Declining or choppy** ❌ — Strategy does not have edge on this symbol/period

---

## Common Issues {#common-issues}

### "Not enough bars"

- Download more history: **Tools → History Center**
- Reduce `Lookback4H` or `LookbackEntry`

### Zero trades in backtest

- Check session filter — are you testing on weekends only?
- Check symbol name matches exactly (some brokers add suffix: `XAUUSD.raw`)
- Reduce `FVGMinPips` or `OBBufferPips` — thresholds may be too strict

### Unrealistic results (>500% in 6 months)

- May be curve-fitting — validate on out-of-sample data
- Check lot size calculation — may be over-sizing due to incorrect `SYMBOL_POINT`

### "Spread too large" errors

- Set a fixed spread override in Strategy Tester Properties
- Use your broker's actual average spread for the symbol

### Strategy Tester crashes

- Reduce optimization parameters tested simultaneously
- Close other applications — optimisation is memory-intensive
- Try "Fast genetic based algorithm" instead of "Slow complete"

---

*4rexbot Backtest Guide — v1.0*  
*Strategy: ICT Order Blocks | FVG | Breaker Blocks | 4H Trend Bias*
