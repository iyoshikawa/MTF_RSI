# EMA River Strategy

A GJBI-style futures trading strategy for YM, NQ, ES, CL, HG.

## Version
- **Current**: v1.2.3

## Strategy Overview

This strategy combines:
- **River-Based Background**: Price position relative to 20 EMA River
- **HMA Cross Ribbon**: 6/21 HMA crossover for momentum confirmation

Background is only painted when **BOTH** indicators agree on direction.

### Background Logic

| Price Position | Background |
|----------------|------------|
| **Above 20 EMA High** | Green (if HMA agrees) |
| **Below 20 EMA Low** | Red (if HMA agrees) |
| **Inside River** | Neutral (always) |

### Entry Conditions

**LONG Entry** (all must be true):
- Price ABOVE 20 EMA High (background bullish)
- 6 HMA > 21 HMA (ribbon bullish)
- River crossing condition met (Strict or Loose)
- In trading session
- Not in chop zone

**SHORT Entry** (all must be true):
- Price BELOW 20 EMA Low (background bearish)
- 6 HMA < 21 HMA (ribbon bearish)
- River crossing condition met (Strict or Loose)
- In trading session
- Not in chop zone

### River Entry Modes

| Mode | Logic | Best For |
|------|-------|----------|
| **Strict** | Price must cross FROM outside INTO river | Catching fresh momentum after pullback |
| **Loose** | Price just needs to be in or past river | More signals, may enter mid-move |

**Strict Math (Long):** `close > river_low AND close[1] <= river_low[1]`
**Loose Math (Long):** `close > river_low`

### Exit Modes

| Mode | Logic |
|------|-------|
| **Previous Bar HL** | Stop at previous bar low (longs) or high (shorts) |
| **HMA Cross** | Exit when 6/21 HMA crosses against position |
| **Both** | Exit on whichever happens first |

## Core Components

| Component | Setting | Purpose |
|-----------|---------|---------|
| 20 EMA River | High / HL2 / Low | Entry zone + Background |
| 6 HMA | Close | Fast ribbon |
| 21 HMA | Close | Slow ribbon |

## Filters

| Filter | Default | Purpose |
|--------|---------|---------|
| Session | ON | Trade only during configured windows |
| BB Chop | OFF | Block entries inside BB bands |
| T-Line | OFF | Block entries inside T-Line channel |

## Session Windows

Default: `1801-1940,0700-0815,0930-1100,1545-1600:23456`

| Window | Time (ET) |
|--------|-----------|
| Globex Evening | 18:01–19:40 |
| Pre-Market | 07:00–08:15 |
| NY AM | 09:30–11:00 |
| NY Close | 15:45–16:00 |

## Position Management

Single contract with configurable TP/SL:
- Take Profit: 50 ticks (default)
- Stop Loss: 40 ticks (default)

## Changelog

### v1.2.3
- **QuantLynk Integration** for automated order routing
  - Enable/disable toggle (default: OFF)
  - User ID and Alert ID inputs
  - OSO bracket orders (market + TP/SL attached)
  - Flatten payload for HMA cross exits
  - Quantity input for contract sizing

### v1.2.2
- **Bright Highlight** with EMA confluence
  - Brighter background when price past 20 EMA HL2 + above/below 9/21/50 EMAs
  - Sustained highlight (not just crossing bar)
  - 9/21/50 EMA confluence inputs
  - Optional confluence EMA plots

### v1.2.1
- **River-based background**: Now uses 20 EMA High/Low instead of 50 EMA
  - Green: Price > 20 EMA High (+ HMA agrees)
  - Red: Price < 20 EMA Low (+ HMA agrees)
  - Neutral: Price inside river (always)
- Removed 50 EMA calculation and plot
- Debug table shows "River BG" with ABOVE/IN RIVER/BELOW status

### v1.2.0 — MAJOR SIMPLIFICATION
- GJBI-style strategy based on Geo-TX methodology
- Combined background: Only paints when indicators agree
- River entry modes: Strict (crossover) or Loose (in river) dropdown
- Exit modes: Previous Bar HL, HMA Cross, or Both dropdown
- Single contract: Removed multi-contract complexity
- Filters simplified: BB and T-Line OFF by default, Session ON

### v1.1.x
- Previous iterative development (see git history)

## QuantLynk Automation

### Setup
1. Enable "QuantLynk Alerts" in strategy settings
2. Enter your `User ID` and `Alert ID` from QuantLynk dashboard
3. Configure `Quantity` (contracts)
4. Choose `Use OSO Bracket` (ON = broker-managed TP/SL)

### TradingView Alert Setup
Set alert message to:
```
{{strategy.order.alert_message}}
```

### Order Types

| Scenario | Payload |
|----------|---------|
| **Entry (OSO ON)** | Market order + TP/SL bracket attached |
| **Entry (OSO OFF)** | Market order only |
| **HMA Cross Exit** | Flatten (closes all positions) |

### TP/SL Calculation
- TP/SL prices calculated from strategy's `Take Profit (ticks)` and `Stop Loss (ticks)` inputs
- Sent as fixed prices in the OSO payload
