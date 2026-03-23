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
| `MTF_RSI_Signals_v7.5.3.pine` | Latest Signals — RSI Speed, MTF Speed, Divergence detection |
| `MTF_RSI_Signals_v7.12.0_Backtested.pine` | Backtested variant — presets optimized via coordinate descent |
| `MTF_RSI_Management_v7.4.2.pine` | Trade management — sizing, TP/SL, runners, pyramids |
| `MTF_RSI_Core_Library.pine` | Shared calculation functions (prep for Pine v6 migration) |
| `EMA_HighLowMedian_Ribbon.pine` | Standalone EMA High/Low/Median ribbon overlay |
| `Session_Range_vs_ATR_Filter.pine` | Standalone session range vs ATR filter |
| `Backtest_Report_v7.12.0.md` | Optimization methodology and results for v7.12.0 presets |
| `YM_1MIN_SETTINGS.md` | Detailed YM/MYM 1-minute chart configuration guide |
| `MTF_RSI_Indicator_Guide.pdf` | User guide |
| `MTF_RSI_Library_Setup.pdf` | Library setup documentation |

---

## Grade Hierarchy

| Grade | Confluence | Trade Style |
|-------|-----------|-------------|
| **A+** | 3 HTF aligned + chart RSI + EMA + VWAP + DStoch + volume + ADX | Full size runner — partial at TP, trail the rest |
| **A** | 3 HTF aligned + chart RSI + EMA + DStoch + ADX (no VWAP/vol req) | Runner — partial at TP, trail the rest |
| **A-** | 2 HTF aligned + chart RSI + EMA + DStoch + macro filter | Standard — full exit at TP |
| **B+** | 2 HTF aligned + HMA trend + anti-chop filters | Scalp — in and out |
| **B** | 1 HTF aligned + HMA trend + anti-chop filters | Scalp — tight leash, 3-5 bars max |

---

## TP/SL Reference — 1-Minute Chart

All values are **points** unless noted. These are the values currently in the code (v7.5.3 Signals / v7.4.2 Management).

### 🛢️ CL (Crude Oil)

Tick = $0.01 · $10/tick · 100 ticks/point · `dollar_per_point = 10`

| Grade | SL | TP | Style | $/Contract Risk |
|-------|----|----|-------|-----------------|
| A+ | $0.40 (40t) | $0.20 (20t) | Runner | $400 |
| A | $0.40 (40t) | $0.20 (20t) | Runner | $400 |
| A- | $0.30 (30t) | $0.15 (15t) | Standard | $300 |
| B+ | $0.30 (30t) | $0.10 (10t) | Scalp | $300 |
| B | $0.30 (30t) | $0.08 (8t) | Scalp | $300 |

> **Note:** CL values are in dollar terms (e.g. $0.40 = 40 ticks). A+ and A should use ATR TP with runner trail — the fixed TPs are conservative first-target partials. B-grade R:R is tight; these are momentum scalps where win rate compensates. Session preset: 0900–1430 ET.

### 📈 NQ/MNQ (NASDAQ Futures)

Tick = 0.25 pts · 4 ticks/point · NQ: $5/tick ($20/pt) · MNQ: $0.50/tick ($2/pt) · Preset targets MNQ (`dollar_per_point = 2`)

| Grade | SL (pts) | SL (ticks) | TP (pts) | TP (ticks) | R:R | $/ct (MNQ) | $/ct (NQ) |
|-------|----------|-----------|----------|-----------|-----|------------|-----------|
| A+ | 20 | 80 | 20 | 80 | Runner | $40 | $400 |
| A | 20 | 80 | 20 | 80 | Runner | $40 | $400 |
| A- | 20 | 80 | 15 | 60 | 0.75:1 | $40 | $400 |
| B+ | 25 | 100 | 10 | 40 | 0.40:1 | $50 | $500 |
| B | 25 | 100 | 10 | 40 | 0.40:1 | $50 | $500 |

> **⚠️ Known issue:** NQ B-grade R:R needs work. B+ and B stops at 25 pts are wide for 1-min scalps and 10pt TP creates negative expectancy below ~65% win rate. Recommended 1-min overrides: B+ → 15pt SL / 15pt TP, B → 12pt SL / 12pt TP. The v7.12.0 backtested variant has improved values (B+ SL=20, B SL=25). Enable ATR TP for A grades — fixed 20pt leaves money on the table when NQ trends 50-100+ points.

### 📊 ES/MES (S&P 500)

Tick = 0.25 pts · 4 ticks/point · ES: $12.50/tick ($50/pt) · MES: $1.25/tick ($5/pt) · Preset targets MES (`dollar_per_point = 5`)

