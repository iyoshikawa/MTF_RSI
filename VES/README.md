# Volume Exhaustion Strategy (VES) — User Guide
**Version:** 2.0.2
**Chart Type:** Renko Traditional (any base timeframe, down to 1s)  
**Instruments:** YM, MYM, NQ, MNQ, ES, MES, CL, MCL, HG  

---

## What This Strategy Does

VES is a **mean-reversion strategy** that detects when buying or selling pressure has exhausted at a swing extreme, then enters the reversal. It fires **VOL BOTTOM** (long) signals at swing lows and **VOL PEAK** (short) signals at swing highs. It manages two independent positions with take-profit, trailing stop, and breakeven stop exits.

---

## What Changed in v2.0.0

Major overhaul of the trend, squeeze, and chop detection systems:

- **EMA stack:** 9/15/34/233 → 21/34/50 (OHLC4). Slower, more deliberate — eliminates false unstacks on minor pullbacks.
- **Removed:** Old Soft Trend Filter (EMA stack + 233 macro gate). Replaced by HMA × EMA 21 trend warning gate.
- **HMA 20:** Source changed from `close` → `OHLC4` to match EMAs. Now plotted on chart. HMA crossing EMA 21 serves as the trend-warning gate.
- **NEW — Cross-Wave Volume Ratio:** Compares counter-wave volume to dominant-wave volume when 21/34/50 is stacked. Weak pop/dip (≤50%) = +1 confluence confirming dominant side controls. (Confluence, not gate.)
- **BB Squeeze:** Replaced static 2×TP threshold with adaptive ratio. Current width must be ≥60% of its own 50-bar moving average. Auto-calibrates across sessions and instruments.
- **NEW — Chop Detector:** Averages the last 5 wave lengths. If average is below 5 bricks, market is flip-heavy chop — all signals blocked until waves lengthen.
- **Confluence count:** 6 → 7 (added Cross-Wave Volume Ratio `W`).
- **Dashboard:** 18 → 19 rows. Added EMA Stack, Trend Gate, XWave Ratio, Chop status rows.
- **All new thresholds** exposed as inputs with conservative/aggressive guidance in tooltips.

---

## How It Works — The Signal Chain

A trade only fires when ALL required gates pass simultaneously:

```
Min Wave → Renko Flip → Volume Exhaustion → BB Outside + No Squeeze → Trend Gate → Session → ADR → Chop → Daily Limits
    ↓           ↓              ↓                    ↓                      ↓          ↓        ↓      ↓         ↓
 REQUIRED    REQUIRED       REQUIRED            REQUIRED               GATE(HMA)   REQUIRED  TOGGLE  GATE     REQUIRED
```

### Gate 1: Minimum Wave Length (Required, default 3 bricks)
The prior directional wave must be at least N bricks long. Prevents whipsaw chains on 1-2 brick chop flips. Set to 1 to disable.

### Gate 2: Renko Flip (Always Required)
A new brick in the opposite direction of the prior run.

### Gate 3: Volume Exhaustion (Required — 3 Methods + OFF)
**Wave Compare (Recommended for Renko):** Accumulates volume per directional wave. Fires on Wave Decline (ending wave had less volume — tiring out) or Wave Climax (blowoff — more volume but nobody left).

**Volume ROC:** Compares fast volume MA (5) to slow MA (20). Fires when volume is decelerating or below average at the flip. Better suited for time-based/Heiken Ashi charts.

**Z-Score:** Per-brick spike detection. Works poorly on Renko. Included for time-based charts.

**OFF:** No volume gate. Fires on every flip that meets other conditions.

### Gate 4: Bollinger Bands (Required — Includes Adaptive Squeeze)
1. **Band Position:** Price at/beyond BB edge within last 2 bars.
2. **Adaptive Squeeze Filter (v2.0.0):** BB width must be ≥ 60% of its own 50-bar moving average. Auto-calibrates — no manual adjustment needed between sessions or instruments. When squeezed, all signals blocked until bands expand.

### Gate 5: Trend Warning Gate — HMA × EMA 21 (v2.0.0, Toggleable)
HMA 20 (OHLC4) crossing through EMA 21 (OHLC4) signals a momentum shift. When HMA drops below EMA 21, longs are blocked (bearish momentum). When HMA rises above EMA 21, shorts are blocked (bullish momentum). Replaces the old 9/15/34 + 233 soft trend filter with a cleaner momentum-based read.

