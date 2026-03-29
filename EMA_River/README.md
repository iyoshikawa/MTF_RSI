# EMA River Strategy

A multi-confluence futures trading strategy for YM, NQ, ES, CL, HG on time-based charts (30s).

## Version
- **Current**: v1.0.12

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

### v1.0.12
- **Multi-contract scaled exits** — Trade 1 or 2 contracts with independent exit rules
- Contract 1 (Scalp): 50 tick TP, 40 tick SL, BE at 30 profit (+10 offset), optional trail
- Contract 2 (Runner): 125 tick TP, 40 tick SL, BE at 30 profit (+10 offset), trail enabled (20 tick stop after 30 profit)
- Toggle `Trade 2 Contracts` to use single or dual contract mode
- Full BE and trailing stop management per contract

### v1.0.11
- Added **EMA River Squeeze Filter** — no trades when river spread < threshold (default 15pts)
- River squeeze exits ALL positions immediately (unlike other chop filters which let winners run)
- Debug table now shows river spread in points and squeeze status

### v1.0.10
- Chop zone exit now only triggers if position is **in a loss**
- Profitable positions are allowed to continue running through chop zone

### v1.0.9
- Added **Double Stochastic** signals (DS▲/DS▼ markers on crossovers)
- Added **RSI Momentum Ribbon** background (bull=green, bear=red, neutral=gray)
- Added **Consolidation Detection** (grays out ribbon in tight ranges)
- Added **DStoch + RSI Confluence** brightening (transparency -20 when aligned)
- Made **Chop Zone** and **No-Trade Session** backgrounds toggleable
- Expanded debug table with DStoch, RSI Ribbon, and Confluence status

### v1.0.8
- Fixed strategy settings preventing trade execution on futures
- Initial capital increased to $1M (futures contract value requirement)
- Added `currency=currency.USD`
- Added `calc_on_order_fills=true`
- Removed `fill_orders_on_standard_ohlc` (not needed)

### v1.0.7
- Added TEST MODE toggle to bypass all filters and test basic entries (simple EMA cross)
- Use this to confirm strategy can execute trades at all

### v1.0.6
- Fixed strategy settings for futures (removed margin settings)
- Simplified exit logic to basic TP/SL/Trail with proper tick conversion
- Changed to `process_orders_on_close = true` for reliable execution
- Changed to `fill_orders_on_standard_ohlc = true` for consistent fills

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