| Grade | SL (pts) | SL (ticks) | TP (pts) | TP (ticks) | R:R | $/ct (MES) | $/ct (ES) |
|-------|----------|-----------|----------|-----------|-----|------------|-----------|
| A+ | 15 | 60 | 6 | 24 | 0.40:1 | $75 | $750 |
| A | 15 | 60 | 6 | 24 | 0.40:1 | $75 | $750 |
| A- | 15 | 60 | 4 | 16 | 0.27:1 | $75 | $750 |
| B+ | 15 | 60 | 3 | 12 | 0.20:1 | $75 | $750 |
| B | 15 | 60 | 2 | 8 | 0.13:1 | $75 | $750 |

> **⚠️ ES has no preset-specific TP/SL — it falls through to DEFAULT values.** These R:R ratios are not viable for 1-min trading. ES needs a dedicated preset update. Recommended 1-min targets:
>
> | Grade | Suggested SL (pts) | Suggested TP (pts) | Target R:R |
> |-------|-------------------|-------------------|------------|
> | A+ | 6 | 12–15 | 2:1+ (runner) |
> | A | 6 | 10–12 | 1.7:1 (runner) |
> | A- | 5 | 8 | 1.6:1 |
> | B+ | 4 | 5–6 | 1.25:1+ |
> | B | 3 | 4 | 1.3:1 |
>
> The v7.12.0 backtested variant has ES-specific SL values (A+=8, A=6, A-=5, B+=8, B=12) that are closer to correct. TP optimization for ES is pending.

### 🔷 YM/MYM (Dow Futures)

Tick = 1 pt · 1 tick/point · YM: $5/tick · MYM: $0.50/tick · Preset targets MYM (`dollar_per_point = 0.50`)

| Grade | SL (pts/ticks) | TP (pts/ticks) | R:R | $/ct (MYM) | $/ct (YM) |
|-------|---------------|---------------|-----|------------|-----------|
| A+ | 15 | 35 | 2.3:1 | $7.50 | $75 |
| A | 15 | 25 | 1.7:1 | $7.50 | $75 |
| A- | 12 | 18 | 1.5:1 | $6.00 | $60 |
| B+ | 10 | 15 | 1.5:1 | $5.00 | $50 |
| B | 8 | 10 | 1.25:1 | $4.00 | $40 |

> YM TP/SL was optimized for 1-min in v7.4.2. All grades maintain minimum 1.25:1 R:R. A+/A grades should enable ATR TP (2.0x multiplier) and runner trail (34 EMA) — YM produces 40-60pt swings on aligned signals during the AM session. See `YM_1MIN_SETTINGS.md` for full configuration guide.

---

## Contract Quick Reference

| Instrument | Tick Size | Ticks/Point | $/Tick (Full) | $/Tick (Micro) | $/Point (Full) | $/Point (Micro) |
|-----------|-----------|-------------|---------------|----------------|----------------|-----------------|
| CL | $0.01 | 100 | $10.00 | — | $1,000 | — |
| NQ / MNQ | 0.25 pts | 4 | $5.00 | $0.50 | $20.00 | $2.00 |
| ES / MES | 0.25 pts | 4 | $12.50 | $1.25 | $50.00 | $5.00 |
| YM / MYM | 1 pt | 1 | $5.00 | $0.50 | $5.00 | $0.50 |

> **Important:** All presets default to micro contract dollar values. If you trade full-size contracts, you must override `$ Per Point Per Contract` in the Management indicator settings.

---

## MTF Stack (1-Minute Chart)

When running on a 1-minute chart, the multi-timeframe RSI checks these higher timeframes:

| Speed Setting | Fast | Mid | Slow | A+ Speed | Best For |
|--------------|------|-----|------|----------|----------|
| Conservative (default) | 5m | 15m | 60m | Slowest | Highest conviction, holds through pullbacks |
| Moderate | 5m | 15m | 30m | Medium | Session-based NY trading |
| Aggressive | 5m | 9m | 15m | Fastest | Strong trending sessions (use anti-chop filters) |

MTF Speed settings available in v7.5.3. Conservative is recommended for most 1-min trading.

---

## Version History

| Version | Indicator | Changes |
|---------|-----------|---------|
| v7.5.3 | Signals | RSI Speed presets, MTF Speed presets, RSI Divergence detection, YM SL fix |
| v7.12.0 | Signals | Backtested presets via coordinate descent (ES-specific SL, tuned NQ/CL/YM) |
| v7.4.2 | Management | YM SL/TP optimization, tick multiplier fix, dollar_per_point tooltip |
| v7.5.0 | Signals | Consolidation filter, anti-chop improvements (superseded by v7.5.3) |
| v7.4.1 | Management | Initial release — sizing dashboard, runners, pyramids |
