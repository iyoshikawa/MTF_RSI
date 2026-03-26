# VES Backtest Guide — Multi-Instrument Comparison
**Version:** 1.16.0

## ⚠️ Renko Backtest Disclaimer
TradingView Renko backtests fill at synthetic brick prices. Results show **directional edge** but dollar amounts are inflated by 30-50%. Use for **relative comparison** between settings, not absolute performance. Forward test on SIM via QuantLynk for real data.

---

## Instruments to Test

| Symbol | Preset | Brick | Base TF | $/pt Full | $/pt Micro | Commission |
|--------|--------|-------|---------|-----------|------------|------------|
| YM1! | 🔷 YM | 8 pts | 30s | $5/pt | — | $2.25/ct |
| MYM1! | 🔷 YM | 8 pts | 30s | — | $0.50/pt | $0.62/ct |
| NQ1! | 📈 NQ | 8 pts | 30s | $20/pt | — | $2.25/ct |
| MNQ1! | 📈 NQ | 8 pts | 30s | — | $2/pt | $0.62/ct |
| ES1! | 📊 ES | 4 pts | 30s | $50/pt | — | $2.25/ct |
| MES1! | 📊 ES | 4 pts | 30s | — | $5/pt | $0.62/ct |
| CL1! | 🛢️ CL | 0.08 | 30s | $1000/pt | — | $2.25/ct |
| MCL1! | 🛢️ CL | 0.08 | 30s | — | $100/pt | $1.25/ct |

---

## Strategy Settings per Instrument

### Strategy Properties
| Setting | Full-Size | Micro |
|---------|-----------|-------|
| Initial Capital | $25,000 | $5,000 |
| Commission | $2.25/ct | $0.62/ct ($1.25 MCL) |
| Slippage | 1 tick | 1 tick |
| Pyramiding | 2 | 2 |
| Default Qty | 1 | 1 |

### Strategy Inputs
| Setting | YM/MYM | NQ/MNQ | ES/MES | CL/MCL |
|---------|--------|--------|--------|--------|
| Preset | 🔷 YM | 📈 NQ | 📊 ES | 🛢️ CL |
| TP | 40 pts | 40 pts | 8 pts | 0.20 |
| Trail Mode | Fixed | Fixed | Fixed | Fixed |
| Trail | 40 pts | 40 pts | 8 pts | 0.20 |
| BE Enable | ON | ON | ON | ON |
| BE Bricks | 3 | 3 | 3 | 3 |
| BE Ticks | 3 | 3 | 3 | 3 |
| Min Wave Length | 3 | 3 | 3 | 3 |
| BB Length | 10 | 10 | 10 | 10 |
| BB StdDev | 1.2 | 1.2 | 1.2 | 1.4 |
| BB TF | 3 min | 3 min | 3 min | 5 min |
| BB Squeeze | ON | ON | ON | ON |
| ADR Filter | ON | ON | ON | ON |
| Session | 0930-1145,1330-1600 | same | same | 0900-1430 |
| Trend Filter | ON | ON | ON | ON |
| Vol Method | Wave Compare | same | same | same |
| DEBUG | OFF | OFF | OFF | OFF |
| All Gates | ON | ON | ON | ON |

---

## Test Procedure

### Step 1: Baseline — All Protections ON
Load each instrument with settings above. Record stats.

### Step 2: Volume Method Comparison
For each instrument, switch only the Detection Method and record:
1. Wave Compare
2. Volume ROC
3. Z-Score
4. OFF (flip + BB only)

### Step 3: Trail Mode Comparison (optional)
For the best volume method, test:
1. Fixed (original)
2. ATR (1.5x multiplier)
3. Wider of Both

---

## Results Template

### [INSTRUMENT] — Renko [BRICK] / [BASE TF]

**Volume Method Comparison:**

| Method | Trades | Win % | Net P&L | PF | Expectancy | Avg Win | Avg Loss | Max DD |
|--------|--------|-------|---------|-----|------------|---------|----------|--------|
| Wave Compare | | | | | | | | |
| Volume ROC | | | | | | | | |
| Z-Score | | | | | | | | |
| OFF (flip+BB) | | | | | | | | |

**Trail Mode Comparison (using best vol method):**

| Trail Mode | Trades | Win % | Net P&L | PF | Expectancy | Max DD |
|------------|--------|-------|---------|-----|------------|--------|
| Fixed 40pt | | | | | | |
| ATR 1.5x | | | | | | |
| Wider of Both | | | | | | |

---

## Pre-Built Result Tables

### YM1! — Renko Trad 8 / 30s

| Method | Trades | Win % | Net P&L | PF | Expectancy | Avg Win | Avg Loss | Max DD |
|--------|--------|-------|---------|-----|------------|---------|----------|--------|
| Wave Compare | | | | | | | | |
| Volume ROC | | | | | | | | |
| Z-Score | | | | | | | | |
| OFF (flip+BB) | | | | | | | | |

### NQ1! — Renko Trad 8 / 30s

| Method | Trades | Win % | Net P&L | PF | Expectancy | Avg Win | Avg Loss | Max DD |
|--------|--------|-------|---------|-----|------------|---------|----------|--------|
| Wave Compare | | | | | | | | |
| Volume ROC | | | | | | | | |
| Z-Score | | | | | | | | |
| OFF (flip+BB) | | | | | | | | |

### ES1! — Renko Trad 4 / 30s

| Method | Trades | Win % | Net P&L | PF | Expectancy | Avg Win | Avg Loss | Max DD |
|--------|--------|-------|---------|-----|------------|---------|----------|--------|
| Wave Compare | | | | | | | | |
| Volume ROC | | | | | | | | |
| Z-Score | | | | | | | | |
| OFF (flip+BB) | | | | | | | | |

### CL1! — Renko Trad 0.08 / 30s

| Method | Trades | Win % | Net P&L | PF | Expectancy | Avg Win | Avg Loss | Max DD |
|--------|--------|-------|---------|-----|------------|---------|----------|--------|
| Wave Compare | | | | | | | | |
| Volume ROC | | | | | | | | |
| Z-Score | | | | | | | | |
| OFF (flip+BB) | | | | | | | | |

---

## What to Look For

**Best method per instrument:** Highest profit factor at reasonable trade count.

**Trade frequency:** Z-Score will likely produce very few on Renko. OFF the most. Wave Compare and ROC in between.

**Stability:** If results are similar across methods, the edge is in exit management (TP/Trail/BE), not entry detection. That's robust.

**Protection impact:** Compare with BB Squeeze ON vs OFF, and Min Wave 3 vs 1. The protections should reduce trade count but improve PF and reduce DD.

**Cross-instrument:** If one instrument consistently outperforms, prioritize SIM testing there first.
