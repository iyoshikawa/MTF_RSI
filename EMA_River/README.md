# EMA River Strategy

A multi-confluence futures trading strategy for YM, NQ, ES, CL, HG on time-based charts (30s).

## Version
- **Current**: v1.0.5

## Core Components

| Component | Setting |
|-----------|---------|
| 9 EMA | OHLC4 |
| 20 EMA River | High / HL2 / Low |
| 20 HMA | Close |
| Trend Lookback | 7 HA candles |

## Entry Types

### Type 1: Continuation (with trend)
- **Long**: Uptrend confirmed → price pulls back between HMA and 9 EMA → breaks above HMA
- **Short**: Downtrend confirmed → price pulls back between HMA and 9 EMA → closes below 9 EMA

### Type 2: Flip (counter-trend reversal)
- **Long**: Prior downtrend → price closes above BOTH 20 HMA AND 9 EMA
- **Short**: Prior uptrend → price closes below BOTH 20 HMA AND 9 EMA

## Chop Filters

| Filter | Settings |
|--------|----------|
| Bollinger Bands | 10 length, 1.2 stddev, 3min MTF |
| T-Line Channel | 55 EMA, 34 point width |

When price is inside either filter zone:
- No new entries allowed
- Existing positions are flattened

## Session Windows (EST)

| Session | Hours |
|---------|-------|
| Globex Evening | 18:01–20:00 |
| Pre-Market | 07:00–08:15 |
| NY Session | 09:30–16:00 |

## Position Management

| Parameter | Default |
|-----------|---------|
| Take Profit | 40 points |
| Stop Loss | 40 points |
| Trail Distance | 20 points |
| BE Activation | 75% of TP (30 points) |
| BE Offset | +3 points |

## Signal Arrows

| Arrow | Color | Meaning |
|-------|-------|---------|
| LC | Green | Long Continuation |
| SC | Red | Short Continuation |
| LF | Aqua | Long Flip |
| SF | Fuchsia | Short Flip |

## Changelog

### v1.0.5
- Added entry attempt labels ("ENTRY LONG" / "ENTRY SHORT") to debug why trades aren't executing

### v1.0.4
- Added debug table (toggle in Display settings) to diagnose entry conditions

### v1.0.3
- Added T-Line cooldown after exiting chop zone
  - Cooldown Bars: suppress signals for X bars after leaving (default 10)
  - Cooldown Distance: OR allow signals if price moves X points from boundary (default 20)
  - Both conditions must be false for cooldown to end (bars elapsed OR distance exceeded)

### v1.0.2
- Fixed session filter type error: `time()` returns int, converted to bool with `not na()`

### v1.0.1
- Dynamic MA colors: all MAs now change color based on price position (lime above, red below)

### v1.0.0
- Initial release
- Continuation and Flip entry logic
- BB + T-Line chop filters
- Session filtering
- TP/SL/Trail/BE exit management
