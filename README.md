# MTF RSI Signal Suite

Multi-timeframe RSI grading system for futures day trading. Grades setups from A+ to B based on confluence across timeframes, trend, momentum, volume, and volatility filters.

**Two indicators, one system:**
- **Signals** (`MTF_RSI_Signals`) — generates graded entry signals with visual overlays
- **Management** (`MTF_RSI_Management`) — trade management: sizing, TP/SL levels, runner trails, pyramids

Settings must match between both indicators (preset, RSI, stochastic, ADX, session config).

---

## Repository Files

| File | Description |
|------|-------------|
| `MTF_RSI_Signals_v7.7.0.pine` | Signals — grades, TP/SL, volume analysis, divergence, Renko support |
| `MTF_RSI_Management_v7.5.0.pine` | Management — trade lifecycle, sizing, runners, pyramids |
| `EMA_HighLowMedian_Ribbon.pine` | Standalone EMA High/Low/Median ribbon overlay |
| `Session_Range_vs_ATR_Filter.pine` | Standalone session range vs ATR filter |
| `Indicator_Guide.md` | Complete reference — every signal, marker, dashboard element, and setting |
| `Alert_Setup_Guide.md` | TradingView alert configuration + full trade examples |
| `Backtest_Report_v7.12.0.md` | Optimization methodology and results for backtested presets |
| `Backtest_Report_ES_NQ_TPSL.md` | ES & NQ TP/SL optimization — 4 param sets × 8 seeds |
| `YM_1MIN_SETTINGS.md` | Detailed YM/MYM 1-minute chart configuration guide |

---

## Grade Hierarchy

| Grade | Confluence Required | Trade Style | Sizing |
|-------|-----------|-------------|--------|
| **A+** | 3 HTF aligned + chart RSI + EMA + VWAP + DStoch + volume + ADX | Runner — partial at TP, trail the rest | 100% max position |
| **A** | 3 HTF aligned + chart RSI + EMA + DStoch + ADX | Runner — partial at TP, trail the rest | 100% max position |
| **A-** | 2 HTF aligned + chart RSI + EMA + DStoch + macro filter | Standard — full exit at TP | 75% max position |
| **B+** | 2 HTF aligned + HMA trend + anti-chop filters | Scalp — in and out | 50% max position |
| **B** | 1 HTF aligned + HMA trend + anti-chop filters | Scalp — tight leash, 3-5 bars max | 25% max position |

---

## TP/SL Reference — 1-Minute Chart

All values in the code as of **v7.7.0 Signals / v7.5.0 Management**.

### At a Glance — All Instruments

| Instrument | Grade | SL | TP | R:R | Validated | Status |
|-----------|-------|-----|-----|-----|-----------|--------|
| **NQ/MNQ** | A+ | 13 pts | 25 pts | 1.92:1 | 8/8 seeds | ✅ Backtested |
| | A | 13 pts | 20 pts | 1.54:1 | | |
| | A- | 10 pts | 15 pts | 1.50:1 | | |
| | B+ | 10 pts | 15 pts | 1.50:1 | | |
| | B | 8 pts | 12 pts | 1.50:1 | | |
| **YM/MYM** | A+ | 15 pts | 35 pts | 2.33:1 | Manual | ✅ Optimized v7.4.2 |
| | A | 15 pts | 25 pts | 1.67:1 | | |
| | A- | 12 pts | 18 pts | 1.50:1 | | |
| | B+ | 10 pts | 15 pts | 1.50:1 | | |
| | B | 8 pts | 10 pts | 1.25:1 | | |
| **ES/MES** | A+ | 15 pts | 6 pts | 0.40:1 | 8/8 seeds | ⚠️ High-WR only |
| | A | 15 pts | 6 pts | 0.40:1 | | |
| | A- | 15 pts | 4 pts | 0.27:1 | | |
| | B+ | 15 pts | 3 pts | 0.20:1 | | |
| | B | 15 pts | 2 pts | 0.13:1 | | |
| **CL** | A+ | $0.40 | $0.20 | 0.50:1 | — | ⚠️ Not yet backtested |
| | A | $0.40 | $0.20 | 0.50:1 | | |
| | A- | $0.30 | $0.15 | 0.50:1 | | |
| | B+ | $0.30 | $0.10 | 0.33:1 | | |
| | B | $0.30 | $0.08 | 0.27:1 | | |

