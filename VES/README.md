# Volume Exhaustion Strategy (VES) — User Guide
**Version:** 2.0.7
**Chart Type:** Renko Traditional (any base timeframe, down to 1s)  
**Instruments:** YM, MYM, NQ, MNQ, ES, MES, CL, MCL, HG  

---

## What This Strategy Does

VES is a **mean-reversion strategy** that detects when buying or selling pressure has exhausted at a swing extreme, then enters the reversal. It fires **VOL BOTTOM** (long) signals at swing lows and **VOL PEAK** (short) signals at swing highs. It manages two independent positions with take-profit, trailing stop, and breakeven stop exits.

---

## What Changed in v2.0.7

Enhanced Wave Compare diagnostics and fine-tuning controls:

- **NEW: Wave Compare Debug Labels** — When Debug Mode is ON, visual labels appear on every flip bar showing raw Wave Compare triggers BEFORE gates filter them. Labels display:
  - Trigger type: EXH (exhaustion), CLX (climax), or NONE
  - Wave volumes (prev vs prev2) for the relevant direction
  - Pass/fail status for each detection type
  - Detailed tooltip with all thresholds and calculations
  - Helps diagnose exactly why Wave Compare is or isn't firing

- **NEW Wave Compare Inputs** (📊 Volume Exhaustion Detection group):
  - `Wave: Enable Exhaustion Detection` (ON) — Toggle declining-volume detection
  - `Wave: Enable Climax Detection` (ON) — Toggle blowoff/spike detection
  - `Wave: Climax Multiplier` (1.2) — Min multiplier for climax vs prior wave. 1.0 = any increase, 1.5 = 50% higher, 2.0 = double
  - `Wave: Min Wave Volume` (0) — Filter out low-volume noise waves. For YM/NQ, try 500–2000 to exclude thin waves

- **NEW Debug Input:** `🔧 Show Wave Compare Labels` (ON) — Toggle Wave Compare labels independently. Only visible when Debug Mode is also ON.

- **Dashboard Vol WAVE row enhanced** — Now shows [EC] indicator for which triggers are enabled (E=Exhaustion, C=Climax), plus trigger type (EXH/CLX) when firing.

---

## What Changed in v2.0.6

- **BB demoted from gate to confluence** — BB band-edge no longer blocks signals. Still computed and contributes +1 to confluence score. Removes over-filtering in trending markets where price doesn't return to bands.
- **FIX: flip_to_bull/flip_to_bear now always required** — Previously with V:OFF, signals fired on every bar meeting other conditions. Now flip is enforced regardless of volume method.

---

## What Changed in v2.0.5

- **FIX: Hard stop loss added** — `strategy.exit()` calls were missing `loss=` parameter entirely. If price went straight against entry, there was no stop. Added `sl_pts` input (default 40pt) and `loss=sl_ticks` to all four exit calls.

---

## What Changed in v2.0.4

- **FIX: BB gate lookback** — Upgraded from fixed 2-bar to per-wave boolean. Tracks whether the prior wave EVER touched the BB edge during its entire duration.
- **FIX: Wave Compare warmup** — First wave in a direction now passes automatically. Prevents permanent block after gaps or one-sided sessions.
- **FIX: Chop gate bootstrap** — Blocks entries until N waves measured. Previously first 4 waves had zero chop protection.

---

## What Changed in v2.0.3

Targeted fixes and a new confluence requirements system:

- **ADR repaint fix:** `daily_high`/`daily_low` now use `[1]` offset + `lookahead_on`. Prevents the daily bar's intraday updates from retroactively changing the ADR calculation on historical bars.
- **Trail Activation Bricks:** New input (default 1, range 1–5). Controls how many Renko bricks price must move in your favor before the trailing stop begins tracking. Replaces the previous hardcoded `trail_points=1`, which started the trail immediately regardless of chart type or brick size. At default (1 brick), behavior is nearly identical to before — but now you can set 2–3 bricks for more room before the trail engages.
- **Confluence Requirements group (🎯):** New input group with six toggles:
  - `req_V` / `req_B` / `req_S` — ON by default (Volume, BB, Session — the core gates)
  - `req_H` / `req_E` / `req_W` — OFF by default (Hpx, EMA Stack, Cross-Wave Ratio)
  - Each toggle makes its confluence factor **required** for a signal to fire, independently of the debug section. With defaults unchanged, backtest results are identical to v2.0.2. Flip `req_H`, `req_E`, or `req_W` to ON to promote those factors from informational → required.
- **HMA tooltip clarified:** Documents both roles — (1) Hpx Confluence: price position vs HMA for mean-reversion scoring; (2) HMA Gate: HMA vs EMA 21 cross for momentum blocking. Previously only the gate role was described.
- **Labeling consistency:** "Trend Gate" → **"HMA Gate"** in dashboard row 8. Confluence detail code `H` → `Hpx` throughout (labels, tooltips, debug row).
- **Debug row 18 expanded:** Now shows `Hpx:` / `E:` / `W:` gate status alongside existing F/V/BB/TR/SS/CH/ADR columns.

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

