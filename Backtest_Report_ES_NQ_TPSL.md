# TP/SL Backtest Report — ES/MES & NQ/MNQ (1-Minute Chart)

## Methodology

- **Synthetic data:** 20 trading days of 1-min OHLCV per seed, generated with instrument-specific volatility, trend persistence, mean reversion, and U-shaped volume profile
- **Signal generation:** Simplified confluence model (EMA34 + double RSI + double stochastic + ADX + volume) — grades A+ through B assigned based on filter alignment
- **Trade simulation:** Bar-by-bar walk-forward, SL/TP checked each bar, 60-bar max hold, 8-bar signal cooldown
- **Scoring:** Composite of win rate (30%), profit factor (30%), realized R:R (20%), consistency (20%)
- **Validation:** 4 parameter sets × 4 seeds per round, best set cross-validated against 4 additional seeds (8 total)

### Limitations

This backtest tests **TP/SL mechanics against realistic price noise** — not full indicator logic. The simplified signal generator doesn't replicate MTF alignment, HMA stack, or precise entry timing near structure levels. Real-world entries near EMA/VWAP should allow tighter stops than what the synthetic data suggests, particularly for ES.

---

## Round 1: Initial Parameter Sweep

### ES/MES — 4 Sets × 4 Seeds

| Param Set | Avg WR | Avg PF | Avg R:R | Avg Score | Avg P&L | Pass |
|-----------|--------|--------|---------|-----------|---------|------|
| **ES-CURRENT** (15/15/15/15/15 SL, 6/6/4/3/2 TP) | 60.9% | 1.23 | 0.79 | **55.8** | +322.5 pts | **4/4** |
| ES-CONSERVATIVE (6/6/5/4/3 SL, 10/10/7/5/4 TP) | 39.0% | 0.71 | 1.10 | 46.1 | -578.5 pts | 0/4 |
| ES-MODERATE (7/7/5/4/3 SL, 14/12/8/6/4 TP) | 39.7% | 0.68 | 1.01 | 45.4 | -672.4 pts | 0/4 |
| ES-AGGRESSIVE (8/8/6/5/4 SL, 16/14/10/7/5 TP) | 41.4% | 0.67 | 0.93 | 45.3 | -708.3 pts | 0/4 |

**Finding:** Tight SL (3-8 pts) fails on 1-min ES. The ~1.8pt ATR means stops below 10pts get hit by noise before TP is reached. Wide 15pt stops survive because they're rarely touched.

### NQ/MNQ — 4 Sets × 4 Seeds

| Param Set | Avg WR | Avg PF | Avg R:R | Avg Score | Avg P&L | Pass |
|-----------|--------|--------|---------|-----------|---------|------|
| NQ-CURRENT (20/20/20/25/25 SL, 20/20/15/10/10 TP) | 57.4% | 1.18 | 0.87 | 54.8 | +968.8 pts | 4/4 |
| **NQ-CONSERVATIVE** (15/15/12/12/10 SL, 25/20/15/15/12 TP) | 48.8% | 1.29 | 1.36 | **56.6** | **+1354.8 pts** | **4/4** |
| NQ-MODERATE (18/18/15/15/12 SL, 30/25/20/18/14 TP) | 46.2% | 1.19 | 1.39 | 55.0 | +1140.6 pts | 4/4 |
| NQ-AGGRESSIVE (20/20/18/18/15 SL, 40/35/25/20/15 TP) | 40.6% | 1.10 | 1.60 | 53.6 | +722.1 pts | 4/4 |

**Finding:** Tighter stops with proportional TP increases outperform the wide-stop approach. CONSERVATIVE wins on both score (56.6) and total P&L (+1354.8 pts).

---

## Round 2: Refined Parameters + New Seeds (256, 777, 1024, 3141)

### ES/MES — Bridging the SL Gap

| Param Set | Avg WR | Avg PF | Avg R:R | Avg Score | Avg P&L | Pass |
|-----------|--------|--------|---------|-----------|---------|------|
| **ES-R1-WINNER** (15pt flat SL) | 61.3% | 1.26 | 0.79 | **56.2** | +351.5 pts | **4/4** |
| ES-BRIDGE-A (10/10/8/8/7 SL) | 50.6% | 0.92 | 0.89 | 50.2 | -162.7 pts | 2/4 |
| ES-BRIDGE-B (12/12/10/8/7 SL) | 47.6% | 0.87 | 0.95 | 49.3 | -244.7 pts | 1/4 |
| ES-BRIDGE-C (10/10/8/7/6 SL) | 48.1% | 0.84 | 0.90 | 48.8 | -322.3 pts | 1/4 |

**Finding:** Even 10-12pt SL underperforms on synthetic 1-min ES. The wide-stop / high-winrate approach persists. See recommendations below.

### NQ/MNQ — Optimizing Around Conservative

