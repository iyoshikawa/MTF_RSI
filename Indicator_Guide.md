# MTF RSI Indicator Guide

**Signals v7.8.0 · Management v7.5.0**

Complete reference for every signal, marker, dashboard element, and setting in the MTF RSI Signal Suite.

---

## Table of Contents

1. [System Overview](#system-overview)
2. [Chart Markers — What Every Symbol Means](#chart-markers)
3. [Dashboard Reference — Signals Indicator](#signals-dashboard)
4. [Dashboard Reference — Management Indicator](#management-dashboard)
5. [Grading System — How Grades Are Calculated](#grading-system)
6. [Multi-Timeframe (MTF) Analysis](#mtf-analysis)
7. [Trade Lifecycle — Entry to Exit](#trade-lifecycle)
8. [Volume Analysis System](#volume-analysis)
9. [RSI Divergence Detection](#rsi-divergence)
10. [Anti-Chop Filters](#anti-chop-filters)
11. [Session & Algo Windows](#sessions-and-algos)
12. [Instrument Presets](#instrument-presets)
13. [Renko & Non-Standard Chart Support](#renko-support)
14. [Settings Reference — Signals](#settings-signals)
15. [Settings Reference — Management](#settings-management)
16. [Alert Reference](#alert-reference)

---

## System Overview

The MTF RSI Signal Suite is a two-indicator system for futures day trading on TradingView.

**Signals indicator** generates graded entry signals (A+ through B) by analyzing confluence across multiple timeframes, trend direction, momentum oscillators, volume, and volatility filters. It plots entry signals, stop/take profit levels, volume markers, divergence warnings, and a real-time dashboard on the chart.

**Management indicator** handles active trade management: partial exit targets (TP1/TP2), breakeven stop automation, live P&L tracking, position sizing, runner trail stops, pyramid add signals, and ADX-based recommendations for when to use ATR vs fixed targets.

Both indicators recalculate the same grade independently. Core settings (preset, RSI, stochastic, ADX, session config) must match between them.

---

## Chart Markers

### Grade Signal Labels

These appear when a graded setup is confirmed. Label below price = LONG, above price = SHORT.

| Marker | Grade | Size | Meaning |
|--------|-------|------|---------|
| **A+** | A+ | Large | Highest conviction. 3 HTF aligned + chart RSI + EMA + VWAP + DStoch + volume + ADX strong. Full size, runner candidate. |
| **A** | A | Normal | Strong confluence. Same as A+ but may be missing VWAP or volume confirmation. Runner candidate. |
| **A-** | A- | Small | Good setup. 2 HTF aligned + chart RSI + EMA + DStoch + macro trend filter. Standard exit at TP. |
| **B+** | B+ | Small | Trend scalp. 2 HTF aligned + HMA trend + anti-chop filters passed. In and out at TP. |
| **B** | B | Tiny | Quick scalp. 1 HTF aligned + HMA trend. Lowest conviction. Tight leash, 3-5 bars max. |

Colors are fully configurable in settings. Default: green shades for long, red/purple shades for short.

Signals repeat at a configurable reminder interval (~10 min default) while the grade remains active, so you see the label periodically even if you missed the initial signal.

### DStoch Crossover Markers

| Marker | Meaning |
|--------|---------|
| **DS▲** (below price) | Double Stochastic crossed UP through the buy level. Bullish momentum shift. |
| **DS▼** (above price) | Double Stochastic crossed DOWN through the sell level. Bearish momentum shift. |

These fire independently of the grading system. Useful for timing entries within an existing grade — a DS▲ during an active A+ long confirms momentum is with you.

### HA Exit Warning

| Marker | Meaning |
|--------|---------|
| **HA✗** (above price during long) | Heikin Ashi candles flipped bearish for N consecutive bars while you're in a long setup. Momentum is turning against you. Consider tightening stop or exiting. |
| **HA✗** (below price during short) | Heikin Ashi flipped bullish during a short setup. Same warning — momentum fading. |

The number of consecutive bars before this fires is configurable (default 2). This is an early warning — it fires before your SL is hit.

### RSI Divergence Markers

| Marker | Meaning |
|--------|---------|
| **DIV▼** (triangle down, above price) | Bearish divergence. Price made a higher high but RSI made a lower high. Momentum is weakening even though price pushed up. Warns that a long position may be exhausting. |
| **DIV▲** (triangle up, below price) | Bullish divergence. Price made a lower low but RSI made a higher low. Selling pressure is fading. Warns that a short position may be losing steam. |

Divergence is detected using pivot-based comparison on the short RSI oscillator. The lookback window and minimum pivot separation are configurable.

### Volume Spike Markers

| Marker | Color | Meaning |
|--------|-------|---------|
| **V+** (diamond, below price) | Green | Volume spike **early** in the signal (within first 5 bars). Confirms the move — volume is supporting the breakout. Stay in. |
| **V!** (diamond, above price) | Orange | Volume spike **late** in the signal (after 10+ bars). Potential exhaustion — climax volume after an extended run often marks a top/bottom. Consider taking profits. |
| **V** (diamond, below price) | White | Volume spike with no clear directional context. Noteworthy volume but neither confirming nor exhausting. |

A volume spike is defined as the current bar's volume exceeding the mean + N standard deviations over the lookback period (default: 2σ over 50 bars).

### Volume-Price Divergence Flags

| Marker | Color | Meaning |
|--------|-------|---------|
| **VD▼** (flag, above price) | Red | Bearish volume divergence. Price is rising but volume is declining over the lookback window. During a long setup, this warns that the move lacks conviction and may reverse. |
| **VD▲** (flag, below price) | Blue | Bullish volume divergence. Price is falling but volume is declining. During a short setup, selling is drying up — the downside move may be exhausting. |

### Stop Loss & Take Profit Levels (Signals Indicator)

| Marker | Color (default) | Meaning |
|--------|----------------|---------|
| **Cross below price** (long) or **above** (short) | Red | Fixed stop loss level for the current grade. Distance varies by grade and instrument. |
| **Cross above price** (long) or **below** (short) | Cyan | Fixed take profit level for the current grade. First target for partials. |

These levels lock at entry price when a signal fires and remain as flat horizontal lines until hit or the grade resets. SL distance depends on the SL Mode setting (Fixed, ATR, or Wider of Both). On grade upgrades, TP adjusts to the median of the old and new grade's target.

### Management Indicator — Chart Lines

| Line | Color (default) | Style | Meaning |
|------|----------------|-------|---------|
| **TP1** | Cyan | Solid line | Fixed take profit — partial exit target (take 50%). Disappears when hit. |
| **TP2** | Purple | Solid line | ATR-based runner target — A grades only. Where to consider closing the remaining runner position. |
| **Breakeven Stop** | Gold | Step line | Appears AFTER TP1 is hit. Plotted at entry price. Move your stop here — risk is now zero. |
| **Runner Trail** | Cyan | Step line | Trailing stop based on selected EMA (34 EMA, 21 HMA, or 21 EMA). Follows the trend — exit when price closes beyond it. |
| **ATR Stop Reference** | Orange | Large cross | ATR × 1.5 distance from price. A-grade only. Shows where an ATR-based stop would be vs the fixed stop. |
| **Pyramid Add (+)** | Cyan | Arrow up/down | Price pulled back to the trail EMA and bounced. Add to winning position (if within pyramid limit). |
| **ADX Breakout (BRK)** | Gold | Square | ADX crossed from low (< 12) to high (> 20). Range breakout detected — a new trend may be starting. |

### Candle Coloring

When "Color Candles by Grade" is enabled, candle bodies change color based on the current active grade. When "Heikin Ashi Outline" is enabled, the body shows grade color while wicks/borders show Heikin Ashi direction (green = HA bullish, red = HA bearish). This gives you two pieces of information on every candle: what grade is active and which direction HA momentum is pointing.

### RSI Momentum Ribbon

The background color of the chart changes based on the double RSI momentum direction. Green background = RSI bullish (short RSI > long RSI by threshold). Red background = RSI bearish. Gray = neutral. The ribbon brightens when DStoch and RSI align (confluence), and dims when volume is contracting (if enabled). During consolidation, the ribbon can turn to a configurable consolidation color (default black).

---

## Signals Dashboard

Located in a configurable corner (default top right). Shows real-time system state.

| Row | Label | What It Shows |
|-----|-------|---------------|
| 0 | **PRESET** | Instrument preset + chart type. E.g. "📈 NQ \| CANDLE" or "🔷 YM \| RENKO 5pt ⚡" |
| 1 | **SETUP GRADE** | Current grade (A+, A, A-, B+, B, or NONE). Shows "⚠ LATE" if ADX is falling or ADR > 70% used on A+/A grades. |
| 2 | **DIRECTION** | LONG, SHORT, or NEUTRAL. Green/red background. |
| 3 | **SIGNAL AGE** | Time since signal fired + bar/brick count. Green = fresh (≤3 bars), yellow = moderate, orange = aging. |
| 4 | **VOL FLOW** | Volume expansion/contraction state. "EXPANDING 1.4x" (green), "CONTRACTING 0.7x" (red), or "STEADY" (gray). The ratio is fast vol MA / slow vol MA. |
| 5 | **VOL SPIKE** | Current bar volume status. Shows "CONFIRM 2.3x avg" (green) if spike early in signal, "EXHAUST 3.1x avg" (orange) if late, or "✓ ACTIVE" / "✗ LOW" for normal volume filter status. |
| 6 | **VOL DIVG** | Volume-price divergence. "⚠ PRICE ▲ VOL ▼" (orange) = bearish divergence during long. "✓ ALIGNED" (green) = no divergence. |
| 7 | **SESSION** | Whether current bar is within configured trading hours. Shows "~" prefix on Renko (approximate). |
| 8 | **ALGO** | Algo window status. Shows which window is active (LONDON, NY OPEN, NY PM, etc.) or OFF. |
| 9 | **ADX TREND** | ADX value + strength label. "🔥 STRONG" (≥ A+ threshold), "✓ GOOD" (≥ A threshold), "⚠ WEAK" (below). |
| 10 | **DSTOCH** | Double Stochastic value + momentum direction. BULL (> 50), BEAR (< 50), NEUTRAL. |
| 11 | **HMA TREND** | Hull MA trend state. STRONG LONG, LONG, NEUTRAL, SHORT, STRONG SHORT. Based on HMA 21/34/50 stack. |
| 12 | **MTF** | MTF speed setting + the three timeframes being used. E.g. "Conservative: 5 / 15 / 60". |
| 13+ | **REGIME / CP SCORE / COMPRESS / RSI DIV** | Optional rows for consolidation regime analysis and RSI divergence status. Only visible when their features are enabled. |

---

## Management Dashboard

Two dashboard panels: Trade Context (bottom left) and Volatility/Sizing (configurable corner).

### Trade Context Panel

| Row | Label | What It Shows |
|-----|-------|---------------|
| 0 | **TRADE CONTEXT** | Header |
| 1 | **ENTRY** | Locked entry price + grade when signal fired. E.g. "19523.5 A+". Blank when no trade. |
| 2 | **P&L** | Live unrealized P&L in points and dollars. Uses grade-appropriate contract count × dollar per point. Green when profitable, red when not. |
| 3 | **TP STATUS** | "TP1: 19548.0" before hit, "✓ TP1 HIT → BE stop" after TP1 is reached. |
| 4 | **HA MOMENTUM** | Heikin Ashi direction + consecutive bar count. "BULL 8 bars" or "BEAR 3 bars". |
| 5 | **MACRO ALIGN** | Whether your trade direction aligns with the HMA macro trend. "✓ WITH TREND" (green), "✗ COUNTER" (red), or "— NEUTRAL". |
| 6 | **ADR LEFT** | Remaining average daily range in points + percentage. "142.3 pts (58%)". Green > 50%, yellow > 20%, red < 20%. |
| 7 | **RUNNER** | Runner trail status. "✓ TRAILING" (active), "✓ ELIGIBLE" (conditions met), "✗ NO RUN (ADX)" (ADX too weak), or "OFF". |
| 8 | **MTF RSI** | Compact multi-timeframe RSI alignment. "3/3 BULL ▲" (all aligned bullish), "3/3 BEAR ▼", or "2▲ 1▼ MIXED". |

### Volatility & Sizing Panel

| Row | Label | What It Shows |
|-----|-------|---------------|
| 0 | **Header** | Contract label + $/pt + Max Pain amount. E.g. "📐 MYM $0.50/pt ... $400 MAX PAIN" |
| 1 | **ATR(14)** | Current ATR value. |
| 2 | **VOLATILITY** | Volatility class: 🔴 EXTREME (ATR ≥ 2x avg), 🟠 HIGH (≥ 1.4x), 🟢 NORMAL (≥ 0.8x), 🔵 LOW. Plus ratio vs 50-bar average. |
| 3 | **STOP DIST** | Current grade's stop distance in points, ticks, and dollars per contract. |
| 4 | **ADX SIZING** | ADX-based size multiplier (if ADX Advanced enabled). Shows percentage + rising/falling arrow. |
| 5 | **TP MODE** | ADX-driven recommendation. "🔥 ATR TP (ADX 28 ▲)" = use ATR target. "FIXED TP (ADX ▼)" = ADX falling, take the fixed target. "FIXED TP (scalp)" = B grades. "FIXED TP (ADR 70%+)" = daily range exhausting. |
| 6 | **SL MODE** | Which stop approach to use. "🏃 TRAIL (34 EMA)" = runner eligible, trail it. "ATR SL (strong trend)" = use ATR-based stop. "FIXED SL" = use the preset fixed stop distance. |
| 7 | **MAX CTS** | Header for contract sizing. Shows your cap setting. |
| 8-11 | **Grade rows** | Max contracts per grade, sized by that grade's SL. Shows "Grade SL" column (using fixed SL) and "ATR Stop" column (using ATR × 1.5). Each row labels the SL value driving the calculation. |

---

## Grading System

Grades represent confluence depth. More filters aligned = higher grade = higher conviction.

### A+ (Highest Conviction)

All of the following must be true simultaneously:
- 3 higher timeframes RSI aligned (fast + mid + slow all bullish/bearish)
- Chart timeframe RSI direction matches
- Price above/below the 34 EMA
- Price above/below the session VWAP
- Double Stochastic above/below 50 (momentum confirms)
- Volume above the moving average × multiplier
- ADX above the A+ threshold (strong trend confirmed)

### A (Strong)

Same as A+ except VWAP and volume are not required. ADX must still be above the A threshold.

### A- (Good)

- Only 2 higher timeframes need to align (fast + mid)
- Chart RSI + EMA + DStoch must confirm
- Macro trend filter: HMA trend must not be counter to the signal direction
- Not promoted to A or A+ (those take priority)

### B+ (Trend Scalp)

- 2 higher timeframes aligned
- Price above/below HMA 34 (smoother than EMA for scalps)
- Anti-chop filters passed: ADX floor, low volatility check, consolidation check
- Macro trend filter applied
- RSI ribbon confirmation (optional)

### B (Quick Scalp)

- Only 1 higher timeframe aligned (fast)
- Price above/below HMA 34
- Same anti-chop filters as B+

### Grade Priority

The system evaluates from highest to lowest: A+ → A → A- → B+ → B → NONE. A bar can only have one grade. Higher grades take priority — if conditions for both A+ and A are met, it's an A+.

### Grade Upgrade Behavior

When a grade upgrades (e.g. A- → A, A → A+, B → B+) in the same direction, the new grade fires immediately — it does not wait for the persistence countdown. The logic: if the system already confirmed A- for 2 bars and conditions improved to A, that improvement IS confirmation. Waiting 2 more bars for the A to "persist" often means missing it entirely as conditions revert.

| Scenario | Behavior |
|----------|----------|
| A- → A (same direction) | Fires immediately |
| A → A+ (same direction) | Fires immediately |
| B → B+ → A- (chain upgrade) | Each fires immediately |
| A+ → A (downgrade) | Resets persistence, must wait |
| A- → A (direction flipped) | Resets persistence — direction change, not an upgrade |
| NONE → B (new entry) | Must wait persist_bars |

### Late Trend Warning

When an A+ or A grade is active but ADX is falling or ADR usage exceeds 70%, the dashboard appends "⚠ LATE" to the grade. This means the trend may be mature — the signal is valid but the easy money may already be made.

---

## Multi-Timeframe (MTF) Analysis

The indicator checks RSI direction on 3 higher timeframes. The specific timeframes depend on your chart timeframe and MTF Speed setting.

### MTF Speed Options

| Speed | 1-Min Chart | 3-Min Chart | 5-Min Chart | A+ Signal Speed |
|-------|-------------|-------------|-------------|-----------------|
| **Conservative** (default) | 5 / 15 / 60 | 5 / 15 / 60 | 15 / 60 / 240 | Slowest — highest conviction |
| **Moderate** | 5 / 15 / 30 | 5 / 15 / 30 | 15 / 60 / 120 | Medium |
| **Aggressive** | 5 / 9 / 15 | 5 / 9 / 15 | 9 / 15 / 60 | Fastest — needs anti-chop filters |
| **Custom** | User-defined | User-defined | User-defined | Depends on selection |

### How MTF Alignment Maps to Grades

| Alignment | Grades Eligible |
|-----------|-----------------|
| 3/3 aligned (fast + mid + slow) | A+, A |
| 2/3 aligned (fast + mid) | A-, B+ |
| 1/3 aligned (fast only) | B |
| 0/3 aligned | NONE |

The RSI direction on each timeframe is determined by the double RSI system: if the EMA-smoothed short RSI minus the EMA-smoothed long RSI exceeds the threshold, direction is bullish (+1). Below negative threshold = bearish (-1). Between = neutral (0).

---

## Trade Lifecycle

### Entry (Signals Indicator)

1. Grade signal fires (label appears on chart)
2. SL and TP lines lock at entry price (SL uses Fixed, ATR, or Wider of Both mode)
3. Entry price and levels locked internally for alert tracking
4. Dashboard shows grade, direction, signal age

### Active Trade (Management Indicator)

5. TP1 (cyan line) and TP2 (purple line, A grades only) plotted at locked levels
6. Live P&L tracking on dashboard in points and dollars
7. TP MODE and SL MODE rows recommend which exit strategy to use
8. Volume analysis provides real-time conviction feedback

### TP1 Hit

9. TP1 line disappears
10. Breakeven stop (gold step line) appears at entry price
11. Dashboard TP STATUS changes to "✓ TP1 HIT → BE stop"
12. 🎯 TP HIT alert fires

### Runner Phase (A Grades Only)

13. Runner trail (step line) tracks the selected EMA
14. TP2 (ATR-based) remains as the runner target
15. Pyramid add signals may fire on EMA pullbacks
16. Exit when price closes beyond the runner trail, or at TP2

### Exit

17. SL hit (🛑 SL HIT alert) or grade drops to NONE
18. All tracking variables reset
19. Chart levels and dashboard entries clear

---

## Volume Analysis

Three components, all gated behind the "Enable Volume Analysis" toggle.

### Volume Expansion/Contraction

Compares a fast volume MA (default 5 bars) against a slow volume MA (default 20 bars). When the fast MA is above the slow MA and rising, volume is expanding — more participants are entering, confirming momentum. When fast is below slow and falling, volume is contracting — conviction is fading.

The RSI momentum ribbon dims automatically when volume contracts (if "Dim Ribbon on Volume Contraction" is enabled), giving a visual cue that the signal may lack follow-through.

### Volume Spike Detection

A spike is flagged when the current bar's volume exceeds the mean + N standard deviations over the lookback period (default: 2σ over 50 bars). The context depends on timing within the signal:

- **First 5 bars** → CONFIRM (V+ marker, green) — volume validates the breakout
- **After 10+ bars** → EXHAUST (V! marker, orange) — climax volume, potential reversal
- **In between** → Neutral spike (V marker, white)

### Volume-Price Divergence

Detects when price trend and volume trend disagree over a configurable lookback (default 10 bars). Bearish divergence: price rising but volume declining during a long — the move lacks conviction. Bullish divergence: price falling but volume declining during a short — selling is drying up.

---

## RSI Divergence Detection

Uses pivot-based comparison to find classic (regular) divergence between price and the short RSI oscillator.

**Bearish divergence:** Price prints a higher high at the current pivot, but RSI prints a lower high at the same pivot. Momentum is weakening despite price pushing higher. Appears as DIV▼ flag above the bar.

**Bullish divergence:** Price prints a lower low but RSI prints a higher low. Selling pressure is fading. Appears as DIV▲ flag below the bar.

Divergence is a warning overlay — it does not change grades or gate entries. It feeds into the dashboard and the Late Trend flag when divergence is detected against your current position direction.

---

## Anti-Chop Filters

These filters reduce false signals in ranging/choppy markets. All are configurable.

| Filter | What It Does | Default |
|--------|-------------|---------|
| **Signal Cooldown** | Minimum bars/seconds between signals. Prevents rapid-fire in chop. | 5 bars / 60s Renko |
| **Grade Persistence** | Grade must hold for N bars before it fires. Prevents flickering. **Exception:** grade upgrades (e.g. A- → A, A → A+) fire immediately — improving conditions ARE confirmation. Downgrades and direction flips still reset and wait. | 2 bars |
| **ADX Floor for B/B+** | B grades require minimum ADX. Blocks signals in dead markets. | 15 |
| **Suppress B/B+ LOW Vol** | Blocks B grades when ATR ratio classifies volatility as LOW. | OFF (auto ON for CL) |
| **Block Counter-Trend** | Prevents B/B+/A- signals against the HMA macro trend. | ON |
| **Require RSI Ribbon** | B grades must have RSI ribbon in their direction. | OFF |
| **Gray Ribbon During Consolidation** | Ribbon turns gray when price range is tight. | OFF |
| **Consolidation Range** | Threshold (points) for the consolidation detector. | 10 pts |
| **Consolidation Lookback** | Bars to measure range over. | 20 bars |

---

## Sessions and Algo Windows

### Session Config

Format: `TIME1,TIME2:DAYS` — e.g. `0930-1145,1330-1600:23456`

Days: 1=Sun, 2=Mon, 3=Tue, 4=Wed, 5=Thu, 6=Fri, 7=Sat. All times in Eastern (America/New_York).

Signals only fire during configured session windows. CL preset overrides to `0900-1430` (pit session).

On Renko charts, session boundaries are approximate since bricks can span time boundaries. The dashboard shows "SESSION ~" to flag this.

### Algo Windows

Named time windows that correspond to high-probability institutional trading periods. Can be set to "TRADE WITH" (only signal during these windows), "AVOID" (suppress signals during them), or "OFF".

| Window | Time (ET) | Characteristics |
|--------|-----------|----------------|
| London Open | 0200-0500 | European open, thinner but trends cleanly |
| NY Open | 0830-1000 | Highest volume for US futures, economic data |
| NY Afternoon | 1330-1500 | Bond market closes at 1500, equity index reacts |
| NY Close | 1500-1600 | MOC flow, noisy (disabled by default) |
| Asia Session | 2000-0000 | Low volume on US futures (disabled by default) |
| Globex Open | 1800-1900 | Overnight session open (disabled by default) |

---

## Instrument Presets

Selecting a preset auto-configures EMA length, stochastic levels, ADX thresholds, volume multiplier, session hours, and SL/TP values.

| Setting | CL | NQ/MNQ | ES/MES | YM/MYM | Default |
|---------|-----|--------|--------|--------|---------|
| EMA Length | 21 | 34 | 34 | 34 | 34 |
| Stoch OB | 72 | 65 | 60 | 62 | 60 |
| Stoch OS | 30 | 35 | 35 | 38 | 35 |
| Stoch Sell | 80 | 85 | 85 | 82 | 85 |
| Volume Mult | 1.4 | 1.2 | 1.1 | 1.2 | 1.1 |
| ADX A+ | 28 | 23 | 25 | 23 | 25 |
| ADX A | 22 | 18 | 20 | 18 | 20 |
| Session | 0900-1430 | User config | User config | User config | User config |

### Contract Size Selector (Management)

A dropdown in Management settings: "Micro" or "Mini/Standard". Combined with the instrument preset, this sets the dollar per point used in all sizing calculations.

| Preset | Micro | Mini/Standard |
|--------|-------|---------------|
| ES | MES $5/pt | ES $50/pt |
| NQ | MNQ $2/pt | NQ $20/pt |
| YM | MYM $0.50/pt | YM $5/pt |
| CL | MCL $100/pt | CL $1,000/pt |

---

## Renko & Non-Standard Chart Support

As of v7.8.0, the Signals indicator auto-detects chart type and adapts behavior.

### Auto-Detection

`chart.is_renko`, `chart.is_range`, `chart.is_heikinashi`, `chart.is_linebreak`, `chart.is_kagi`, `chart.is_pnf`, `chart.is_standard` — all checked automatically. The dashboard PRESET row shows the detected type.

### Adaptations for Renko

| Feature | Candle Charts | Renko Charts |
|---------|---------------|--------------|
| Signal cooldown | Bar-based (default 5 bars) | Time-based (default 60 seconds, configurable down to 10s) |
| Signal age display | "12m (8 bars)" | "12m (8 bricks)" |
| Session filter | Exact | Approximate (shown as "SESSION ~") |
| Reminder interval | Based on bars/minute | Estimated at 1 bar/minute fallback |
| SL/TP values | Fixed points | Same fixed points (still valid) |
| MTF security calls | Candle data from higher TFs | Same — higher TFs are always candles regardless of chart type |
| Volume analysis | Per-bar volume | Per-brick aggregated volume |

### What to Watch on Renko

Session boundaries are approximate. A brick may have started before the session open and finished after — the session filter uses the brick's timestamp, which is the close time. Volume per brick is the total volume of all ticks that formed that brick, so a brick that took 30 minutes has much more accumulated volume than one that formed in 30 seconds. The expansion/contraction ratios still work for relative comparison but absolute spike thresholds may need adjustment.

---

## Settings Reference — Signals v7.8.0

### Quick Instrument Presets
- **Instrument**: DEFAULT, CL, NQ/MNQ, ES/MES, YM/MYM

### Core Settings
- **EMA Length**: Primary trend EMA (default 34)
- **Show VWAP**: Session VWAP line
- **Show NY Anchored VWAP**: 0930 ET anchored VWAP
- **Show Previous NY VWAP**: Prior session's closing VWAP level

### RSI Configuration
- **RSI Speed**: Standard / Fast / Ultra-Fast presets
- **Short RSI Period**: Fast RSI lookback (default 5)
- **Long RSI Period**: Slow RSI lookback (default 13)
- **Short/Long Smoothing**: EMA smoothing on each RSI
- **RSI Neutral Zone**: Min diff to register direction (default 1.0)

### Stochastic Configuration
- **Stoch Period**: Double Stochastic lookback (default 10)
- **Stoch Smoothing**: Double-EMA smoothing (default 3)
- **Oversold/Overbought Levels**: Threshold levels

### DStoch Crossover Signals
- **Show DStoch Signals**: Toggle DS▲/DS▼ markers
- **Sell/Buy Signal Levels**: Crossover trigger levels
- **Brighten on Confluence**: Ribbon brightens when DStoch and RSI agree

### Volume Filter
- **Volume MA Length**: Baseline SMA period (default 20)
- **Volume Multiplier**: Required ratio above MA for A+ (default 1.1)
- **Enable Volume Filter**: Toggle for A+ volume requirement

### Volume Analysis
- **Enable Volume Analysis**: Master toggle
- **Fast/Slow Vol MA**: Short and long volume MAs (default 5/20)
- **Spike Threshold**: Standard deviations for spike detection (default 2.0)
- **Spike Lookback**: Bars for mean/stdev calculation (default 50)
- **Show Spike Markers**: Toggle V+/V!/V chart markers
- **Spike Colors**: Confirm (green), Exhaust (orange), Neutral (white) — all configurable
- **Show Vol-Price Divergence**: Toggle VD▼/VD▲ flags
- **Divergence Colors**: Bearish (red), Bullish (blue) — configurable
- **Divergence Lookback**: Bars for price vs volume trend comparison (default 10)
- **Dim Ribbon on Contraction**: Ribbon transparency increases when volume contracts

### ADX Trend Strength Filter
- **Enable ADX Filter**: Toggle for A+/A requirement
- **ADX Length**: Lookback period (default 14)
- **ADX Thresholds**: Minimum ADX for A+ and A grades

### MTF Timeframe Selection
- **MTF Speed**: Conservative / Moderate / Aggressive / Custom
- **Custom TFs**: Manual fast/mid/slow timeframe entry (when Custom selected)

### Time & Session Filters
- **Session Config**: Time windows and days

### Algo Windows
- **Algo Window Mode**: OFF / TRADE WITH / AVOID
- **Window toggles**: London, NY Open, NY PM, NY Close, Asia, Globex

### ADR Filter
- **Enable ADR Exhaustion Filter**: Block signals when daily range is exhausted
- **ADR Lookback**: Days for average calculation (default 10)
- **Exhaustion Threshold**: Percentage of ADR used (default 85%)

### Dashboard & Visuals
- **Show Grade Dashboard**: Master toggle
- **Dashboard Position**: Corner selection
- **Color Candles by Grade**: Toggle candle coloring
- **Heikin Ashi Outline**: Grade body + HA direction wicks
- **HA Colors**: Bull/bear outline colors
- **Show Stop/TP Levels**: Toggle SL and TP lines on chart
- **SL Mode**: Fixed / ATR / Wider of Both. Fixed = preset point values. ATR = ATR × multiplier. Wider of Both (default) = uses whichever is larger, giving a fixed floor in normal vol and ATR room in high vol.
- **ATR SL Multiplier**: Multiplier for ATR SL (default 1.5). Only used in ATR and Wider of Both modes.
- **Note**: On grade upgrades, TP adjusts to the median of old and new grade's target distance. This avoids stretching for a target you're already partway to.
- **HA Exit Warning**: Toggle + bars to confirm flip
- **Ribbon Colors**: Bull/bear/consolidation + transparency
- **Ribbon Visible Hours**: Independent time window for ribbon

### Line Colors & Widths
- All EMA, VWAP, SL, TP line colors and widths are individually configurable

### Signal Colors
- All 10 grade signal colors (5 grades × long/short) + text colors are configurable

### Anti-Chop Filters
- **Signal Cooldown**: Bars between signals (default 5)
- **Renko Cooldown**: Seconds between signals on Renko (default 60, min 10)
- **Grade Persistence**: Min bars to hold grade (default 2). Upgrades bypass this — see Grade Upgrade Behavior.
- **ADX Floor for B/B+**: Min ADX for B grades (default 15)
- **Block Counter-Trend**: Prevent B/B+/A- against macro trend
- **Consolidation filters**: Range, lookback, suppression toggles

### RSI Divergence
- **Enable Divergence**: Master toggle
- **Pivot lookback/max bars**: Detection parameters
- **Show Markers/Lines**: Toggle chart visuals
- **Colors/sizes/styles**: All configurable

---

## Settings Reference — Management v7.5.0

### Shared Settings (Must Match Signals)
All core settings (preset, RSI, stochastic, ADX, volume, session) must be identical to the Signals indicator. These are duplicated inputs.

### Additional EMAs
- **Show 21/50/233 EMA**: Toggle additional EMA overlays

### ADX Advanced
- **Dynamic Sizing by ADX**: Scale position size with ADX strength
- **Runner ADX Gate**: Minimum ADX to qualify for runner
- **Pyramid Only ADX Rising**: Only add when ADX is increasing
- **Breakout Detection**: Alert when ADX crosses from low to high
- **Exit Runner ADX Below**: Close runner when ADX drops below threshold

### ADR Dashboard
- **Show ADR on Dashboard**: Toggle remaining ADR display

### Globex Levels
- **Show Previous Globex High/Low**: 2 days of prior session H/L
- **Show Globex POC**: Prior session Point of Control (volume-based)
- **Colors and line styles**: All configurable

### Dashboard & Management
- **Show Trade Context Panel**: Master toggle for bottom-left panel
- **Show Volatility/Sizing Dashboard**: Master toggle for sizing panel
- **Show ATR Stop Reference**: Toggle ATR-based stop crosses (A grades)
- **ATR TP Multiplier**: Multiplier for TP2 runner target (default 2.0)
- **ATR Period**: Lookback for ATR (default 14)
- **Max Pain ($)**: Maximum dollar loss per trade (default $400)
- **Max Contracts Cap**: Hard ceiling on suggested contracts (default 10)
- **Contract Size**: Micro or Mini/Standard
- **Sizing Dashboard Position**: Corner selection

### Runner Trail Stop
- **Show Runner Trail**: Toggle trail stop line
- **Trail Source**: 34 EMA / 21 HMA / 21 EMA

### Pyramid Adds
- **Enable Pyramid**: Toggle add signals
- **Max Adds Per Trend**: Limit (default 2)
- **Pyramid EMA Source**: Runner Trail / 34 EMA / Dual Confirm
- **Pullback Zone (%)**: How close to EMA for qualification
- **Require Bounce**: Must close back in trend direction

---

## Alert Reference

### Signals v7.8.0 — 16 Alerts

| Alert | Fires When |
|-------|------------|
| 🔥 A+ LONG | A+ long signal confirmed |
| 🔥 A+ SHORT | A+ short signal confirmed |
| ⚡ A LONG | A long signal confirmed |
| ⚡ A SHORT | A short signal confirmed |
| ✓ A- LONG | A- long signal confirmed |
| ✓ A- SHORT | A- short signal confirmed |
| 💨 B+ LONG SCALP | B+ long scalp confirmed |
| 💨 B+ SHORT SCALP | B+ short scalp confirmed |
| ⚡ B LONG SCALP | B long scalp confirmed |
| ⚡ B SHORT SCALP | B short scalp confirmed |
| 🎯 TP HIT | Price crosses locked TP level (once per trade) |
| 🛑 SL HIT | Price crosses locked SL level (once per trade) |
| ⚠ HA MOMENTUM FLIP | Heikin Ashi flips against position |
| 📊 REGIME CHANGE | Consolidation regime shifts |
| 🔀 RSI DIVERGENCE | Price-RSI divergence detected |
| 📊 VOLUME SPIKE | Volume exceeds 2σ threshold |

### Management v7.5.0 — 3 Alerts

| Alert | Fires When |
|-------|------------|
| 📈 PYRAMID ADD LONG | Grade upgrade or EMA pullback (long) |
| 📈 PYRAMID ADD SHORT | Grade upgrade or EMA pullback (short) |
| 💥 ADX BREAKOUT | ADX crosses from low to high range |

See `Alert_Setup_Guide.md` for step-by-step TradingView alert configuration and full trade examples.
