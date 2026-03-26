# Volume Exhaustion Strategy (VES) — User Guide
**Version:** 1.16.0  
**Chart Type:** Renko Traditional (any base timeframe, down to 1s)  
**Instruments:** YM, MYM, NQ, MNQ, ES, MES, CL, MCL, HG  

---

## What This Strategy Does

VES is a **mean-reversion strategy** that detects when buying or selling pressure has exhausted at a swing extreme, then enters the reversal. It fires **VOL BOTTOM** (long) signals at swing lows and **VOL PEAK** (short) signals at swing highs. It manages two independent positions with take-profit, trailing stop, and breakeven stop exits.

---

## How It Works — The Signal Chain

A trade only fires when ALL required gates pass simultaneously:

```
Min Wave → Renko Flip → Volume Exhaustion → BB Outside + No Squeeze → Trend → Session → ADR → Daily Limits
    ↓           ↓              ↓                    ↓                   ↓        ↓        ↓         ↓
 REQUIRED    REQUIRED       REQUIRED            REQUIRED              SOFT    REQUIRED  TOGGLE    REQUIRED
```

### Gate 1: Minimum Wave Length (Required, default 3 bricks)
The prior directional wave must be at least N bricks long. Prevents whipsaw chains on 1-2 brick chop flips. Set to 1 to disable.

### Gate 2: Renko Flip (Always Required)
A new brick in the opposite direction of the prior run.

### Gate 3: Volume Exhaustion (Required — 3 Methods + OFF)
**Wave Compare (Recommended for Renko):** Accumulates volume per directional wave. Fires on Wave Decline (ending wave had less volume — tiring out) or Wave Climax (blowoff — more volume but nobody left).

**Volume ROC:** Compares fast volume MA (5) to slow MA (20). Fires when volume is decelerating or below average at the flip.

**Z-Score:** Per-brick spike detection. Works poorly on Renko. Included for time-based charts.

**OFF:** No volume gate. Fires on every flip that meets other conditions.

### Gate 4: Bollinger Bands (Required — Includes Squeeze Filter)
1. **Band Position:** Price at/beyond BB edge within last 2 bars.
2. **Squeeze Filter (v1.15.0):** BB width must be ≥ 2× TP distance. If TP=40pt but bands are 30pt wide, TP is unreachable — all signals blocked until bands expand.

### Gate 5: Trend Filter (Soft — Toggleable)
Blocks longs in strong downtrends (EMAs stacked bearish + below 233). Blocks shorts in strong uptrends.

### Gate 6: Session Window (Required)
Default: 0930-1145, 1330-1600 ET. CL uses 0900-1430.

### Gate 7: ADR Exhaustion Filter (Toggleable, default ON)
80% of ADR consumed → new entries suppressed. 70% → trail tightens 30%.

### Gate 8: Daily Limits (Required)
DD limit ($3,000) and profit target (disabled by default). Flattens and blocks when hit.

---

## The 6 Confluences

Informational score on each signal label and dashboard. Does NOT gate entries.

| # | Code | Factor | Long | Short |
|---|------|--------|------|-------|
| 1 | F | Renko Flip | Bull brick after bear run | Bear brick after bull run |
| 2 | V | Volume Exhaustion | Wave decline/climax at low | Wave decline/climax at high |
| 3 | B | BB Outside Bands | At/below lower BB (2-bar) | At/above upper BB (2-bar) |
| 4 | S | Session Active | In trading window | In trading window |
| 5 | H | HMA Position | Price < HMA 20 | Price > HMA 20 |
| 6 | E | EMA Alignment | 9 > 15 > 34 stacked | 9 < 15 < 34 stacked |

**Label example:** `VEB WvDecl 5b 4/6` = VOL BOTTOM, Wave Decline, 5-brick wave, 4/6 confluences.

---

## Position Management

