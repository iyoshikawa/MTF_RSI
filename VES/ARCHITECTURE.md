# Volume Exhaustion Strategy (VES) — Architecture Document v4
## Updated for VES v2.0.3

---

## STRATEGY TYPE
- **PineScript v6 `strategy()`** with backtesting
- **Mean-reversion** on volume exhaustion
- **Instruments:** YM, NQ, ES, CL, HG futures (full + micro)
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

### Volume Exhaustion Detection — Wave Compare (Primary for Renko)
1. Accumulate volume per directional wave (consecutive same-color bricks)
2. On Renko flip, compare ending wave's volume to prior same-direction wave
3. **Wave Decline:** Ending wave had less volume → side is tiring out
4. **Wave Climax:** Ending wave had more volume → blowoff, nobody left

### Swing Detection (Renko-optimized)
- Primary: Renko color flip (brick direction change)
- A bearish brick after bullish run = potential VOL PEAK zone
- A bullish brick after bearish run = potential VOL BOTTOM zone
- Volume exhaustion on or near the flip confirms signal

---

## GATE CHAIN (v2.0.3)

All active gates must pass for a signal to fire. Gates marked `req_*` are controlled by the 🎯 Confluence Requirements input group.

| Gate | Control | Default | What It Does |
|------|---------|---------|-------------|
| Min Wave Length | Always on | Required | Prior wave ≥ N bricks (default 3) |
| Renko Flip | Always on | Required | Brick direction changed |
| Volume Exhaustion | req_V | ON | Wave Compare / ROC / Z-Score / OFF |
| BB Outside + Squeeze | req_B | ON | Price at band edge + width above adaptive threshold |
| HMA Gate (HMA×EMA21) | Trend Filter toggle | ON | HMA 20 above/below EMA 21 — momentum direction |
| Hpx Confluence | req_H | OFF | Price position vs HMA 20 — mean-reversion side |
| EMA Stack | req_E | OFF | Full 21/34/50 alignment in signal direction |
| Cross-Wave Ratio | req_W | OFF | Counter-wave ≤ threshold % of dominant wave |
| Session Window | req_S | ON | Inside configured trading hours |
| ADR Exhaustion | Toggleable | ON | Day's range < 80% of ADR |
| Chop Detector | Toggleable | ON | Avg of last 5 waves ≥ 5 bricks |
| Daily Limits | Always on | Required | DD and PT not hit |

---

## TREND SYSTEM (v2.0.3)

### Moving Averages (ALL on OHLC4 source)
| MA | Length | Purpose |
|----|--------|---------|
| EMA | 21 | Fast — trend structure, HMA cross reference |
| EMA | 34 | Mid — alignment confirmation |
| EMA | 50 | Slow — trend anchor (replaces old 233 macro) |
| HMA | 20 | Dual role: HMA Gate (vs EMA 21) + Hpx confluence (vs price) |

### HMA Dual Role (v2.0.3 — clarified)
1. **HMA Gate:** HMA 20 crossing EMA 21. HMA above = bullish momentum → shorts blocked. HMA below = bearish momentum → longs blocked. Controlled by Trend Filter toggle.
2. **Hpx Confluence:** Price position vs HMA 20. Close below HMA = +1 long confluence (mean-reversion into bearish structure). Close above HMA = +1 short confluence. Optionally required via `req_H`.

### Three-State Trend Read
1. **Trend Intact:** 21/34/50 stacked + HMA on right side of EMA 21 → trade with trend
2. **Trend Warning:** HMA crosses through EMA 21 → stop taking new signals in losing direction
3. **Trend Broken:** EMAs unstack → no trend, wait for new structure

### Cross-Wave Volume Ratio (Confluence)
- Only evaluates when 21/34/50 is stacked
- Compares counter-trend wave volume to dominant wave volume
- If counter-wave ≤ 50% of dominant wave → pop/dip was hollow → +1 confluence
- Confirms the dominant side still controls the market

---

## CHOP DETECTION (v2.0.2)

### Wave Length Chop Detector
- Tracks the last N completed wave lengths in a rolling array
- Averages them. If average < threshold, market is flip-heavy chop
- All signals blocked until average rises above threshold
- Default: last 5 waves, minimum average 5 bricks

### Adaptive BB Squeeze
- Compares current BB width to its own 50-bar moving average
- If width < 60% of average → squeeze → signals blocked
- Auto-calibrates across sessions and instruments
- No manual tuning needed between London/NY/Asia

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
- **Breakeven Stop:** After N bricks in favor, stop moves to entry + offset ticks
- **NO fixed stop loss** — only TP, trail, BE
- Whichever hits first closes that specific position

### Opposite Signal Behavior
- **Flatten and reverse:** Close ALL open positions, then enter new direction

---

## 7 CONFLUENCES (v2.0.3)

Informational scoring — each factor can be promoted to a **required gate** via the 🎯 Confluence Requirements group.

| # | Code | Factor | req_* default |
|---|------|--------|---------------|
| 1 | F | Renko Flip | Always required |
| 2 | V | Volume Exhaustion | req_V = ON |
| 3 | B | BB Outside Bands | req_B = ON |
| 4 | S | Session Active | req_S = ON |
| 5 | Hpx | HMA Position (price vs HMA) | req_H = OFF |
| 6 | E | EMA 21/34/50 Alignment | req_E = OFF |
| 7 | W | Cross-Wave Volume Ratio | req_W = OFF |

---

## VISUAL ELEMENTS

### Chart Overlays
- EMA 21 (OHLC4) — white
- EMA 34 (OHLC4) — gold
- EMA 50 (OHLC4) — cyan, thicker
- HMA 20 (OHLC4) — lime green, thicker
- Bollinger Bands (MTF) — blue with fill
- Yellow dots on wick touches of selected EMA
- Colored candles (blue/cyan = bullish, pink = bearish)

### Signal Labels
- "VEB [method] [wave_len]b [conf]/7" (green, below bar)
- "VEP [method] [wave_len]b [conf]/7" (orange, above bar)
- Tooltip shows all confluence pass/fail: F V BB S H E W

### Dashboard (19 rows)
Full status panel with signal state, position, session, P&L, all gates, all confluences, chop status, trend gate, EMA stack, cross-wave ratio, and performance stats.

---

## SESSION FILTER

### Trading Windows (ET, configurable)
- Default: 0930-1145, 1330-1600
- CL: 0900-1430
- Supports up to 4 comma-separated windows
- Configurable day filter (Mon-Fri default)

---

## KEY DESIGN DECISIONS
1. **Wall-clock timing only** — no bar-count cooldowns (Renko bars are irregular)
2. **OHLC4 source for all EMAs + HMA** — consistent basis, smooths wick noise
3. **No fixed SL** — TP, trail, and BE are the only exits (plus flatten on reversal)
4. **Independent position tracking** — each of 2 positions manages its own levels
5. **Flatten-and-reverse** — opposite signal closes everything, then enters new direction
6. **Adaptive thresholds** — BB squeeze and chop detector self-calibrate, no manual session tuning
7. **Gates vs Confluences** — gates block signals (binary), confluences score quality (additive)
8. **req_* system** — any confluence factor can be promoted to a required gate without touching debug toggles. V/B/S default ON (preserve prior behavior), H/E/W default OFF (informational)
9. **Renko-native trail activation** — `trail_activation_bricks` maps trail start to actual brick moves rather than a hardcoded 1-tick trigger
10. **ADR uses prior day range** — `daily_high/low[1]` with `lookahead_on` prevents intraday bar updates from repainting ADR calculations
