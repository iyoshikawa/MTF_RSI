# VES Backtest Guide — Multi-Instrument Comparison
**Version:** 2.0.0

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

## Strategy Settings per Instrument (v2.0.1 Defaults)

### Strategy Properties
| Setting | Full-Size | Micro |
|---------|-----------|-------|
| Initial Capital | $25,000 | $5,000 |
| Commission | $2.25/ct | $0.62/ct ($1.25 MCL) |
| Slippage | 3 ticks | 3 ticks |
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
| EMA Stack | 21/34/50 | same | same | same |
| HMA Length | 20 | 20 | 20 | 20 |
| Trend Gate (HMA×EMA21) | ON | ON | ON | ON |
| BB Length | 10 | 10 | 10 | 10 |
| BB StdDev | 1.2 | 1.2 | 1.2 | 1.4 |
| BB TF | 3 min | 3 min | 3 min | 5 min |
| BB Squeeze | ON (0.60 ratio) | same | same | same |
| Chop Detector | ON (avg ≥5b) | same | same | same |
| XWave Ratio | 0.50 | same | same | same |
| ADR Filter | ON | ON | ON | ON |
| Session | 0930-1145,1330-1600 | same | same | 0900-1430 |
| Vol Method | Wave Compare | same | same | same |
| DEBUG | OFF | OFF | OFF | OFF |
| All Gates | ON | ON | ON | ON |

---

## Test Procedure

### Step 1: Baseline — All Protections ON (v2.0.1 defaults)
Load each instrument with settings above. Record stats.

### Step 2: Volume Method Comparison
For each instrument, switch only the Detection Method and record:
1. Wave Compare
2. Volume ROC
3. Z-Score
4. OFF (flip + BB only)

### Step 3: New Feature Isolation (v2.0.1)
Test each new feature by toggling its debug gate:
1. **Chop Detector impact:** Toggle 🔧 Require Chop Detector ON/OFF
2. **Trend Gate impact:** Toggle 🔧 Require Trend Gate ON/OFF
3. **BB Squeeze ratio:** Compare 0.45 / 0.60 / 0.70

### Step 4: Cross-Wave Volume Ratio Tuning
Compare confluence quality at different thresholds:
1. 0.40 (very tight — only hollow pops)
2. 0.50 (default)
3. 0.65 (permissive)

### Step 5: Trail Mode Comparison (optional)
For the best volume method, test:
1. Fixed (original)
2. ATR (1.5x multiplier)
3. Wider of Both

---

## Results Template

### [INSTRUMENT] — Renko [BRICK] / [BASE TF]

**v2.0.1 Baseline:**

| Config | Trades | Win % | Net P&L | PF | Expectancy | Avg Win | Avg Loss | Max DD |
|--------|--------|-------|---------|-----|------------|---------|----------|--------|
| All defaults | | | | | | | | |
| Chop OFF | | | | | | | | |
| Trend Gate OFF | | | | | | | | |
| Both OFF | | | | | | | | |

**Volume Method Comparison:**

| Method | Trades | Win % | Net P&L | PF | Expectancy | Avg Win | Avg Loss | Max DD |
|--------|--------|-------|---------|-----|------------|---------|----------|--------|
| Wave Compare | | | | | | | | |
| Volume ROC | | | | | | | | |
| Z-Score | | | | | | | | |
| OFF (flip+BB) | | | | | | | | |

**XWave Ratio Comparison (note confluence counts, not P&L):**

| Threshold | Avg Conf Long | Avg Conf Short | Notes |
|-----------|--------------|----------------|-------|
| 0.40 | | | |
| 0.50 | | | |
| 0.65 | | | |

---

## What to Look For

**Chop Detector value:** Compare trade count and PF with chop ON vs OFF. If PF improves and trade count drops, the detector is filtering real chop.

**Trend Gate value:** Compare with HMA×EMA21 gate ON vs OFF. Should reduce losers more than it reduces winners.

**BB Squeeze adaptive:** Should auto-calibrate. If results are similar across instruments without manual width tuning, the adaptive ratio is working.

**Cross-Wave Ratio:** Since it's a confluence (not gate), check if high-confluence signals (6+/7) outperform low ones (4/7). If yes, the XWave ratio adds real information.

**Stability:** If results are similar across methods, the edge is in exit management (TP/Trail/BE), not entry detection. That's robust.

**Cross-instrument:** If one instrument consistently outperforms, prioritize SIM testing there first.
