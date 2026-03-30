# EMA River Strategy

A GJBI-style futures trading strategy for YM, NQ, ES, CL, HG.

## Version
- **Current**: v1.2.0

## Strategy Overview

This strategy combines two GJBI indicators:
- **GJBIEMABG**: Background color based on price vs 50 EMA
- **GJBIHMACrossRibbon**: 6/21 HMA crossover ribbon

Background is only painted when **BOTH** indicators agree on direction.

### Entry Conditions

**LONG Entry** (all must be true):
- Price ABOVE 50 EMA (background bullish)
- 6 HMA > 21 HMA (ribbon bullish)
- River crossing condition met (Strict or Loose)
- In trading session
- Not in chop zone

**SHORT Entry** (all must be true):
- Price BELOW 50 EMA (background bearish)
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
| 50 EMA | Close | Background trend (GJBIEMABG) |
| 20 EMA River | High / HL2 / Low | Entry zone |
| 6 HMA | Close | Fast ribbon (GJBIHMACrossRibbon) |
| 21 HMA | Close | Slow ribbon (GJBIHMACrossRibbon) |

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

### v1.2.0 — MAJOR SIMPLIFICATION
- **GJBI-style strategy** based on Geo-TX methodology
- **Combined background**: Only paints when 50 EMA AND HMA ribbon agree
- **River entry modes**: Strict (crossover) or Loose (in river) dropdown
- **Exit modes**: Previous Bar HL, HMA Cross, or Both dropdown
- **Single contract**: Removed multi-contract complexity
- **Filters simplified**: BB and T-Line OFF by default, Session ON
- Removed RSI Ribbon, Double Stochastic, and other complexity

### v1.1.x
- Previous iterative development (see git history)

### v1.0.x
- Initial development