---

### 📈 NQ/MNQ (NASDAQ Futures) — ✅ BACKTESTED

Tick = 0.25 pts · 4 ticks/point · NQ: $5/tick ($20/pt) · MNQ: $0.50/tick ($2/pt)
Preset `dollar_per_point = 2` (MNQ). Override to `20` for full NQ.

| Grade | SL (pts) | SL (ticks) | TP (pts) | TP (ticks) | R:R | $/ct Risk (MNQ) | $/ct Risk (NQ) |
|-------|----------|-----------|----------|-----------|-----|-----------------|----------------|
| A+ | 13 | 52 | 25 | 100 | 1.92:1 | $26 | $260 |
| A | 13 | 52 | 20 | 80 | 1.54:1 | $26 | $260 |
| A- | 10 | 40 | 15 | 60 | 1.50:1 | $20 | $200 |
| B+ | 10 | 40 | 15 | 60 | 1.50:1 | $20 | $200 |
| B | 8 | 32 | 12 | 48 | 1.50:1 | $16 | $160 |

#### What Changed (v7.4.3)

| Grade | Old SL | New SL | Old TP | New TP | Old R:R | New R:R |
|-------|--------|--------|--------|--------|---------|---------|
| A+ | 20 | **13** | 20 | **25** | 1.00:1 | **1.92:1** |
| A | 20 | **13** | 20 | 20 | 1.00:1 | **1.54:1** |
| A- | 20 | **10** | 15 | 15 | 0.75:1 | **1.50:1** |
| B+ | 25 | **10** | 10 | **15** | 0.40:1 | **1.50:1** |
| B | 25 | **8** | 10 | **12** | 0.40:1 | **1.50:1** |

#### Backtest Validation

Method: NQ-CONS-TIGHT parameter set, 20 days synthetic 1-min OHLCV per seed, 8 seeds total (42, 137, 256, 512, 777, 1024, 3141, 9999).

| Metric | Result |
|--------|--------|
| Seeds passed | **8/8** |
| Avg win rate | 45.1% |
| Avg profit factor | 1.30 |
| Avg realized R:R | 1.58:1 |
| Avg P&L (MNQ) | +1,264.9 pts |
| P&L std dev | 430.7 pts |
| All seeds positive | Yes |

#### Runner & ATR TP Guidance

- **A+/A grades:** Enable ATR TP (2.0x multiplier). Take partial at fixed TP, trail runner on 34 EMA. NQ regularly produces 50-100+ pt moves when 3 timeframes align. The 25/20pt fixed TPs are first targets, not final exits.
- **A- grades:** Full exit at 15pt TP. Not enough conviction for a runner — 2TF alignment can flip faster.
- **B+/B grades:** Scalp only. In and out at the fixed TP. If it doesn't hit in 5-8 bars, scratch it.

---

### 🔷 YM/MYM (Dow Futures) — ✅ OPTIMIZED

Tick = 1 pt · 1 tick/point · YM: $5/tick · MYM: $0.50/tick
Preset `dollar_per_point = 0.50` (MYM). Override to `5` for full YM.

| Grade | SL (pts/ticks) | TP (pts/ticks) | R:R | $/ct Risk (MYM) | $/ct Risk (YM) |
|-------|---------------|---------------|-----|-----------------|----------------|
| A+ | 15 | 35 | 2.33:1 | $7.50 | $75 |
| A | 15 | 25 | 1.67:1 | $7.50 | $75 |
| A- | 12 | 18 | 1.50:1 | $6.00 | $60 |
| B+ | 10 | 15 | 1.50:1 | $5.00 | $50 |
| B | 8 | 10 | 1.25:1 | $4.00 | $40 |