A trade only fires when ALL active gates pass simultaneously. The `req_*` toggles in **🎯 Confluence Requirements** control which factors are required gates vs informational-only:

```
Min Wave → Flip → Volume → BB+Squeeze → HMA Gate → Hpx → EMA Stack → XWave → Session → ADR → Chop → Daily Limits
    ↓        ↓     req_V    req_B        GATE(HMA)  req_H   req_E      req_W   req_S     TOGGLE  GATE    REQUIRED
  ALWAYS   ALWAYS  ON       ON           TOGGLE     OFF     OFF        OFF     ON
```

With defaults (V/B/S = ON, H/E/W = OFF), behavior is identical to v2.0.2. Enable `req_H`, `req_E`, or `req_W` to promote those factors from informational → required.

### Gate 1: Minimum Wave Length (Required, default 3 bricks)
The prior directional wave must be at least N bricks long. Prevents whipsaw chains on 1-2 brick chop flips. Set to 1 to disable.

### Gate 2: Renko Flip (Always Required)
A new brick in the opposite direction of the prior run.

### Gate 3: Volume Exhaustion (req_V — default ON)
**Wave Compare (Recommended for Renko):** Accumulates volume per directional wave. Two detection modes (both ON by default, independently toggleable):

- **Exhaustion** — Ending wave had LESS volume than prior same-direction wave. Sellers/buyers tiring out.
- **Climax** — Ending wave had MORE volume than prior (blowoff). Configurable multiplier (default 1.2x = 20% higher).

**Wave Compare Settings (v2.0.7):**
- `Wave: Enable Exhaustion Detection` — Toggle declining-volume triggers
- `Wave: Enable Climax Detection` — Toggle blowoff/spike triggers  
- `Wave: Climax Multiplier` — Min ratio for climax (1.0 = any increase, 1.5 = 50% higher)
- `Wave: Min Wave Volume` — Filter out thin waves. Try 500–2000 for YM/NQ to exclude noise.

**Volume ROC:** Compares fast volume MA (5) to slow MA (20). Better suited for time-based/Heiken Ashi charts.

**Z-Score:** Per-brick spike detection. Works poorly on Renko. Included for time-based charts.

**OFF:** No volume gate. Fires on every flip that meets other conditions.

### Gate 4: Bollinger Bands (Confluence Only as of v2.0.6)
BB band-edge is now a confluence factor (+1), not a gate. Still contributes to confluence score but doesn't block signals. Squeeze filter remains active — when BB width < 60% of its 50-bar MA, all signals are blocked until bands expand.

### Gate 5: HMA Gate — HMA × EMA 21 (Toggleable via Trend Filter group)
HMA 20 (OHLC4) crossing through EMA 21 signals a momentum shift. HMA below EMA 21 → longs blocked. HMA above EMA 21 → shorts blocked. This is a separate, always-evaluated gate — independent from the Hpx confluence factor.

### Gate 6: Hpx Confluence (req_H — default OFF)
Price position relative to HMA 20. Long: close below HMA (entering into bearish structure). Short: close above HMA (entering into bullish structure). Reflects the mean-reversion premise. Turn ON to require this in addition to the HMA Gate.

### Gate 7: EMA Stack (req_E — default OFF)
Requires full 21/34/50 alignment in the signal direction. Reduces trade count in transitional markets. Turn ON to require trend stack for every entry.

### Gate 8: Cross-Wave Volume Ratio (req_W — default OFF)
Requires counter-wave volume ≤ configured ratio vs dominant wave. Only meaningful when EMAs are stacked. Turn ON to require hollow counter-waves.

### Gate 9: Session Window (req_S — default ON)
Default: 0930-1145, 1330-1600 ET. CL uses 0900-1430.

### Gate 10: ADR Exhaustion Filter (Toggleable, default ON)
80% of ADR consumed → new entries suppressed. 70% → trail tightens 30%.

### Gate 11: Chop Detector (Toggleable, default ON)
Averages the last 5 completed wave lengths. Below 5 bricks average → flip-heavy chop → all signals blocked.

### Gate 12: Daily Limits (Required)
DD limit ($3,000) and profit target (disabled by default). Flattens and blocks when hit.

---

## The 7 Confluences (v2.0.3)

Scored on every signal label and dashboard. Default behavior: informational only. Each factor can be promoted to a **required gate** via the 🎯 Confluence Requirements group.

