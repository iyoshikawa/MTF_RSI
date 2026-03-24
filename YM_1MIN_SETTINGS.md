# YM/MYM 1-Minute Chart Settings Guide
### MTF RSI v7.6.0 (Signals) / v7.5.0 (Management)

---

## Quick Summary

YM on the 1-min is lower liquidity than ES/NQ, moves in chunks, and consolidates in
tight ranges. These settings tighten anti-chop filters, fix TP/SL ratios to maintain
positive expectancy, and reduce signal spam.

**Start with the 🔷 YM/MYM preset**, then apply these manual overrides.

---

## Signals Indicator — Manual Overrides

### Volume Filter
| Setting               | Default (YM preset) | Recommended 1-Min |
|----------------------|---------------------|-------------------|
| Volume Multiplier     | 1.2                 | **1.0**           |
| Enable Volume Filter  | true                | **true** (or OFF if trading MYM — volume data is unreliable on micros) |

### Anti-Chop Filters (Critical for 1-Min)
| Setting                           | Default | Recommended 1-Min |
|----------------------------------|---------|-------------------|
| Signal Cooldown (bars)            | 5       | **8–10**          |
| Grade Persistence (bars)          | 2       | **3**             |
| ADX Floor for B/B+                | 15.0    | **18.0**          |
| Suppress B/B+ During LOW Vol      | false   | **true**          |
| Block Counter-Trend on B/B+/A-    | true    | **true** (keep)   |
| Require RSI Ribbon for B/B+       | false   | **true**          |
| Gray Ribbon During Consolidation   | false   | **true**          |
| Consolidation Range (pts)          | 10.0    | **15.0–20.0**     |
| Consolidation Lookback (bars)      | 20      | **20** (keep)     |

### Session Config
| Setting        | Default               | Recommended 1-Min          |
|---------------|-----------------------|---------------------------|
| Session Config | 0930-1145,1330-1600   | **0930-1145,1330-1545**   |

Trim the last 15 minutes — MOC order flow creates noise, not tradeable signals.

### Settings to Leave Alone (YM Preset Handles These)
- EMA Length: 34 ✓
- Stoch OB/OS: 62/38 ✓
- Stoch Sell Level: 82 ✓
- ADX Threshold A+: 23 ✓
- ADX Threshold A: 18 ✓
- RSI Periods (5/13, 2/3 smoothing) ✓
- MTF Stack: auto-selects 5/15/60 on 1-min ✓

---

## Management Indicator — Manual Overrides

### TP/SL (Updated in v7.4.2)

The preset now ships with these values. **No manual override needed.**

| Grade | SL (pts) | TP (pts) | R:R     | Trade Style |
|-------|----------|----------|---------|-------------|
| A+    | 15       | 35       | 2.3:1   | Runner — partial at TP, trail the rest |
| A     | 15       | 25       | 1.7:1   | Runner — partial at TP, trail the rest |
| A-    | 12       | 18       | 1.5:1   | Standard — full exit at TP |
| B+    | 10       | 15       | 1.5:1   | Scalp — in and out |
| B     | 8        | 10       | 1.25:1  | Scalp — tight leash, 3–5 bars max |

### ATR TP (for A grades)
| Setting          | Recommended   |
|-----------------|---------------|
| Show ATR TP      | **true**      |
| ATR TP Multiplier | **2.0**      |

Use ATR-based TPs on A+/A grades. On high-ADR days the fixed 35pt target may be too
conservative — the ATR TP adapts automatically.

### Dollar Per Point
| Contract | $ Per Point |
|----------|-------------|
| MYM      | $0.50 (preset default) |
| Full YM  | **$5.00** (must override manually) |

**If you trade full YM, you MUST change this.** The preset defaults to MYM ($0.50).
At $5/point the sizing dashboard will be 10x off if you leave it.

### Runner Trail
| Setting        | Recommended     |
|---------------|-----------------|
| Show Runner    | **true**        |
| Trail Source   | **34 EMA**      |

YM trends are choppier than NQ — the 34 EMA gives enough room to ride the move without
getting stopped on pullbacks. The 21 HMA is too tight for 1-min YM.

### Max Pain
Scale to your account. At $5/point (full YM) with a 15pt A-grade stop, that's $75/contract
risk. At $400 max pain, you're looking at 5 contracts max on A+.

---

## Tick Reference

| Instrument | Tick Size | Ticks/Point | $/Tick (full) | $/Tick (micro) |
|-----------|-----------|-------------|---------------|----------------|
| YM/MYM    | 1 pt      | 1           | $5.00         | $0.50          |
| ES/MES    | 0.25 pt   | 4           | $12.50        | $1.25          |
| NQ/MNQ    | 0.25 pt   | 4           | $5.00         | $0.50          |
| CL        | 0.01      | 100/pt      | $10.00        | —              |

---

## What Changed in v7.5.0 (Management)

1. **YM SL tightened**: A+/A from 20→15, A- from 20→12, B+ from 15→10, B from 15→8
2. **YM TP widened for A grades**: A+ from 20→35, A from 20→25, A- from 15→18, B+ from 10→15
3. **Dollar per point tooltip** clarified for MYM vs full YM
4. **Tick multiplier** already correct on this branch (× 1 for YM vs × 4 for ES/NQ)

---

## Algo Windows (Optional but Recommended)

For YM 1-min, the highest-probability windows are:

| Window     | Time (ET)     | Notes                              |
|-----------|---------------|-------------------------------------|
| NY Open    | 0830–1000     | Best window — YM leads on economic data |
| NY PM      | 1330–1500     | Bond market closes 1500, YM reacts  |
| London     | 0200–0500     | Thinner but trends cleanly          |

Set Algo Mode to **TRADE WITH** and enable NY Open + NY PM at minimum.
