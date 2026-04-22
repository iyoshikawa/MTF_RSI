# ORB-Donchian Retest Indicator — SPEC

**Version:** 1.1  
**Branch:** `feature/orb-donchian-retest`  
**File:** `ORB_Donchian_Retest/ORB_Donchian_Retest_v1_1.pine`  
**Chart:** 1-minute standard candlestick  
**Instruments:** YM, NQ, ES, CL, HG (and micros)  

---

## Origin

Based on Robert Anderson's multi-part futures strategy playlist:
- [Strategy Training Videos](https://youtube.com/playlist?list=PLir78uvI10JyqmrxEYhnJjXIuu7bVnTFf)
- [Open Box Range Trades (corrected)](https://youtu.be/fon4F6bZZbA)

Bob's exact NinjaTrader indicator stack (from video description):
1. Bar Timer
2. Current Day OHL
3. EMA 9
4. SMA 105
5. Donchian Channel 60
6. Swing Indicator period 5
7. PriorDay OHLC

All on a 1-minute chart.

---

## Architecture

### Layer 1 — Visual Levels (chart overlay)

| Component | Implementation | Configurable |
|-----------|---------------|--------------|
| ORB Box | `box.new()` shaded rectangle, 0930–0945 ET default | Start/end hour+minute, fill/border color, extend right |
| ORB High / Low / Gold Line | `plot.style_linebr` persistent after lock | Show toggles, colors |
| 9 EMA | `ta.ema(close, 9)` | Length, color, show toggle |
| 45 EMA | `ta.ema(close, 45)` | Length, color, show toggle |
| 55 EMA (T-Line) | `ta.ema(close, 55)` + ATR/fixed envelope band | Length, width type, ATR mult, fixed pts, band opacity |
| 105 SMA | `ta.sma(close, 105)` | Length, color, show toggle |
| 233 EMA | `ta.ema(close, 233)` | Length, color, show toggle |
| Donchian Channel (60) | `ta.highest/ta.lowest` + midline + fill | Period, colors, fill opacity, mid toggle |
| VWAP | `ta.vwap(hlc3)` | Color, show toggle |
| Prior Day H/L/O/C | Tracked via `is_new_day` detection | Individual show toggles, colors |
| Current Day O/H/L | Running high/low, open on new day | Individual show toggles, colors |
| Swing Pivots (period 5) | `ta.pivothigh/ta.pivotlow` with SH/SL labels | Period, label toggle, colors |

### Layer 2 — Signal Detection Engine

| Signal | Logic | Type |
|--------|-------|------|
| ORB Breakout | First candle close beyond ORB H/L after window locks | One-shot pulse, sets `orb_bias` |
| Cross-the-Box | Price broke one side, then closes beyond opposite side | One-shot pulse, **flips** `orb_bias` |
| ORB Breakout Candle | `barcolor` white on breakout bar | Visual only |
| CTB Candle | `barcolor` purple + diamond "CTB" label | Visual + pulse plot |
| Donchian Break | Close crosses prior-bar DC upper/lower | Pulse, sets `dc_break_dir` |
| Donchian Retest | Price returns to broken DC level within lookback window, bounces | Pulse within N bars of break |
| EMA Retest | Price touches 9/45/55 EMA zone (within tolerance) + bounces with directional close | Pulse |
| Candlestick Patterns | Bullish/Bearish Engulfing, Hammer, Shooting Star, Pin Bar | Pulse, labeled on chart |
| MA Alignment | 9 > 45 > 55 (bull stack) or 9 < 45 < 55 (bear stack) | Held state |
| 233 EMA Trend | Close above/below 233 EMA | Held state |
| VWAP Filter | Close above/below VWAP | Held state |
| Swing Structure | HH+HL (bull) or LH+LL (bear) from last two pivot pairs | Held state |
| Swing Break | Close crosses prior pivot high/low | Pulse |

### Layer 3 — Confluence Scoring + Dashboard

10 toggleable gates, each +1 to score:

| # | Gate | Signal Type |
|---|------|-------------|
| 1 | ORB Break | Held (bias) |
| 2 | DC Break | Pulse within lookback |
| 3 | DC Retest | Pulse |
| 4 | EMA Retest | Pulse |
| 5 | Candle Conf | Pulse |
| 6 | VWAP Filter | Held |
| 7 | MA Stack | Held |
| 8 | 233 Trend | Held |
| 9 | Swing Struct | Held |
| 10 | Cross-the-Box | Held (fired state) |

- Min confluence configurable (default 5/10)
- Signal fires when score >= min AND inside active session window
- Dashboard: 16-row compact overlay table, 8-position placement, configurable size/opacity

### Layer 4 — Session Filter

| Window | Default |
|--------|---------|
| Window 1 | 0930–1145 ET |
| Window 2 | 1330–1600 ET |
| Enable toggle | ON by default |

### Layer 5 — Gate + Trigger Outputs (Galgoom)

All `display.none` plots:

| Plot Name | Type | Logic |
|-----------|------|-------|
| Bull Gate | Held | ORB bias ≥ 0 + MA stack bull + above VWAP + 233 bull + swing bull |
| Bear Gate | Held | ORB bias ≤ 0 + MA stack bear + below VWAP + 233 bear + swing bear |
| Bull Trigger | Pulse | Full confluence + (DC retest OR EMA retest) + candle confirmation |
| Bear Trigger | Pulse | Full confluence + (DC retest OR EMA retest) + candle confirmation |
| ORB Bull/Bear | Held | ORB bias direction |
| DC Retest Bull/Bear | Pulse | Donchian retest events |
| EMA Retest Bull/Bear | Pulse | EMA touch + bounce events |
| Candle Bull/Bear | Pulse | Pattern detected |
| Swing Bull/Bear | Held | Structure direction |
| Cross Box Bull/Bear | Pulse | CTB fire event |
| Cross Box Bull/Bear State | Held | CTB has fired today |
| Bull/Bear Score | Value | Integer confluence count |

**Galgoom Wiring:**
```
Buy  Gate Source   → "Bull Gate"     | Open Mode: Rise > 0
Buy  Trigger Src  → "Bull Trigger"  | Trigger Pulse: Rise > 0
Sell Gate Source   → "Bear Gate"     | Open Mode: Rise > 0
Sell Trigger Src  → "Bear Trigger"  | Trigger Pulse: Rise > 0
```

---

## Alert Conditions (10 total)

1. Bull Confluence Signal
2. Bear Confluence Signal
3. ORB Bullish Breakout
4. ORB Bearish Breakout
5. Cross-the-Box Bull
6. Cross-the-Box Bear
7. Donchian Bull Retest
8. Donchian Bear Retest
9. Swing High Break
10. Swing Low Break

---

## Cross-the-Box Logic Detail

The Cross-the-Box (CTB) is a **failed breakout reversal**:

1. ORB window locks at 0945 ET (default)
2. Price breaks below ORB Low → `orb_break_bear_fired = true`, `orb_bias = -1`
3. Price reverses, rallies through the entire box, closes above ORB High
4. → `orb_cross_bull = true` (pulse), `orb_cross_bull_fired = true` (held)
5. → `orb_bias` **flips to +1** — the cross overrides the original break
6. Candle colored purple, diamond "CTB" marker plotted
7. One-shot per direction per day

The inverse applies for bearish CTB (broke above first, then crosses below).

**Why it matters:** Trapped breakout traders who entered on the initial break are now offside. Their stop-outs and forced exits become fuel for the reversal move. This is one of Bob's highest-conviction setups.

---

## Retest Detection Parameters

| Parameter | Default | Purpose |
|-----------|---------|---------|
| Retest Lookback | 10 bars | How many bars after DC break to watch for retest |
| Retest Tolerance | 3.0 pts | How close price must come to broken DC level |
| EMA Retest Tolerance | 3.0 pts | How close price must come to an EMA |

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-04-22 | Initial build: ORB + Donchian + EMA retest + candle patterns + confluence dashboard + Gate/Trigger plots |
| 1.1 | 2026-04-22 | Added: Swing Indicator (period 5), Prior Day Close, ORB Box visual, Cross-the-Box detection + gate + alertconditions |

---

## Future Considerations

- Convert to `strategy()` for backtesting once indicator logic validated on live charts
- QuantLynk webhook integration (requires strategy conversion)
- VEG (Volume Efficiency Gate) integration after VEG standalone build completes
- Bond Market Dashboard cross-reference as additional gate
- Q_Range day type filter integration (deferred pending screen time)
- Instrument-specific parameter presets (YM tighter tolerance, CL wider, etc.)
