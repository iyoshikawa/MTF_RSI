# Volume Exhaustion Strategy (VES) — User Guide
**Version:** 1.10.0  
**Chart Type:** Renko Traditional (any base timeframe, down to 1s)  
**Instruments:** YM, MYM, NQ, MNQ, ES, MES, CL, MCL, HG  

---

## What This Strategy Does

VES is a **mean-reversion strategy** that detects when buying or selling pressure has exhausted at a swing extreme, then enters the reversal. It trades the idea that when volume dries up or climaxes at a high/low, the move is done and price will snap back.

The strategy fires **VOL BOTTOM** (long) signals at swing lows and **VOL PEAK** (short) signals at swing highs. It manages two independent positions with take-profit and trailing stop exits, plus a breakeven stop that protects capital once the trade moves in your favor.

---

## How It Works — The Signal Chain

A trade only fires when ALL required gates pass simultaneously:

```
Renko Flip → Volume Exhaustion → BB Outside Bands → Trend Filter → Session → Daily Limits
     ↓              ↓                    ↓               ↓            ↓           ↓
  REQUIRED       REQUIRED            REQUIRED          SOFT       REQUIRED    REQUIRED
```

### Gate 1: Renko Flip (Always Required)
A new brick forms in the opposite direction of the prior run. Bullish brick after bearish run = potential VOL BOTTOM. Bearish brick after bullish run = potential VOL PEAK.

### Gate 2: Volume Exhaustion (Required — 3 Methods)
Detects when the dominant side has run out of gas. Select the detection method in settings:

**Wave Compare (Recommended for Renko)**  
Accumulates total volume across each directional wave (consecutive same-color bricks). When the ending wave's volume declines vs the prior same-direction wave, that side is exhausting. Also fires on climactic volume (blowoff top/bottom where the ending wave exceeded the prior).

**Volume ROC**  
Measures volume rate-of-change using fast vs slow volume MAs. Fires when volume is decelerating or below average at the flip point.

**Z-Score**  
Per-brick volume spike detection. Compares current bar's volume to a rolling mean. Works poorly on Renko (uniform brick volume) — included for time-based charts.

**OFF**  
Disables volume gate entirely. Fires on every Renko flip that meets other conditions.

### Gate 3: Bollinger Bands (Required)
Price must have been at or beyond the BB band edge within the last 2 bars. For longs, price touched or crossed below the lower band. For shorts, price touched or crossed above the upper band. The 2-bar lookback catches the turn — price was at the extreme, now the brick just flipped.

BB Settings match TradingView's standard indicator: SMA basis, close source, configurable length (default 10), std dev (default 1.2), calculated on 3-minute timeframe via `request.security()`.

### Gate 4: Trend Filter (Soft — Can Be Disabled)
Prevents entries directly into a strong stacked trend. If all EMAs are stacked bearish (9 < 15 < 34) AND price is below the 233 EMA, VOL BOTTOM longs are suppressed. Mirror for shorts. This is a "don't catch a falling knife" filter, not a hard block.

### Gate 5: Session Window (Required)
Only trades during configured time windows. Default: 0930-1145 and 1330-1600 ET. Configurable via string format supporting multiple windows and day filters.

### Gate 6: Daily Limits (Required)
Blocks new entries if the daily drawdown limit or daily profit target has been hit. Flattens all positions when triggered.

---

## The 6 Confluences

Each signal is scored on 6 independent confluence factors. The count is displayed on the signal label and dashboard. This is **informational** — it does not gate entries but helps you assess signal quality at a glance.

| # | Code | Factor | Long Condition | Short Condition |
|---|------|--------|----------------|-----------------|
| 1 | F | Renko Flip | Bullish brick after bearish run | Bearish brick after bullish run |
| 2 | V | Volume Exhaustion | Wave decline/climax at low | Wave decline/climax at high |
| 3 | B | BB Outside Bands | Price at/below lower BB (2-bar) | Price at/above upper BB (2-bar) |
| 4 | S | Session Active | Within configured trading window | Within configured trading window |
| 5 | H | HMA Position | Price below HMA 20 | Price above HMA 20 |
| 6 | E | EMA Alignment | 9 > 15 > 34 (buying dip in uptrend) | 9 < 15 < 34 (selling rally in downtrend) |

**Reading the label:** `VEB WvDecl 4/6` means Volume Exhaustion Bottom, detected via Wave Decline, with 4 of 6 confluences met. Hover the label for the full breakdown.

**Reading the dashboard:** The Confluence row shows `L:FVB·HE` — Long setup with Flip, Volume, BB passing (✓), Session missing (·), HMA and EMA passing.

---

## Position Management

### Scaling: 2 Independent Positions
- **Pyramiding = 2**: Max 2 positions in the same direction
- Each enters on its own VOL signal (can be same bar or different bars)
- D:1/2 = first position, D:2/2 = second position
- Each tracks its own entry price, TP, and trailing stop independently

### Exit Rules (Per Position)
1. **Take Profit**: Fixed distance from entry (default 40 points YM)
2. **Trailing Stop**: Fixed distance from best price reached (default 40 points)
3. **Breakeven Stop**: After N bricks in favor (default 3), stop moves to entry + offset ticks (default 3 ticks). Protects capital — worst case becomes +3 ticks profit instead of a full loss.

**No fixed stop loss** — TP, trail, and BE are the only exits (plus flatten on reversal).

### Flatten-and-Reverse
When an opposite signal fires while in a position, ALL open positions are closed, then the new direction is entered. Example: Long D:1/2 + D:2/2 open → VOL PEAK fires → close both longs → enter short D:1/2.

---