| # | Code | Factor | Long | Short | Default |
|---|------|--------|------|-------|---------|
| 1 | F | Renko Flip | Bull brick after bear run | Bear brick after bull run | Always required |
| 2 | V | Volume Exhaustion | Wave decline/climax at low | Wave decline/climax at high | req_V = ON |
| 3 | B | BB Outside Bands | At/below lower BB (2-bar) | At/above upper BB (2-bar) | req_B = ON |
| 4 | S | Session Active | In trading window | In trading window | req_S = ON |
| 5 | Hpx | HMA Position | Price < HMA 20 | Price > HMA 20 | req_H = OFF |
| 6 | E | EMA Alignment | 21 > 34 > 50 stacked | 21 < 34 < 50 stacked | req_E = OFF |
| 7 | W | Cross-Wave Vol Ratio | Dip vol ≤50% of push vol (bull stack) | Pop vol ≤50% of push vol (bear stack) | req_W = OFF |

**Label example:** `VEB WvDecl 5b 5/7` = VOL BOTTOM, Wave Decline, 5-brick wave, 5 of 7 confluences active.

---

## Position Management

- **Pyramiding = 2:** Max 2 independent positions, same direction
- **Exits per position:** TP (fixed), Trail (Fixed/ATR/Wider of Both), BE (after N bricks)
- **No fixed stop loss** — only TP, trail, BE, and flatten-reverse
- **Flatten-and-reverse:** Opposite signal closes all, enters new direction

---

## New v2.0.3 Settings — Defaults and Tuning

### Trail Activation Bricks

| Setting | Default | Conservative | Aggressive |
|---------|---------|-------------|------------|
| Trail Activation | 1 brick | 2–3 bricks | 1 brick |

At 1 brick (default), trail behavior matches v2.0.2. Increase to 2–3 bricks to give trades more room before the trail engages — reduces premature exits on initial volatility but gives back more on reversals.

### Confluence Requirements (🎯 group)

| Toggle | Default | Effect When ON |
|--------|---------|---------------|
| req_V — Volume Exhaustion | ON | Signal requires volume confirmation |
| req_B — BB Band-Edge | ON | Signal requires price at BB edge |
| req_S — Session Window | ON | Signal requires active session |
| req_H — Hpx (HMA Position) | OFF | Signal requires price on mean-reversion side of HMA |
| req_E — EMA Stack | OFF | Signal requires full 21/34/50 trend alignment |
| req_W — Cross-Wave Ratio | OFF | Signal requires hollow counter-wave (needs stack) |

**Note:** req_H, req_E, and req_W are OFF by default. Enabling them significantly reduces trade count — test one at a time in backtest before combining.

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
| HMA Gate (HMA×EMA21) | Momentum shift trades | ON |
| Trail Activation Bricks | Trail kicking in too early | 1 brick (immediate) |
| Breakeven Stop | Winners turning to losers | ON (BE+3 after 3b) |
| Daily DD Limit | Blowing up on bad day | $3,000 |
| Daily PT Target | Overtrading winning day | Disabled |
| Slippage | Unrealistic backtest P&L | 3 ticks |
| Safety Flatten | Broker position desync | Disabled |

---

## Dashboard Reference (19 rows)

| Row | Label | Shows |
|-----|-------|-------|
| 0 | VES v2.0.3 | Preset, chart type, debug |
| 1 | TP/TRAIL | TP + trail with mode (FIX/ATR/MAX) + ⚡ tighten |
| 2 | SIGNAL | scanning / VEB / VEP + chop status + avg wave len |
| 3 | Position | FLAT / LONG D:1/2 / SHORT D:2/2 + trail level |
| 4 | Session | Active / Outside |
| 5 | Daily P&L | P&L, trades W/L, DD/PT limits |
| 6 | ADR | % used, OK / TIGHTENED / EXHAUSTED |
| 7 | BB | Position + squeeze status + width vs threshold |
| 8 | HMA Gate | HMA vs EMA 21 position + HMA slope |
| 9 | EMA Stack | 21/34/50 alignment (BULL / BEAR / MIXED) |
| 10 | XWave Ratio | Cross-wave volume comparison + ratio % |
| 11 | Confluence | X/7 with F·V·B·S·Hpx·E·W breakdown |
| 12 | Win Rate | Win % + closed trades |
| 13 | Net P&L | All-time P&L + max DD |
| 14 | Prof Factor | PF + W/L count |
| 15 | Expectancy | $/trade |
| 16 | Avg W/L | Avg win $ / avg loss $ |
| 17 | Vol [METHOD] | Exhaustion status + method stats |
| 18 | 🔧 GATES | F·V·BB·TR + Hpx·E·W·SS·CH·ADR pass/fail |

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
| **2.0.3** | **ADR repaint fix, trail activation bricks, confluence requirements group (req_H/E/W), HMA Gate rename, Hpx labeling, debug row expansion** |

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
