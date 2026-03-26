# Volume Exhaustion Strategy (VES) — Architecture Document v2
## Reverse-engineered from "The SSS" behavior — ALL ANSWERS CONFIRMED

---

## STRATEGY TYPE
- **PineScript v5 `strategy()`** with backtesting
- **Mean-reversion** on volume exhaustion
- **Instruments:** YM, NQ, ES, CL, HG futures
- **Chart:** Renko Traditional 8-tick (must work on any base timeframe down to 1s)
- **Timeframe-agnostic:** No hardcoded bar-count assumptions. All timing is wall-clock based.

---

## ENTRY LOGIC

### Core Signal: Volume Exhaustion Detection
- **VOL BOTTOM (VEB)** → LONG entry (volume exhausted at swing low, expect bounce)
- **VOL PEAK (VEP)** → SHORT entry (volume exhausted at swing high, expect drop)
- Entry is **immediate** on signal bar — no confirmation wait
- Signals are **symmetrical** (same logic mirrored for long/short)
- Both D:1/2 and D:2/2 can fire on the same bar or very close together

### Volume Exhaustion Detection — Starting Hypothesis
Z-score based detection (to be calibrated):
1. Volume baseline: SMA over N bars
2. Volume std dev over same period
3. Z-score = (current_volume - mean) / std_dev
4. **VOL PEAK:** High Z-score + price at local high (Renko flip from bull→bear)
5. **VOL BOTTOM:** High Z-score + price at local low (Renko flip from bear→bull)

Z-score threshold: configurable (default ~1.5-2.0), displayed on entry labels.

### Swing Detection (Renko-optimized)
- Primary: Renko color flip (brick direction change)
- A bearish brick after bullish run = potential VOL PEAK zone
- A bullish brick after bearish run = potential VOL BOTTOM zone
- Volume spike on or near the flip confirms exhaustion

---

## TREND FILTER (Soft — NOT a hard block)

### Moving Averages (ALL on OHLC4 source)
| MA | Length | Purpose |
|----|--------|---------|
| EMA | 9 | Fast — immediate momentum |
| EMA | 15 | Medium — short-term trend |
| EMA | 34 | Primary — trend direction |
| EMA | 233 | Macro — strong trend anchor (Fibonacci) |

### Soft Filter Behavior
- Signals CAN fire counter-trend
- **Suppress** (or flag lower confidence) when:
  - VOL BOTTOM long AND price below 233 EMA AND 9 < 15 < 34 (all falling)
  - VOL PEAK short AND price above 233 EMA AND 9 > 15 > 34 (all rising)
- Goal: avoid catching falling knives / shorting rockets
- TP and trail protect against being wrong

---

## POSITION MANAGEMENT

### Scaling
- **`pyramiding = 2`** — max 2 independent positions, same direction
- Each enters on its own VOL signal (can be same bar or different bars)
- D:1/2 = first position, D:2/2 = second position
- Each tracks its own entry price, TP level, and trail level independently

### Exit Rules (Per Position)
- **Take Profit:** 40 points from that position's entry price (configurable)
- **Trailing Stop:** 40 points from that position's best price (configurable)
- **NO fixed stop loss** — only TP and trail
- Whichever hits first closes that specific position

### Opposite Signal Behavior
- **Flatten and reverse:** Close ALL open positions, then enter new direction
- Example: Long D:1/2 + D:2/2 open → VOL PEAK fires → close both longs → enter short D:1/2

---

## VISUAL ELEMENTS

### Chart Overlays
- EMA 9 (OHLC4) — colored by trend direction
- EMA 15 (OHLC4) — colored by trend direction
- EMA 34 (OHLC4) — colored by trend direction
- EMA 233 (OHLC4) — macro reference line
- Yellow dots on wick touches of MA (visual only, NOT entry condition)
- Colored candles (blue/cyan = bullish, red/pink = bearish)

### Signal Labels
- "▼ VOL PEAK" (orange, above bar) — short entry signal
- "▲ VOL BOTTOM" (green, below bar) — long entry signal
- Entry detail label: "VEB Z:X.X D:N/2 TP:XXXXX TRL-40"

### Dashboard (table, top-right)
| Row | Content |
|-----|---------|
| TP/TRAIL | "40pt TP  40pt TRAIL" (configurable) |
| SIGNAL | "scanning" / "VEP" / "VEB" |
| Position | "FLAT" / "LONG D:1/2" / "LONG D:2/2" / etc |
| Trail Level | Live trailing stop price per position |
| Win Rate | % with W:xx L:xx breakdown |
| Closed Trades | Total count |
| Net P&L | Dollar value |
| Profit Factor | Gross profit / gross loss |
| Expectancy | $/trade |
| Avg W/L | Average win $ / Average loss $ |
| Max DD | strategy.max_drawdown |
| Daily DD | Custom tracked, reset daily |
| Session | Active window or "OUTSIDE" |

---

## SESSION FILTER

### Trading Windows (ET, configurable)
- London: 0200-0500
- NY Pre-market: 0830-0930
- NY AM: 0930-1145
- NY PM: 1330-1600
- Configurable string format (same as current MTF RSI approach)
- Must work across all base timeframes (1s to 30s+)

---

## PERFORMANCE TRACKING (strategy.* built-ins)

| Stat | Source |
|------|--------|
| Win Rate | strategy.wintrades / strategy.closedtrades × 100 |
| Closed Trades | strategy.closedtrades |
| Net P&L | strategy.netprofit |
| Profit Factor | strategy.grossprofit / strategy.grossloss |
| Expectancy | strategy.netprofit / strategy.closedtrades |
| Avg Win | strategy.grossprofit / strategy.wintrades |
| Avg Loss | strategy.grossloss / strategy.losstrades |
| Max DD | strategy.max_drawdown |

---

## BUILD PHASES

### Phase 1: Core Engine (BUILD FIRST)
- [ ] strategy() shell with pyramiding=2, proper commission/slippage
- [ ] EMAs on OHLC4 (9, 15, 34, 233) with colored plots
- [ ] Volume exhaustion detection (Z-score + Renko flip)
- [ ] VOL PEAK / VOL BOTTOM signal labels
- [ ] Entry logic: immediate on signal, 2 independent positions
- [ ] Exit logic: 40pt TP + 40pt Trail per position, flatten on opposite signal
- [ ] Session time filter
- [ ] Performance dashboard with all stats
- [ ] Colored candles

### Phase 2: Refinement
- [ ] Soft trend filter (suppress counter-trend in strong moves)
- [ ] Z-score calibration per instrument (presets for YM/NQ/ES/CL/HG)
- [ ] Yellow dot MA wick-touch markers
- [ ] Daily DD limit with auto-flatten
- [ ] Entry detail labels (Z:X.X D:N/2 TP:XXXX TRL-XX)
- [ ] Instrument presets dropdown

### Phase 3: Automation
- [ ] QuantVue ATS webhook payloads
- [ ] QuantLynk payload generation
- [ ] Alert conditions for all entry/exit events

---

## KEY DESIGN DECISIONS
1. **Wall-clock timing only** — no bar-count cooldowns (Renko bars are irregular)
2. **OHLC4 source for all EMAs** — smooths wick noise on Renko
3. **No fixed SL** — TP and trail are the only exits (plus flatten on reversal)
4. **Independent position tracking** — each of 2 positions manages its own levels
5. **Flatten-and-reverse** — opposite signal closes everything, then enters new direction
