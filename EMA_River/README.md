# EMA River Strategy

A multi-confluence futures trading strategy for YM, NQ, ES, CL, HG.

## Version
- **Current**: v1.1.2

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
- **Session Filter**: Trade only during configured sessions

## Core Components

| Component | Setting |
|-----------|---------|
| 9 EMA | OHLC4 |
| 20 EMA River | High / HL2 / Low |
| 20 HMA | Close |
| RSI Ribbon | 5/13 RSI diff |
| HMA Exit Ribbon | 6 HMA / 21 HMA |

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

### v1.1.2
- **HMA Cross Exit**: Exit signals now based on 6/21 HMA crossover (faster exit on reversals)
- Long exit: Fast HMA crosses below Slow HMA
- Short exit: Fast HMA crosses above Slow HMA
- HMA Exit Ribbon visual with fill between fast/slow HMA
- Debug table shows HMA Exit status

### v1.1.1
- **Session filter now uses single string input**
- Format: `TIME1,TIME2,TIME3:DAYS`
- Example: `1801-1940,0700-0815,0930-1100,1545-1600:23456`
- Days: 1=Sun, 2=Mon, 3=Tue, 4=Wed, 5=Thu, 6=Fri, 7=Sat
- Supports up to 6 time windows

### v1.1.0 — MAJOR REWRITE
- **Simplified entry logic**: RSI + 9 EMA + 20 HMA + 20 EMA confluence
- **New exit condition**: Close vs 20 EMA HL2
- **Removed**: T-Line chop, T-Line cooldown, EMA squeeze, HA trend detection, continuation/flip logic
- Cleaner, more direct entry/exit rules

### v1.0.12
- Multi-contract scaled exits — Trade 1 or 2 contracts with independent exit rules
- Full BE and trailing stop management per contract

### v1.0.11
- Added EMA River Squeeze Filter

### v1.0.10
- Chop zone exit only triggers if position is in a loss

### v1.0.9
- Added Double Stochastic signals and RSI Momentum Ribbon

### v1.0.8
- Fixed strategy settings for futures execution

### v1.0.0 - v1.0.7
- Initial development and debugging