#### What Changed (v7.4.2)

| Grade | Old SL | New SL | Old TP | New TP | Old R:R | New R:R |
|-------|--------|--------|--------|--------|---------|---------|
| A+ | 20 | **15** | 20 | **35** | 1.00:1 | **2.33:1** |
| A | 20 | **15** | 20 | **25** | 1.00:1 | **1.67:1** |
| A- | 20 | **12** | 15 | **18** | 0.75:1 | **1.50:1** |
| B+ | 15 | **10** | 10 | **15** | 0.67:1 | **1.50:1** |
| B | 15 | **8** | 10 | **10** | 0.67:1 | **1.25:1** |

#### Runner & ATR TP Guidance

- **A+/A grades:** Enable ATR TP (2.0x multiplier) and runner trail on **34 EMA**. YM produces 40-60pt swings during the AM session on aligned signals. The 21 HMA is too tight for 1-min YM — 34 EMA gives enough room to ride pullbacks.
- **A- grades:** Full exit at 18pt TP. Standard trade, not a runner.
- **B+/B grades:** Pure scalp. Get your 10-15 points and move on.

#### Additional 1-Min Settings

YM on the 1-min needs tighter anti-chop settings than the preset defaults. See `YM_1MIN_SETTINGS.md` for the full configuration guide, including cooldown bars (8-10), persist bars (3), ADX floor (18-20), consolidation filter, and session trim.

#### Tick Multiplier Fix

YM is 1 tick = 1 point. The code originally used `stop_dist * 4` for the tick display (correct for ES/NQ, wrong for YM). Fixed in v7.4.2 to use `stop_dist * 1` for the YM preset.

---

### 📊 ES/MES (S&P 500) — ⚠️ BACKTESTED, KEPT DEFAULTS

Tick = 0.25 pts · 4 ticks/point · ES: $12.50/tick ($50/pt) · MES: $1.25/tick ($5/pt)
Preset `dollar_per_point = 5` (MES). Override to `50` for full ES.

| Grade | SL (pts) | SL (ticks) | TP (pts) | TP (ticks) | R:R | $/ct Risk (MES) | $/ct Risk (ES) |
|-------|----------|-----------|----------|-----------|-----|-----------------|----------------|
| A+ | 15 | 60 | 6 | 24 | 0.40:1 | $75 | $750 |
| A | 15 | 60 | 6 | 24 | 0.40:1 | $75 | $750 |
| A- | 15 | 60 | 4 | 16 | 0.27:1 | $75 | $750 |
| B+ | 15 | 60 | 3 | 12 | 0.20:1 | $75 | $750 |
| B | 15 | 60 | 2 | 8 | 0.13:1 | $75 | $750 |

#### Why These R:R Ratios Survive

The R:R looks terrible on paper. It was backtested specifically because it looked wrong. Here's what the data showed:

**Tested 4 tighter alternatives across 8 seeds:**

| Param Set | SL Range | TP Range | Avg WR | Avg PF | Seeds Passed |
|-----------|----------|----------|--------|--------|-------------|
| **Current (15pt flat)** | 15/15/15/15/15 | 6/6/4/3/2 | **61.1%** | **1.24** | **8/8** |
| Conservative (6-3pt SL) | 6/6/5/4/3 | 10/10/7/5/4 | 39.0% | 0.71 | 0/4 |
| Moderate (7-3pt SL) | 7/7/5/4/3 | 14/12/8/6/4 | 39.7% | 0.68 | 0/4 |
| Aggressive (8-4pt SL) | 8/8/6/5/4 | 16/14/10/7/5 | 41.4% | 0.67 | 0/4 |
| Bridge-A (10-7pt SL) | 10/10/8/8/7 | 10/8/6/5/4 | 50.6% | 0.92 | 2/4 |
| Bridge-B (12-7pt SL) | 12/12/10/8/7 | 14/12/8/6/4 | 47.6% | 0.87 | 1/4 |

