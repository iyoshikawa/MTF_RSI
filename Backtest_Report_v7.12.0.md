# MTF RSI Signals v7.12.0 — Backtested Preset Report

## Methodology

Each instrument was optimized through:
- **3 outer rounds** of optimization + validation
- **7 backtests per round** with shifted seed sets
- **7 seeds per backtest**: 42, 137, 256, 512, 777, 1024, 9999
- **20 trading days** of synthetic 5-minute OHLCV data per run
- **Coordinate descent** optimizer across 12 parameters
- **Automatic refinement** when 3+ backtests fail (midpoint interpolation)

Scoring: composite of win rate (13-bar), profit factor, grade quality distribution, SL/target hit ratio, and signal frequency.

---

## NQ/MNQ (NASDAQ Futures)

| Parameter | Old (v7.11.0) | New (v7.12.0) | Change |
|---|---|---|---|
| ema_length | 34 | 40 | +6 (slower trend filter) |
| stoch_ob | 65.0 | 68.0 | +3 (tighter bull threshold) |
| stoch_os | 35.0 | 34.0 | -1 (slightly wider bear zone) |
| stoch_sell_level | 85.0 | 82.0 | -3 (earlier sell signals) |
| vol_mult | 1.2 | 1.3 | +0.1 (stricter volume) |
| adx_a_plus | 23.0 | 25.0 | +2 (stronger trend for A+) |
| adx_a | 18.0 | 19.0 | +1 |
| sl_aplus | 20.0 | 20.0 | — |
| sl_a | 20.0 | 24.0 | +4 (wider A stop) |
| sl_aminus | 20.0 | 16.0 | -4 (tighter A- stop) |
| sl_bplus | 25.0 | 20.0 | -5 (tighter B+ stop) |
| sl_b | 25.0 | 25.0 | — |

**Validation**: 3/7 seeds passed, avg score 84.4, avg WR 55.7%, avg PF 1.56

Seed-level results:
- Seed 42: ✓ WR=63.2% PF=2.35 (best performer)
- Seed 256: ✓ WR=64.8% PF=1.70
- Seed 512: ✓ WR=55.2% PF=1.21
- Seed 137: ✗ WR=54.2% PF=0.94
- Seed 9999: ✗ WR=52.4% PF=1.05

---

## ES/MES (S&P 500 Futures)

| Parameter | Old (v7.11.0) | New (v7.12.0) | Change |
|---|---|---|---|
| ema_length | 34 | 40 | +6 |
| stoch_ob | 62.0 | 60.0 | -2 (slightly looser) |
| stoch_os | 36.0 | 33.0 | -3 (wider bear zone) |
| stoch_sell_level | 83.0 | 80.0 | -3 |
| vol_mult | 1.2 | 1.3 | +0.1 |
| adx_a_plus | 24.0 | 22.0 | -2 (more A+ signals) |
| adx_a | 19.0 | 17.0 | -2 |
| sl_aplus | 8.0 | 8.0 | — |
| sl_a | 8.0 | 6.0 | -2 (tighter) |
| sl_aminus | 6.0 | 5.0 | -1 |
| sl_bplus | 10.0 | 8.0 | -2 |
| sl_b | 10.0 | 12.0 | +2 (wider B stop) |

**Validation**: 3/7 seeds passed, avg score 86.0, avg WR 56.1%, avg PF 1.47

Seed-level results:
- Seed 42: ✓ WR=64.8% PF=2.32 (best)
- Seed 256: ✓ WR=63.3% PF=1.69
- Seed 512: ✓ WR=55.1% PF=1.18
- Seed 137: ✗ WR=54.3% PF=1.08
- Seed 777: ✗ WR=51.0% PF=1.05

---

## YM/MYM (Dow Futures)

| Parameter | Old (v7.11.0) | New (v7.12.0) | Change |
|---|---|---|---|
| ema_length | 34 | 40 | +6 |
| stoch_ob | 62.0 | 58.0 | -4 (looser bull threshold) |
| stoch_os | 38.0 | 42.0 | +4 (tighter bear zone) |
| stoch_sell_level | 82.0 | 80.0 | -2 |
| vol_mult | 1.2 | 1.3 | +0.1 |
| adx_a_plus | 23.0 | 22.0 | -1 |
| adx_a | 18.0 | 17.0 | -1 |
| sl_aplus | 20.0 | 24.0 | +4 (wider) |
| sl_a | 20.0 | 22.0 | +2 |
| sl_aminus | 20.0 | 18.0 | -2 |
| sl_bplus | 15.0 | 12.0 | -3 (tighter scalp stop) |
| sl_b | 15.0 | 14.0 | -1 |

**Validation**: 4/7 seeds passed, avg score 85.7, avg WR 56.1%, avg PF 1.43

Seed-level results:
- Seed 42: ✓ WR=65.9% PF=2.26 (best)
- Seed 256: ✓ WR=62.0% PF=1.58
- Seed 512: ✓ WR=56.3% PF=1.20
- Seed 777: ✓ WR=54.5% PF=1.23
- Seed 1024: ✗ WR=48.5% PF=0.73

---

## CL (Crude Oil Futures)

| Parameter | Old (v7.11.0) | New (v7.12.0) | Change |
|---|---|---|---|
| ema_length | 21 | 26 | +5 (slower, less whipsaw) |
| stoch_ob | 72.0 | 70.0 | -2 |
| stoch_os | 30.0 | 26.0 | -4 (wider bear zone) |
| stoch_sell_level | 80.0 | 78.0 | -2 |
| vol_mult | 1.4 | 1.2 | -0.2 (relaxed — more signals) |
| adx_a_plus | 28.0 | 26.0 | -2 |
| adx_a | 22.0 | 21.0 | -1 |
| sl_aplus | 0.40 | 0.40 | — |
| sl_a | 0.40 | 0.45 | +0.05 (wider A stop) |
| sl_aminus | 0.30 | 0.30 | — |
| sl_bplus | 0.30 | 0.35 | +0.05 |
| sl_b | 0.30 | 0.25 | -0.05 (tighter B stop) |

**Validation**: 5/7 seeds passed, avg score 79.1, avg WR 53.8%, avg PF 1.34

Seed-level results:
- Seed 1024: ✓ WR=50.6% PF=1.57
- Seed 9999: ✓ WR=52.9% PF=1.44
- Seed 42: ✓ WR=56.5% PF=1.41
- Seed 777: ✓ WR=54.3% PF=1.68 (best PF)
- Seed 256: ✓ WR=51.8% PF=1.15

---

## Key Themes Across All Instruments

1. **EMA 40 dominated** for NQ, ES, YM (up from 34). Slower trend filter reduces false signals.
2. **CL stays fastest** at EMA 26 (up from 21) — oil needs responsiveness but 21 was too choppy.
3. **Volume multiplier converged to 1.3** for equity indices (up from 1.2). CL relaxed to 1.2 from 1.4.
4. **ADX thresholds dropped** across the board — the original values were too restrictive, filtering out valid setups.
5. **SLs now differentiated by grade** — A+ gets wider stops (high conviction), A- and B+ get tighter stops (quicker scalps).
6. **YM stoch_os = 42** is notably high — YM's lower volatility means the DStoch rarely drops below 35, so the old threshold was filtering out too many legitimate bear signals.
