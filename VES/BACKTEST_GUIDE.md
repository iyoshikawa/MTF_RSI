# VES Backtest Guide — Multi-Instrument Comparison

## ⚠️ Renko Backtest Disclaimer
TradingView Renko backtests fill at synthetic brick prices, not real market prices. Results show **directional edge** but dollar amounts are inflated by 30-50%. Use these numbers for **relative comparison** between instruments and settings, not as absolute performance expectations.

For real performance data, forward test on SIM via QuantLynk.

---

## Instruments to Test

| Symbol | Preset | Chart | Brick | Base TF | $/pt Full | $/pt Micro | Commission |
|--------|--------|-------|-------|---------|-----------|------------|------------|
| YM1! | 🔷 YM | Renko Trad 8 | 8 pts | 30s | $5/pt | — | $2.25/ct |
| MYM1! | 🔷 YM | Renko Trad 8 | 8 pts | 30s | — | $0.50/pt | $0.62/ct |
| NQ1! | 📈 NQ | Renko Trad 8 | 8 pts | 30s | $20/pt | — | $2.25/ct |
| MNQ1! | 📈 NQ | Renko Trad 8 | 8 pts | 30s | — | $2/pt | $0.62/ct |
| ES1! | 📊 ES | Renko Trad 4 | 4 pts | 30s | $50/pt | — | $2.25/ct |
| MES1! | 📊 ES | Renko Trad 4 | 4 pts | 30s | — | $5/pt | $0.62/ct |
| CL1! | 🛢️ CL | Renko Trad 8 | 0.08 | 30s | $1000/pt | — | $2.25/ct |
| MCL1! | 🛢️ CL | Renko Trad 8 | 0.08 | 30s | — | $100/pt | $1.25/ct |

**Notes:**
- Micro contracts (MYM, MNQ, MES, MCL) use the same price chart as full-size. Same preset, same TP/Trail points. Only commission and dollar-per-point differ.
- ES uses 4-pt brick (not 8) because ES has smaller range than YM/NQ.
- CL brick size is 0.08 (8 ticks at 0.01 per tick).

---

## Backtest Settings per Instrument

### Strategy Properties (in TradingView Strategy Tester settings)
| Setting | Full-Size | Micro |
|---------|-----------|-------|
| Initial Capital | $25,000 | $5,000 |
| Commission | $2.25/contract | $0.62/contract (MNQ/MYM/MES), $1.25 (MCL) |
| Slippage | 1 tick | 1 tick |
| Pyramiding | 2 | 2 |
| Default Qty | 1 | 1 |

### Strategy Inputs (same for full and micro)
| Setting | YM/MYM | NQ/MNQ | ES/MES | CL/MCL |
|---------|--------|--------|--------|--------|
| Preset | 🔷 YM | 📈 NQ | 📊 ES | 🛢️ CL |
| TP | 40 pts | 40 pts | 8 pts | 0.20 |
| Trail | 40 pts | 40 pts | 8 pts | 0.20 |
| BB Length | 10 | 10 | 10 | 10 |
| BB StdDev | 1.2 | 1.2 | 1.2 | 1.2 |
| BB TF | 3 min | 3 min | 3 min | 3 min |
| Vol Method | Wave Compare | Wave Compare | Wave Compare | Wave Compare |
| Session | 0930-1145,1330-1600 | 0930-1145,1330-1600 | 0930-1145,1330-1600 | 0900-1430 |
| DEBUG | OFF | OFF | OFF | OFF |
| All Gates | ON | ON | ON | ON |

---

## How to Run Each Backtest

1. Open the instrument chart (e.g. NQ1!)
2. Set chart to Renko Traditional, brick size per table above, 30s base
3. Add VES strategy
4. In Settings → Properties: set commission and initial capital per table
5. In Settings → Inputs: select the correct instrument preset
6. Set Detection Method to "Wave Compare"
7. Make sure DEBUG is OFF, all gate toggles ON
8. Wait for strategy to load, then record the results below

**Repeat for each of the 4 detection methods:**
- Wave Compare
- Volume ROC
- Z-Score
- OFF (flip + BB only)

---

## Results Template

Copy this table and fill in after each test run.

### [INSTRUMENT] — [SYMBOL] — Renko [BRICK] [BASE TF]

| Method | Trades | Win % | Net P&L | Prof Factor | Expectancy | Avg Win | Avg Loss | Max DD |
|--------|--------|-------|---------|-------------|------------|---------|----------|--------|
| Wave Compare | | | | | | | | |
| Volume ROC | | | | | | | | |
| Z-Score | | | | | | | | |
| OFF (flip+BB) | | | | | | | | |

---

### YM1! — Renko Trad 8 / 30s

| Method | Trades | Win % | Net P&L | Prof Factor | Expectancy | Avg Win | Avg Loss | Max DD |
|--------|--------|-------|---------|-------------|------------|---------|----------|--------|
| Wave Compare | | | | | | | | |
| Volume ROC | | | | | | | | |
| Z-Score | | | | | | | | |
| OFF (flip+BB) | | | | | | | | |

### NQ1! — Renko Trad 8 / 30s

| Method | Trades | Win % | Net P&L | Prof Factor | Expectancy | Avg Win | Avg Loss | Max DD |
|--------|--------|-------|---------|-------------|------------|---------|----------|--------|
| Wave Compare | | | | | | | | |
| Volume ROC | | | | | | | | |
| Z-Score | | | | | | | | |
| OFF (flip+BB) | | | | | | | | |

### ES1! — Renko Trad 4 / 30s

| Method | Trades | Win % | Net P&L | Prof Factor | Expectancy | Avg Win | Avg Loss | Max DD |
|--------|--------|-------|---------|-------------|------------|---------|----------|--------|
| Wave Compare | | | | | | | | |
| Volume ROC | | | | | | | | |
| Z-Score | | | | | | | | |
| OFF (flip+BB) | | | | | | | | |

### CL1! — Renko Trad 8 / 30s

| Method | Trades | Win % | Net P&L | Prof Factor | Expectancy | Avg Win | Avg Loss | Max DD |
|--------|--------|-------|---------|-------------|------------|---------|----------|--------|
| Wave Compare | | | | | | | | |
| Volume ROC | | | | | | | | |
| Z-Score | | | | | | | | |
| OFF (flip+BB) | | | | | | | | |

---

## What to Look For

**Best method per instrument:** Compare profit factor and expectancy across the 4 methods. The one with highest PF at reasonable trade count wins.

**Trade frequency:** Z-Score will likely produce very few signals on Renko. OFF will produce the most. Wave Compare and ROC should be in between.

**Stability:** If an instrument's results are wildly different between methods, the edge is likely coming from the TP/Trail exit management, not the entry detection. That's fine — it means the exits are doing the work.

**Cross-instrument:** If one instrument consistently outperforms across all methods, prioritize forward testing on that instrument first.