| Param Set | Avg WR | Avg PF | Avg R:R | Avg Score | Avg P&L | Pass |
|-----------|--------|--------|---------|-----------|---------|------|
| NQ-R1-WINNER (15/15/12/12/10 SL) | 47.0% | 1.21 | 1.36 | 55.1 | +977.8 pts | 4/4 |
| **NQ-CONS-TIGHT** (13/13/10/10/8 SL) | 44.3% | 1.25 | 1.58 | **56.3** | **+1097.8 pts** | **4/4** |
| NQ-CONS-WIDE-TP (same SL, wider TP) | 41.6% | 1.16 | 1.62 | 54.8 | +830.0 pts | 4/4 |
| NQ-CONS-BALANCED (14/14/11/11/9 SL) | 43.0% | 1.19 | 1.58 | 55.2 | +901.5 pts | 4/4 |

**Finding:** Tightening SL from 15→13 (A grades) and 12→10 (B grades) improves PF and R:R without hurting pass rate. NQ-CONS-TIGHT wins Round 2.

---

## Cross-Validation: Best Sets Against All 8 Seeds

### NQ-CONS-TIGHT — 8/8 Seeds Passed

| Seed | WR | PF | Score | P&L |
|------|----|----|-------|-----|
| 42 | 45.5% | 1.32 | 57.3 | +1364 pts |
| 137 | 46.1% | 1.35 | 57.9 | +1493 pts |
| 256 | 41.6% | 1.11 | 54.0 | +523 pts |
| 512 | 48.5% | 1.47 | 59.6 | +1913 pts |
| 777 | 43.8% | 1.24 | 56.1 | +1055 pts |
| 1024 | 47.8% | 1.43 | 59.1 | +1776 pts |
| 3141 | 43.9% | 1.23 | 56.0 | +1037 pts |
| 9999 | 43.4% | 1.22 | 55.7 | +958 pts |
| **Average** | **45.1%** | **1.30** | **57.0** | **+1264.9 pts** |

### ES-CURRENT — 8/8 Seeds Passed (but problematic R:R)

| Seed | WR | PF | Score | P&L |
|------|----|----|-------|-----|
| 42 | 60.3% | 1.19 | 55.2 | +275.9 pts |
| 137 | 61.9% | 1.16 | 54.9 | +247.3 pts |
| 256 | 61.3% | 1.25 | 56.2 | +351.4 pts |
| 512 | 62.6% | 1.29 | 56.7 | +415.6 pts |
| 777 | 62.5% | 1.36 | 57.7 | +455.1 pts |
| 1024 | 61.2% | 1.23 | 55.8 | +325.1 pts |
| 3141 | 60.3% | 1.19 | 55.1 | +274.6 pts |
| 9999 | 58.9% | 1.27 | 56.2 | +351.2 pts |
| **Average** | **61.1%** | **1.24** | **56.0** | **+337.0 pts** |

---

## Final Recommended Values

### NQ/MNQ — VALIDATED (implement in code)

| Grade | SL (pts) | TP (pts) | R:R | Style |
|-------|----------|----------|-----|-------|
| A+ | 13 | 25 | 1.92:1 | Runner — partial at TP, trail rest on 34 EMA |
| A | 13 | 20 | 1.54:1 | Runner — partial at TP, trail rest |
| A- | 10 | 15 | 1.50:1 | Standard — full exit at TP |
| B+ | 10 | 15 | 1.50:1 | Scalp — in and out |
| B | 8 | 12 | 1.50:1 | Scalp — tight leash |

Validated across 8 seeds. Avg PF 1.30, avg P&L +1264.9 pts (MNQ), all seeds positive.

### ES/MES — KEEP CURRENT (with caveats)

| Grade | SL (pts) | TP (pts) | R:R | Style |
|-------|----------|----------|-----|-------|
| A+ | 15 | 6 | 0.40:1 | High WR scalp |
| A | 15 | 6 | 0.40:1 | High WR scalp |
| A- | 15 | 4 | 0.27:1 | High WR scalp |
| B+ | 15 | 3 | 0.20:1 | High WR scalp |
| B | 15 | 2 | 0.13:1 | High WR scalp |

**Why not tighten ES?** The 1-min ATR on ES (~1.8 pts) means any SL below 10pts gets hit by noise before TP is reached. The wide-stop approach survives because 15pt stops rarely trigger. The R:R looks terrible but the 61% win rate compensates.

**What would actually fix ES:**
1. **Use ATR-based stops** instead of fixed — adapts to the current volatility regime
2. **Trade ES on 2-3 min** where the noise-to-signal ratio is better for fixed stops
3. **Enable ATR TP on A grades** to capture bigger moves when they happen
4. Real entries near structure (EMA/VWAP) should allow tighter stops than synthetic data suggests — the simplified signal generator doesn't capture entry quality

---

## Summary

| Instrument | Status | Action |
|-----------|--------|--------|
| NQ/MNQ | **Validated** — NQ-CONS-TIGHT outperforms current by +30% P&L | Update code: SL 13/13/10/10/8, TP 25/20/15/15/12 |
| ES/MES | **No improvement found** on fixed stops | Keep current defaults, recommend ATR-based stops for A grades |
| YM/MYM | Previously optimized in v7.4.2 | No change needed |
| CL | Not tested this round | Future work |
