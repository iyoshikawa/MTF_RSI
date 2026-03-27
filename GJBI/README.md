# GJBI Indicators

Adapted from the UAI Backdoor Heiken-Ashi trading methodology by @Geo-TX.

## Overview

These indicators are designed to work together to identify trend direction and optimal entry/exit points. They are particularly effective on Heiken-Ashi charts but work on any chart type.

---

## GJBIEMABG - EMA Background & River

**Purpose:** Chart background indicator that highlights trend direction with a visual "River" for price action context.

### Features
- **Background Coloring**: Green background when price > 50 EMA (bullish), Red when price < 50 EMA (bearish)
- **The River**: Three EMA lines (20 EMA applied to High, Close, and Low) creating a price channel
- **Real-time Updates**: Calculates on every price change, not just bar close

### Trading Logic (from source material)
1. Look for background color to match trade direction
2. Wait for price to cross INTO the River from below (bullish) or above (bearish)
3. Entry trigger: Price rising from below River Low, crossing into the River zone

### Default Settings
| Parameter | Default | Description |
|-----------|---------|-------------|
| Trend EMA | 50 | Main trend direction EMA |
| River EMA | 20 | EMA length for River (H/L/C) |

---

## GJBIHMACrossRibbon - HMA Crossover Ribbon

**Purpose:** Ribbon indicator showing the interaction between fast and slow Hull Moving Averages.

### Features
- **HMA Ribbon**: Visual ribbon between 6 HMA (fast) and 21 HMA (slow)
- **Trend Colors**: Green ribbon = bullish (fast > slow), Red ribbon = bearish (fast < slow)
- **Crossover Markers**: Triangle markers at HMA crossover points
- **Momentum Tracking**: Dashboard shows if HMA is rising or falling

### Hull Moving Average (HMA)
HMA reduces lag while maintaining smoothness:
```
HMA = WMA(2 × WMA(n/2) − WMA(n), √n)
```

### Default Settings
| Parameter | Default | Description |
|-----------|---------|-------------|
| Fast HMA | 6 | Fast HMA period |
| Slow HMA | 21 | Slow HMA period |
| Source | Close | Price source |

---

## Combined Usage

The indicators work together for multi-confluence entries:

### Bullish Setup
1. ✅ **GJBIEMABG**: Green background (price > 50 EMA)
2. ✅ **GJBIEMABG**: Price entering River from below
3. ✅ **GJBIHMACrossRibbon**: Green ribbon (6 HMA > 21 HMA)
4. ✅ **GJBIHMACrossRibbon**: Fast HMA rising

### Bearish Setup
1. ✅ **GJBIEMABG**: Red background (price < 50 EMA)
2. ✅ **GJBIEMABG**: Price entering River from above
3. ✅ **GJBIHMACrossRibbon**: Red ribbon (6 HMA < 21 HMA)
4. ✅ **GJBIHMACrossRibbon**: Fast HMA falling

---

## Session Recommendations

Per the source methodology:
- **Best**: Asia session open, London session open
- **Good**: NY open (wait for 9:30 AM)
- **Caution**: Pre-market (use smaller timeframes like 1-min)

---

## Installation

1. Open TradingView
2. Pine Editor → New Script
3. Paste indicator code
4. Save and add to chart

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-03-27 | Initial release - PineScript v6 |

---

## Credits

- Original concept: @Geo-TX (UAI Backdoor methodology)
- PineScript adaptation: iyoshikawa