The issue: ES 1-min ATR is ~1.8 pts. Any stop below 10 pts gets eaten by noise. The 15pt flat stop rarely triggers (~3-7% of trades), so the strategy wins on frequency — lots of small TP hits, very rare SL hits. It's not a good R:R strategy; it's a high-probability strategy.

#### How to Actually Improve ES

The fix isn't tighter fixed stops. It's:

1. **Enable ATR-based TP on A grades** — the fixed 6pt TP leaves money on the table on trending days. ATR TP at 2.0x captures more of those 10-20pt moves.
2. **Trade ES on 2-3 minute charts** where the noise-to-signal ratio is better for structured SL/TP.
3. **Use real structure for stops** — entries near EMA/VWAP should allow tighter stops than 15pts, but the synthetic backtest can't capture entry quality.
4. **ES preset needs dedicated TP/SL values** — it currently falls through to DEFAULT. Future optimization should test ATR-scaled stops rather than fixed-point.

See `Backtest_Report_ES_NQ_TPSL.md` for full results.

---

### 🛢️ CL (Crude Oil) — ⚠️ NOT YET BACKTESTED

Tick = $0.01 · $10/tick · 100 ticks/point · `dollar_per_point = 10`
Session preset: 0900–1430 ET.

| Grade | SL | SL (ticks) | TP | TP (ticks) | R:R | $/Contract Risk |
|-------|-----|-----------|-----|-----------|-----|-----------------|
| A+ | $0.40 | 40 | $0.20 | 20 | 0.50:1 | $400 |
| A | $0.40 | 40 | $0.20 | 20 | 0.50:1 | $400 |
| A- | $0.30 | 30 | $0.15 | 15 | 0.50:1 | $300 |
| B+ | $0.30 | 30 | $0.10 | 10 | 0.33:1 | $300 |
| B | $0.30 | 30 | $0.08 | 8 | 0.27:1 | $300 |

#### Status

CL values are from the original presets and have not been run through the synthetic backtester yet. The R:R ratios on B grades are concerning (same issue NQ and YM had before optimization). CL has unique characteristics — wider bid-ask spread, inventory report spikes, pit session dynamics — that may require a different approach than the equity index futures.

#### Runner & ATR TP Guidance

- **A+/A grades:** These are runner candidates. The fixed $0.20 TP is a first partial target. Use ATR TP and trail on the 21 EMA (CL preset uses 21 EMA, not 34).
- **B grades:** Momentum scalps. CL moves fast — 10-tick scalps work when you have the direction right, but 30-tick stops may be too generous. Needs backtesting to validate.

---

## Trade Management Quick Reference

### When to Use ATR TP vs Fixed TP

| Scenario | Use |
|----------|-----|
| A+/A grades in trending session | **ATR TP** — let the runner ride |
| A+/A grades in choppy/range session | **Fixed TP** — take what you can get |
| A- grades | **Fixed TP** — always full exit |
| B+/B grades | **Fixed TP** — always full exit, never a runner |
| ADR > 70% used | **Fixed TP or tighter** — range is exhausting |
| ADX strong and rising | **ATR TP** — momentum supports wider targets |
| ADX falling or below 20 | **Fixed TP** — trend is weakening |

### Runner Trail Settings by Instrument

| Instrument | Recommended Trail | Why |
|-----------|------------------|-----|
| NQ/MNQ | 34 EMA | Standard — NQ trends smoothly, 34 EMA gives room |
| YM/MYM | 34 EMA | YM is choppier — 21 HMA too tight, 34 EMA holds better |
| ES/MES | 34 EMA or 21 HMA | Either works — 21 HMA if you want tighter trail |
| CL | 21 EMA (preset default) | CL moves fast — 21 EMA tracks momentum well |

