# EMA River Strategy

A multi-confluence futures trading strategy for YM, NQ, ES, CL, HG.

## Version
- **Current**: v1.1.4

## Strategy Overview

### Entry Conditions

**LONG Entry** (all must be true):
- RSI Ribbon = Bullish (green)
- 9 EMA = Bullish (price above)
- 20 HMA = Bullish (price above)
- 20 EMA High = Bullish (price above)

**SHORT Entry** (all must be true):
- RSI Ribbon = Bearish (red)
- 9 EMA = Bearish (price below)
- 20 HMA = Bearish (price below)
- 20 EMA Low = Bearish (price below)

### Exit Conditions
- **Long Exit**: Fast HMA (6) crosses below Slow HMA (21)
- **Short Exit**: Fast HMA (6) crosses above Slow HMA (21)
- Plus TP/SL/BE/Trail per contract settings

### Filters
- **BB Chop Filter**: No trades when price inside BB bands (3min MTF)
- **T-Line 1**: No trades when price inside 20 EMA ± 34 points, 15s cooldown
- **T-Line 2**: No trades when price inside 50 EMA ± 80 points, 15s cooldown
- **Session Filter**: Trade only during configured sessions

## Core Components

| Component | Setting |
|-----------|---------|
| 9 EMA | OHLC4 |
| 20 EMA River | High / HL2 / Low |
| 20 HMA | Close |
| RSI Ribbon | 5/13 RSI diff |
| HMA Exit Ribbon | 6 HMA / 21 HMA |
| T-Line 1 | 20 EMA ± 34 pts |
| T-Line 2 | 50 EMA ± 80 pts |

## Session Windows

Single string input format: `TIME1,TIME2,TIME3:DAYS`

**Default**: `1801-1940,0700-0815,0930-1100,1545-1600:23456`

| Window | Time (ET) |
|--------|-----------|
| Globex Evening | 18:01–19:40 |
| Pre-Market | 07:00–08:15 |
| NY AM | 09:30–11:00 |
| NY Close | 15:45–16:00 |

Days: 1=Sun, 2=Mon, 3=Tue, 4=Wed, 5=Thu, 6=Fri, 7=Sat

## Position Management (Multi-Contract)

### Contract 1 (Scalp)
| Parameter | Default |
|-----------|---------|
| Take Profit | 50 ticks |
| Stop Loss | 40 ticks |
| BE Trigger | 30 ticks profit |
| BE Offset | +10 ticks |
| Trail | OFF |

### Contract 2 (Runner)
| Parameter | Default |
|-----------|---------|
| Take Profit | 125 ticks |
| Stop Loss | 40 ticks |
| BE Trigger | 30 ticks profit |
| BE Offset | +10 ticks |
| Trail | ON (20 tick stop after 30 profit) |

Toggle `Trade 2 Contracts` to enable/disable second contract.

## Signal Arrows

| Arrow | Color | Meaning |
|-------|-------|---------|
| L | Lime | Long Entry |
| S | Red | Short Entry |
| DS▲ | Aqua | DStoch Buy Signal |
| DS▼ | Fuchsia | DStoch Sell Signal |

## Changelog

### v1.1.4
- **Comprehensive tooltips** on all inputs explaining purpose and default values
- **Section header tooltips** explaining what each group of settings does
- **Full style customization** for all chart elements:
  - Colors (bullish/bearish) for all MAs and channels
  - Line widths (1-5) for all plotted lines
  - Line styles (Solid/Dashed/Dotted) where applicable
  - Fill colors for ribbons and channels
- **Second T-Line Channel** (50 EMA / 80pt width) for broader consolidation filtering
- T-Line 1 (20/34) now uses yellow color scheme
- T-Line 2 (50/80) uses orange color scheme for differentiation
- Entry signal arrow colors now customizable
- DStoch marker colors now customizable

### v1.1.3
- T-Line Channel Chop Filter — No entries when price inside 20 EMA ± 34 points
- 15-second cooldown after price exits channel
- Yellow channel fill visualization
- Debug table shows T-Line status

### v1.1.2
- HMA Cross Exit: Exit signals based on 6/21 HMA crossover
- HMA Exit Ribbon visual with fill

### v1.1.1
- Session filter changed to single string input
- Supports up to 6 time windows

### v1.1.0 — MAJOR REWRITE
- Simplified entry logic: RSI + 9 EMA + 20 HMA + 20 EMA confluence
- New exit condition: Close vs 20 EMA HL2
- Removed legacy complexity

### v1.0.x
- Initial development and debugging