### Gate 6: Session Window (Required)
Default: 0930-1145, 1330-1600 ET. CL uses 0900-1430.

### Gate 7: ADR Exhaustion Filter (Toggleable, default ON)
80% of ADR consumed → new entries suppressed. 70% → trail tightens 30%.

### Gate 8: Chop Detector (v2.0.0, Toggleable)
Averages the length of the last 5 completed waves. If the average is below 5 bricks, the market is flipping back and forth without follow-through. All signals blocked until the average rises above the threshold.

### Gate 9: Daily Limits (Required)
DD limit ($3,000) and profit target (disabled by default). Flattens and blocks when hit.

---

## The 7 Confluences (v2.0.0)

Informational score on each signal label and dashboard. Does NOT gate entries.

| # | Code | Factor | Long | Short |
|---|------|--------|------|-------|
| 1 | F | Renko Flip | Bull brick after bear run | Bear brick after bull run |
| 2 | V | Volume Exhaustion | Wave decline/climax at low | Wave decline/climax at high |
| 3 | B | BB Outside Bands | At/below lower BB (2-bar) | At/above upper BB (2-bar) |
| 4 | S | Session Active | In trading window | In trading window |
| 5 | H | HMA Position | Price < HMA 20 | Price > HMA 20 |
| 6 | E | EMA Alignment | 21 > 34 > 50 stacked | 21 < 34 < 50 stacked |
| 7 | W | Cross-Wave Vol Ratio | Dip vol ≤50% of push vol (bull stack) | Pop vol ≤50% of push vol (bear stack) |

**Label example:** `VEB WvDecl 5b 5/7` = VOL BOTTOM, Wave Decline, 5-brick wave, 5/7 confluences.

---

## Position Management

- **Pyramiding = 2:** Max 2 independent positions, same direction
- **Exits per position:** TP (fixed), Trail (Fixed/ATR/Wider of Both), BE (after N bricks)
- **No fixed stop loss** — only TP, trail, BE, and flatten-reverse
- **Flatten-and-reverse:** Opposite signal closes all, enters new direction

---

## New v2.0.0 Settings — Defaults and Tuning Ranges

### Adaptive BB Squeeze Filter

| Setting | Default | Conservative | Aggressive |
|---------|---------|-------------|------------|
| Squeeze Ratio | 0.60 | 0.70 | 0.45 |
| Avg Lookback | 50 bars | 75 bars | 30 bars |

### Chop Detector

| Setting | Default | Conservative | Aggressive |
|---------|---------|-------------|------------|
| Waves to Average | 5 | 7 | 3 |
| Min Avg Length | 5.0 bricks | 6.0 | 4.0 |

### Cross-Wave Volume Ratio

| Setting | Default | Conservative | Aggressive |
|---------|---------|-------------|------------|
| Max Counter-Wave Ratio | 0.50 | 0.40 | 0.65 |

### EMA Stack

| Setting | Default | Conservative | Aggressive |
|---------|---------|-------------|------------|
| EMA Fast | 21 | 21 | 18 |
| EMA Slow | 50 | 50 | 44 |

### HMA Trend Gate

| Setting | Default | Conservative | Aggressive |
|---------|---------|-------------|------------|
| HMA Length | 20 | 24 | 16 |

---

## Protection Features Summary

| Feature | Prevents | Default |
|---------|----------|---------|
| Min Wave Length | Whipsaw chains on chop | 3 bricks |
| Adaptive BB Squeeze | Entries in compression | ON (60% ratio) |
| Chop Detector | Trading flip-heavy markets | ON (avg ≥5b) |
| ADR Exhaustion | Late entries, range spent | ON (80%) |
| ADR Trail Tighten | Giving back profits | ON (70%) |
| HMA × EMA 21 Gate | Momentum shift trades | ON |
| Breakeven Stop | Winners turning to losers | ON (BE+3 after 3b) |
| Daily DD Limit | Blowing up on bad day | $3,000 |
| Daily PT Target | Overtrading winning day | Disabled |
| Slippage | Unrealistic backtest P&L | 3 ticks |
| Safety Flatten | Broker position desync | Disabled |

---

## Dashboard Reference (19 rows)