### Position Sizing by Grade

Based on the Max Pain $ setting in Management indicator. The sizing dashboard auto-calculates contracts.

| Grade | % of Max Position | Risk Profile |
|-------|------------------|--------------|
| A+ | 100% | Full conviction — everything aligned |
| A | 100% | Strong setup — same size as A+ |
| A- | 75% | Missing slow TF — reduce exposure |
| B+ | 50% | Scalp only — half size |
| B | 25% | Quick scalp — quarter size |

---

## Contract Quick Reference

| Instrument | Tick Size | Ticks/Point | $/Tick (Full) | $/Tick (Micro) | $/Point (Full) | $/Point (Micro) |
|-----------|-----------|-------------|---------------|----------------|----------------|-----------------|
| CL | $0.01 | 100 | $10.00 | — | $1,000 | — |
| NQ / MNQ | 0.25 pts | 4 | $5.00 | $0.50 | $20.00 | $2.00 |
| ES / MES | 0.25 pts | 4 | $12.50 | $1.25 | $50.00 | $5.00 |
| YM / MYM | 1 pt | 1 | $5.00 | $0.50 | $5.00 | $0.50 |

> **Important:** All presets default to micro contract dollar values. If you trade full-size contracts, you must override `$ Per Point Per Contract` in the Management indicator settings. Getting this wrong means your sizing dashboard will calculate 10x too many contracts.

---

## MTF Stack (1-Minute Chart)

When running on a 1-minute chart, the multi-timeframe RSI checks these higher timeframes:

| Speed Setting | Fast | Mid | Slow | A+ Speed | Best For |
|--------------|------|-----|------|----------|----------|
| Conservative (default) | 5m | 15m | 60m | Slowest | Highest conviction, holds through pullbacks |
| Moderate | 5m | 15m | 30m | Medium | Session-based NY trading |
| Aggressive | 5m | 9m | 15m | Fastest | Strong trending sessions (use anti-chop filters) |

MTF Speed settings available in v7.7.0. Conservative is recommended for most 1-min trading.

---

## Optimization Status

| Instrument | SL/TP Status | Method | Version |
|-----------|-------------|--------|---------|
| NQ/MNQ | ✅ Backtested & implemented | Synthetic 1-min, 4 param sets × 8 seeds | v7.4.3 |
| YM/MYM | ✅ Manually optimized & implemented | Experience-based R:R calibration | v7.4.2 |
| ES/MES | ⚠️ Backtested, defaults kept | Synthetic 1-min, 7 param sets × 8 seeds — tighter SL all failed | v7.4.3 |
| CL | ❌ Not yet optimized | Original preset values, needs backtesting | — |

**Next up:** CL backtesting, ES ATR-based stop testing.

---

## Version History

| Version | Indicator | Changes |
|---------|-----------|---------|
| v7.7.0 | Signals | Volume analysis, TP/SL levels, split alerts (long/short + TP/SL hit), Renko support, plot optimization, all colors configurable |
| v7.5.0 | Management | Trade lifecycle (TP1/TP2/breakeven/P&L), contract size selector (Micro/Mini), per-grade sizing, TP/SL MODE rows |
| v7.5.3 | Signals | RSI Speed presets, MTF Speed presets, RSI Divergence detection, YM SL fix, NQ SL fix |
| v7.12.0 | Signals | Backtested presets via coordinate descent (ES-specific SL, tuned NQ/CL/YM) |
| v7.4.3 | Management | NQ SL/TP backtested optimization (CONS-TIGHT), NQ TP updated, version bump |
| v7.4.2 | Management | YM SL/TP optimization, tick multiplier fix, dollar_per_point tooltip |
| v7.5.0 | Signals | Consolidation filter, anti-chop improvements (superseded by v7.5.3) |
| v7.4.1 | Management | Initial release — sizing dashboard, runners, pyramids |