- **Pyramiding = 2:** Max 2 independent positions, same direction
- **Exits per position:** TP (fixed), Trail (Fixed/ATR/Wider of Both), BE (after N bricks)
- **No fixed stop loss** — only TP, trail, BE, and flatten-reverse
- **Flatten-and-reverse:** Opposite signal closes all, enters new direction

---

## ATR Adaptive Trail (v1.13.0)

| Mode | Behavior |
|------|----------|
| Fixed (default) | Original 40pt trail. Consistent. |
| ATR | Trail = ATR × multiplier (default 1.5). Adapts to volatility. |
| Wider of Both | MAX(fixed, ATR). Fixed floor + ATR room. |

---

## ADR Exhaustion Filter (v1.13.0)

| Threshold | Default | Action |
|-----------|---------|--------|
| Tighten | 70% ADR | Trail reduced 30% |
| Suppress | 80% ADR | New entries blocked |

Dashboard: `62% of 285pt ✓ OK` / `78% ⚡ TIGHTENED` / `85% ⛔ EXHAUSTED`

---

## BB Squeeze Filter (v1.15.0)

Auto: min width = 2× TP. Manual: set fixed points. Dashboard: `⛔ SQUEEZE | 45/80pt`

---

## Min Wave Length (v1.14.0)

Default 3 bricks. Labels show `5b`. Debug shows `F:✗2b<3` when rejected.

---

## Protection Features Summary

| Feature | Prevents | Default |
|---------|----------|---------|
| Min Wave Length | Whipsaw chains on chop | 3 bricks |
| BB Squeeze Filter | Entries where TP unreachable | ON (2× TP) |
| ADR Exhaustion | Late entries, range spent | ON (80%) |
| ADR Trail Tighten | Giving back profits | ON (70%) |
| Trend Filter | Catching falling knives | ON (soft) |
| Breakeven Stop | Winners turning to losers | ON (BE+3 after 3b) |
| Daily DD Limit | Blowing up on bad day | $3,000 |
| Daily PT Target | Overtrading winning day | Disabled |
| Slippage (v1.16.0) | Unrealistic backtest P&L | 3 ticks |
| Safety Flatten (v1.16.0) | Broker position desync | Disabled |

---

## Dashboard Reference (18 rows)

| Row | Label | Shows |
|-----|-------|-------|
| 0 | VES v1.16.0 | Preset, chart type, debug |
| 1 | TP/TRAIL | TP + trail with mode (FIX/ATR/MAX) + ⚡ tighten |
| 2 | SIGNAL | scanning / VEB / VEP + Z-score |
| 3 | Position | FLAT / LONG D:1/2 / SHORT D:2/2 + trail level |
| 4 | Session | Active / Outside |
| 5 | Daily P&L | P&L, trades W/L, DD/PT limits |
| 6 | ADR | % used, OK / TIGHTENED / EXHAUSTED |
| 7 | BB | Position + squeeze status + width |
| 8 | HMA 20 ℹ | Price vs HMA, slope (info only) |
| 9 | Confluence | X/6 with FVBSHE breakdown |
| 10 | Win Rate | Win % + closed trades |
| 11 | Net P&L | All-time P&L + max DD |
| 12 | Prof Factor | PF + W/L count |
| 13 | Expectancy | $/trade |
| 14 | Avg W/L | Avg win $ / avg loss $ |
| 15 | Trend | EMA stacking |
| 16 | Vol [METHOD] | Exhaustion status + method stats |
| 17 | 🔧 GATES | F V BB TR SS ADR pass/fail |

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

All tooltips show `(Default: X)` for recovery if saved over.

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
| 1.16 | Slippage 3 ticks, ATS safety flatten, input layout (QZeus style) |

---

## Quick Start

1. Add to Renko Traditional 8-tick chart (YM, 30s base)
2. Select instrument preset
3. Detection Method → "Wave Compare"
4. Min Wave Length → 3
5. BB Squeeze → ON
6. Debug OFF, all gates ON
7. Check GATES row if no signals appear
8. Wire QuantLynk to SIM for real fills