| Row | Label | Shows |
|-----|-------|-------|
| 0 | VES v2.0.2 | Preset, chart type, debug |
| 1 | TP/TRAIL | TP + trail with mode (FIX/ATR/MAX) + ⚡ tighten |
| 2 | SIGNAL | scanning / VEB / VEP + chop status + avg wave len |
| 3 | Position | FLAT / LONG D:1/2 / SHORT D:2/2 + trail level |
| 4 | Session | Active / Outside |
| 5 | Daily P&L | P&L, trades W/L, DD/PT limits |
| 6 | ADR | % used, OK / TIGHTENED / EXHAUSTED |
| 7 | BB | Position + squeeze status + width vs threshold |
| 8 | Trend Gate | HMA vs EMA 21 position + HMA slope |
| 9 | EMA Stack | 21/34/50 alignment (BULL / BEAR / MIXED) |
| 10 | XWave Ratio | Cross-wave volume comparison + ratio % |
| 11 | Confluence | X/7 with FVBSHEW breakdown |
| 12 | Win Rate | Win % + closed trades |
| 13 | Net P&L | All-time P&L + max DD |
| 14 | Prof Factor | PF + W/L count |
| 15 | Expectancy | $/trade |
| 16 | Avg W/L | Avg win $ / avg loss $ |
| 17 | Vol [METHOD] | Exhaustion status + method stats |
| 18 | 🔧 GATES | F V BB CH TR SS ADR pass/fail |

---

## Instrument Presets

| Preset | TP | Trail | Session | BB Mult | BB TF | DD |
|--------|-----|-------|---------|---------|-------|----|
| 🔷 YM | 40pt | 40pt | 0930-1145,1330-1600 | 1.2 | 3m | $3K |
| 📈 NQ | 40pt | 40pt | 0930-1145,1330-1600 | 1.2 | 3m | $3K |
| 📊 ES | 8pt | 8pt | 0930-1145,1330-1600 | 1.2 | 3m | $3K |
| 🛢️ CL | 0.20 | 0.20 | 0900-1430 | 1.4 | 5m | $2K |
| 🟤 HG | 0.005 | 0.005 | 0930-1145,1330-1600 | 1.2 | 3m | $3K |
| ⚙️ CUSTOM | Manual | Manual | Manual | Manual | Manual | Manual |

All tooltips show `(Default: X)` and conservative/aggressive ranges for recovery and tuning.

---

## QuantLynk Webhook

1. Enable in QUANTLYNK COMPATIBILITY settings, enter User ID + Alert ID
2. TradingView alert message: `{{strategy.order.alert_message}}`
3. Webhook URL: Your QuantLynk endpoint (see QuantLynk dashboard)
4. **Test on SIM first** — flatten is account-wide

---

## ATS Safety Flatten (v1.16.0)

Fires a redundant flatten webhook whenever the strategy transitions to fully flat. Catches position desync from manual flattens, missed webhooks, or broker-side auto-flattens. Harmless no-op if broker is already flat.

1. Enable in ATS COMPATIBILITY settings, enter User ID + Spam Key
2. Create a **second** TradingView alert on this strategy:
   - Condition: "Any alert() function call"
   - Webhook URL: Your ATS webhook endpoint
   - Message: leave default
3. Also fires a redundant QL flatten if QuantLynk is enabled

---

## Version History

| Ver | Feature |
|-----|---------|
| 1.0-1.4 | Core engine, BB/HMA gates, debug tools |
| 1.5 | 3 volume methods with selector |
| 1.6-1.8 | Daily P&L, BE stop, profit target |
| 1.9 | QuantLynk webhooks |
| 1.10-1.12 | Confluence counter, labels, preset system, tooltips |
| 1.13 | ATR adaptive trail + ADR exhaustion filter |
| 1.14 | Min wave length filter |
| 1.15 | BB squeeze filter |
| 1.16 | Slippage 3 ticks, ATS safety flatten, input layout |
| **2.0** | **EMA 21/34/50 stack, HMA×EMA21 trend gate, cross-wave volume ratio, adaptive BB squeeze, chop detector, 7 confluences** |
| 2.0.1 | Bug fixes: na protection, multi-close tracking, confluence in entry comments, squeeze release alert, HMA color input |
| 2.0.2 | Code refinements: VER constant, explicit YM/NQ presets, entry dedup, tooltip extraction, BB TF label fix, dead code removal |

---

## Quick Start

1. Add to Renko Traditional 8-tick chart (YM, 30s base)
2. Select instrument preset
3. Detection Method → "Wave Compare"
4. Min Wave Length → 3
5. BB Squeeze → ON (adaptive — no tuning needed)
6. Chop Detector → ON
7. Trend Gate → ON
8. Debug OFF, all gates ON
9. Check GATES row if no signals appear
10. Wire QuantLynk to SIM for real fills