## Moving Averages

All EMAs use **OHLC4 source** to smooth wick noise on Renko.

| MA | Length | Purpose |
|----|--------|---------|
| EMA 9 | Fast | Immediate momentum |
| EMA 15 | Medium | Short-term trend |
| EMA 34 | Primary | Trend direction, wick touch dots |
| EMA 233 | Macro | Strong trend anchor (Fibonacci) |
| HMA 20 | Hull | Confluence factor (dashboard only, not plotted) |

---

## Bollinger Bands

Calculated on a **3-minute timeframe** via `request.security()`, not on Renko bricks. This gives a time-consistent volatility envelope regardless of brick formation speed.

| Setting | Default | Notes |
|---------|---------|-------|
| Length | 10 | SMA lookback |
| Std Dev | 1.2 | Tighter than standard 2.0 |
| Source | close | Matches TradingView standard BB |
| Timeframe | 3 min | Configurable |

---

## Dashboard Reference

| Row | Label | What It Shows |
|-----|-------|---------------|
| 0 | VES v1.10.0 | Preset, chart type, debug status |
| 1 | TP/TRAIL | Take profit and trail distances |
| 2 | SIGNAL | Current signal: scanning, VEB, or VEP |
| 3 | Position | FLAT, LONG D:1/2, SHORT D:2/2, etc. |
| 4 | Session | Active/Outside trading window |
| 5 | Daily P&L | Running daily P&L, trade count W/L, DD/PT limits |
| 6 | BB | Price position relative to bands, L/S gate status |
| 7 | HMA 20 ℹ | Price vs HMA, slope direction (info only) |
| 8 | Confluence | X/6 count with factor breakdown |
| 9 | Win Rate | Win % with total closed trades |
| 10 | Net P&L | All-time P&L with max drawdown |
| 11 | Prof Factor | Profit factor with W/L count |
| 12 | Expectancy | $/trade |
| 13 | Avg W/L | Average win $ vs average loss $ |
| 14 | Trend | EMA stacking + macro direction |
| 15 | Vol [METHOD] | Volume exhaustion method status + stats |
| 16 | 🔧 GATES | Debug gate diagnostic (pass/fail per gate) |

---

## Daily Risk Management

| Setting | Default | Description |
|---------|---------|-------------|
| Daily DD Limit | $3,000 | Flattens all positions and blocks entries for rest of day. Set to 0 to disable. |
| Daily Profit Target | $0 (disabled) | Flattens and locks in the day when target hit. |

Both reset automatically at the start of each new trading day.

---

## QuantLynk Webhook Integration

The strategy includes built-in QuantLynk webhook payloads for automated execution.

### Setup
1. In strategy settings → 🔗 QuantLynk Webhook:
   - Enable QuantLynk Alerts
   - Enter your QuantLynk User ID
   - Enter your QuantLynk Alert ID
2. In TradingView alert dialog:
   - Create alert on the strategy
   - Set Message to: `{{strategy.order.alert_message}}`
   - Add webhook URL: `https://lynk.quantvue.io/quantlynk/place-order`

### What Fires
| Event | QuantLynk Action |
|-------|------------------|
| VOL BOTTOM entry | Market buy, qty 1 |
| VOL PEAK entry | Market sell, qty 1 |
| Flatten + reverse | Flatten, then buy/sell |
| TP / Trail / BE hit | Sell 1 (long) or Buy 1 (short) |
| Session end | Flatten |
| Daily DD / PT limit | Flatten |

**Always test on a SIM account first.** QuantLynk flatten is account-wide.

---

## Debug Tools

### Debug Bypass
Turn ON to fire signals on every Renko flip regardless of gates. Use to verify the engine works, then turn OFF.

### Individual Gate Toggles
Each gate can be disabled independently to isolate which one is blocking signals:
- 🔧 Require Volume Exhaustion
- 🔧 Require BB Outside Bands
- 🔧 Require Trend Filter
- 🔧 Require Session Filter

The GATES row on the dashboard shows real-time pass/fail for each gate.

---

## Instrument Presets

Select your instrument in settings to auto-configure TP, Trail, and defaults:

| Preset | TP | Trail | Notes |
|--------|-----|-------|-------|
| 🔷 YM (Dow) | 40 pts | 40 pts | Default. 1 pt = 1 tick on YM/MYM |
| 📈 NQ (NASDAQ) | 40 pts | 40 pts | Same default — adjust for MNQ |
| 📊 ES (S&P) | 8 pts | 8 pts | Smaller range instrument |
| 🛢️ CL (Crude) | 0.20 | 0.20 | Decimal pricing |
| 🟤 HG (Copper) | 0.0050 | 0.0050 | Decimal pricing |
| ⚙️ CUSTOM | User | User | Manual TP/Trail input |

---

## Backtesting Notes

**⚠️ TradingView Renko backtests are unreliable.** Renko bricks use synthetic prices — fills happen at perfect brick close prices with zero slippage. Backtest stats show directional edge but dollar amounts are inflated.

**For reliable performance data:**
1. Forward test on a SIM account via QuantLynk webhooks
2. Track real fills for at least 1 week per instrument
3. Compare SIM results to backtest — expect 30-50% lower performance on SIM

---

## Quick Start

1. Add strategy to Renko Traditional 8-tick chart (YM, 30s base)
2. Select your instrument preset
3. Set Detection Method to "Wave Compare"
4. Turn OFF debug mode, turn ON all gate toggles
5. If no signals appear, turn OFF volume gate → check if BB is the blocker
6. Once signals fire, compare placement to your reference strategy
7. Wire QuantLynk to SIM account for real fill tracking
