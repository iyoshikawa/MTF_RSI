# TradingView Alert Setup Guide — MTF RSI Signal Suite

## Overview

The MTF RSI suite has **19 alert conditions** across two indicators. This guide covers how to set them up in TradingView, which alerts matter for different trading styles, and walks through full trade examples from entry to exit.

| Indicator | Alert Count | Purpose |
|-----------|-------------|---------|
| Signals v7.8.0 | 16 | Entry signals, TP/SL hits, warnings |
| Management v7.5.0 | 3 | Pyramid adds, ADX breakout |

---

## Step 1: How to Create an Alert in TradingView

1. Open your chart with the indicator applied
2. Click the **Alert** button (clock icon in the right toolbar, or press `Alt+A`)
3. In the **Condition** dropdown, select your indicator name:
   - `MTF RSI Signals [v7.8.0]` for entry/exit alerts
   - `MTF RSI Management [v7.5.0]` for pyramid/breakout alerts
4. A second dropdown appears — select the specific alert condition
5. Set **Options**:
   - **Trigger**: "Once Per Bar Close" for most alerts (prevents intrabar noise)
   - **Expiration**: Set to "Open-ended" or your session end time
6. Set **Notifications**: Enable push (phone), email, webhook, or all three
7. Click **Create**

> **Tip:** "Once Per Bar Close" is recommended for all alerts. "Once Per Bar" can trigger mid-bar and then the condition may disappear by bar close — you'll get phantom alerts.

---

## Step 2: Which Alerts to Enable

### Recommended Setup: Active Day Trader (1-Min Chart)

**Must have — entry signals:**

| Alert | Why |
|-------|-----|
| 🔥 A+ LONG | Highest conviction long — don't miss these |
| 🔥 A+ SHORT | Highest conviction short |
| ⚡ A LONG | Strong long setup |
| ⚡ A SHORT | Strong short setup |

**Must have — exit management:**

| Alert | Why |
|-------|-----|
| 🎯 TP HIT | Take profit reached — take partials or exit |
| 🛑 SL HIT | Stop loss reached — exit immediately |
| ⚠ HA MOMENTUM FLIP | Heikin Ashi flipped against you — tighten or exit |

**Nice to have — situational:**

| Alert | Why |
|-------|-----|
| ✓ A- LONG / SHORT | Good setups but lower conviction — enable if you want more signals |
| 📊 VOLUME SPIKE | Confirms or warns of exhaustion — useful during active trades |
| 🔀 RSI DIVERGENCE | Early exhaustion warning — helps manage runners |

**Optional — scalpers only:**

| Alert | Why |
|-------|-----|
| 💨 B+ LONG / SHORT SCALP | Quick trend scalps — high signal frequency |
| ⚡ B LONG / SHORT SCALP | Lowest conviction — lots of signals, small edge |

**Optional — Management indicator:**

| Alert | Why |
|-------|-----|
| 📈 PYRAMID ADD LONG / SHORT | Add to winning position on pullback |
| 💥 ADX BREAKOUT | Range breakout detected — new trend starting |

### Recommended Setup: Swing/Position Trader (5-15 Min Chart)

Enable A+ and A only. Skip B grades entirely. Add Regime Change for context shifts.

### Recommended Setup: Alerts-Only (Away From Screen)

Enable all long alerts OR all short alerts (not both — pick your session bias). Add TP HIT, SL HIT, and HA MOMENTUM FLIP so you know when to check your position.

---

## Step 3: Alert Configuration Examples

### Example 1: A+ Long Entry Alert

```
Condition:    MTF RSI Signals [v7.8.0] → 🔥 A+ LONG
Trigger:      Once Per Bar Close
Expiration:   Open-ended
Message:      A+ LONG: All systems aligned — full size runner!
```

**Notifications:** Push + sound. This is your highest conviction signal — you want to hear it.

### Example 2: TP Hit Alert

```
Condition:    MTF RSI Signals [v7.8.0] → 🎯 TP HIT
Trigger:      Once Per Bar Close
Expiration:   Open-ended
Message:      TP HIT: Take profit reached — take partials or exit!
```

**Notifications:** Push. Fires exactly once per trade when price crosses your locked TP level. After this fires, your Management indicator will show the breakeven stop line.

### Example 3: SL Hit Alert

```
Condition:    MTF RSI Signals [v7.8.0] → 🛑 SL HIT
Trigger:      Once Per Bar Close
Expiration:   Open-ended
Message:      SL HIT: Stop loss reached — exit position!
```

**Notifications:** Push + sound + email. This is your emergency exit — make it impossible to ignore.

### Example 4: Pyramid Add (Management)

```
Condition:    MTF RSI Management [v7.5.0] → 📈 PYRAMID ADD LONG
Trigger:      Once Per Bar Close
Expiration:   Open-ended
Message:      ADD LONG: Grade upgraded or EMA pullback — add to position!
```

**Notifications:** Push only. Optional signal — only act if your existing position is in profit and the runner is active.

---

## Full Trade Examples

### Example A: A+ Long Runner on MNQ (Best Case)

**Setup:** 1-min MNQ chart, NY session, Conservative MTF speed.

| Time | Event | Alert | Action |
|------|-------|-------|--------|
| 9:42 | A+ LONG signal fires | 🔥 A+ LONG | Enter long at 19,523. Dashboard shows: 3/3 MTF BULL, ADX 28 STRONG, DStoch BULL, VOL EXPANDING |
| 9:42 | Signals indicator plots levels | — | SL locked at 19,510 (13pts below). TP locked at 19,548 (25pts above). Both visible as crosses on chart |
| 9:42 | Management shows trade lifecycle | — | TP1 (cyan line) at 19,548. TP2 (purple line, ATR-based) at 19,573. TP/SL MODE dashboard says "🔥 ATR TP" and "🏃 TRAIL (34 EMA)" |
| 9:55 | Volume spike early in move | 📊 VOLUME SPIKE | Dashboard shows "CONFIRM 2.3x avg" — volume validates the move. Stay in full size |
| 10:08 | Price hits 19,548 | 🎯 TP HIT | Take 50% off. Management shows breakeven stop (gold line) at 19,523. TP2 runner target still showing at 19,573 |
| 10:08 | Move SL to breakeven | — | Your risk is now zero on the remaining 50%. Runner trail (34 EMA stepline) is tracking at ~19,535 |
| 10:22 | Pyramid pullback to 34 EMA | 📈 PYRAMID ADD LONG | Price pulled back to trail EMA and bounced. Add 25-50% position back. New avg entry ~19,540 |
| 10:41 | Price hits TP2 at 19,573 | — | Take remaining position. Dashboard P&L shows +50pts on original, +33pts on add |
| 10:41 | Grade drops to NONE | — | All levels and tracking reset. Wait for next signal |

**Result:** +50 pts on first half (entry to TP1), +50 pts on runner (entry to TP2), +33 pts on pyramid add. Total: ~133 MNQ pts × $2/pt = $266 per contract.

---

### Example B: A Short Standard Exit on YM (Clean Trade)

**Setup:** 1-min YM chart, NY PM session.

| Time | Event | Alert | Action |
|------|-------|-------|--------|
| 13:45 | A SHORT signal fires | ⚡ A SHORT | Enter short at 42,180. Dashboard: 3/3 MTF BEAR, ADX 22 GOOD, HMA STRONG SHORT |
| 13:45 | Levels locked | — | SL at 42,195 (15pts above). TP at 42,155 (25pts below) |
| 13:45 | Management dashboard | — | TP MODE: "FIXED TP (ADX ▼)" — ADX falling, don't stretch for ATR TP. SL MODE: "FIXED SL" |
| 14:02 | Price grinds to 42,155 | 🎯 TP HIT | Full exit at TP — this was an A grade with falling ADX, not a runner. Dashboard recommended FIXED TP for exactly this reason |

**Result:** +25 pts × $5/pt (full YM) = $125 per contract. Clean, no drama.

---

### Example C: B+ Scalp with SL Hit on MES (Loss Management)

**Setup:** 1-min MES chart, morning session.

| Time | Event | Alert | Action |
|------|-------|-------|--------|
| 10:15 | B+ LONG SCALP signal | 💨 B+ LONG SCALP | Enter long at 5,542. Only 50% position (B+ sizing). SL at 5,527 (15pts). TP at 5,545 (3pts) |
| 10:15 | Dashboard context | — | VOL FLOW: CONTRACTING 0.8x. ADX: 17 WEAK. Not ideal — volume isn't confirming |
| 10:18 | Price reverses | — | Dropping toward stop. HA flips bearish |
| 10:19 | HA momentum warning | ⚠ HA MOMENTUM FLIP | 2 consecutive HA bars against you. Consider exiting early before SL |
| 10:21 | Price hits 5,527 | 🛑 SL HIT | Exit at stop. Loss of 15 pts × $5/pt = $75 per MES contract. At 50% sizing (B+ = 2-3 cts), total loss $150-225 — within max pain |

**Lesson:** The VOL FLOW was already showing contraction at entry. The B+ grade was marginal. The HA MOMENTUM FLIP gave an early warning 2 bars before the stop was hit — exiting there would have saved 4-5 points.

---

### Example D: A- Long with Divergence Warning on MCL (Exit Early)

**Setup:** 1-min CL chart, pit session.

| Time | Event | Alert | Action |
|------|-------|-------|--------|
| 9:52 | A- LONG signal | ✓ A- LONG | Enter long at 72.40. SL at 72.10 ($0.30). TP at 72.55 ($0.15) |
| 9:52 | Management dashboard | — | TP MODE: "FIXED TP". SL MODE: "FIXED SL". A- doesn't qualify for ATR TP or trail |
| 10:05 | Price at 72.52, approaching TP | — | Almost there — 3 ticks from TP |
| 10:06 | RSI divergence warning | 🔀 RSI DIVERGENCE | Dashboard shows "⚠ BEARISH DIV" — price making higher high but RSI making lower high. Momentum fading |
| 10:06 | Volume divergence too | — | VOL DIVG row: "⚠ PRICE ▲ VOL ▼". Two divergence warnings at once |
| 10:07 | Exit manually at 72.51 | — | Don't wait for TP. Two divergence warnings + A- grade = take what you have |

**Result:** +$0.11 ($110 per CL contract) instead of waiting for the $0.15 TP that may not have hit. Price reversed to 72.35 by 10:15 — the divergence warnings saved the trade.

---

### Example E: Signals-Only (No Management Indicator)

You can run Signals v7.8.0 standalone without Management. Here's what that looks like:

| Time | Event | Alert | Action |
|------|-------|-------|--------|
| 9:35 | A+ LONG signal | 🔥 A+ LONG | Enter long. Chart shows: SL cross (red) and TP cross (cyan) |
| 9:35 | Dashboard shows | — | Grade, direction, signal age, VOL FLOW, VOL SPIKE, VOL DIVG, ADX, DStoch, HMA trend, MTF alignment |
| 9:48 | Price hits TP level | 🎯 TP HIT | Exit at TP. You don't get the partial/runner/breakeven management — that's in Management |
| — | What you're missing | — | No TP1/TP2 split, no breakeven stop, no live P&L, no sizing dashboard, no runner trail, no pyramid adds |

**When to use Signals-only:** If you trade a simple approach (enter at signal, exit at TP or SL, no scaling) or you're running into TradingView's indicator limit and can't load both.

---

### Example F: Management-Only (Trade Already Open)

If you entered a trade from a different system or discretionary read, you can use Management alone for exit management:

| Time | Event | What Management Shows |
|------|-------|-----------------------|
| — | Apply Management to chart with YM preset | Sizing dashboard, trade context panel, Globex levels |
| 10:00 | Grade happens to align with your existing position | TP1 line (cyan), TP2 line (purple) appear. Runner trail starts tracking. P&L row activates |
| 10:15 | TP1 hit | Breakeven stop line appears. P&L updates. TP STATUS: "✓ TP1 HIT → BE stop" |
| 10:30 | Runner trail catches up | Trail stop (stepline) moving with 34 EMA. SL MODE: "🏃 TRAIL (34 EMA)" |

**When to use Management-only:** You have your own entry method but want professional exit management with partial targets, breakeven automation, and live P&L tracking.

---

## Alert Cheat Sheet

### Signals v7.8.0 (16 alerts)

| Alert | Direction | Fires When | Frequency |
|-------|-----------|------------|-----------|
| 🔥 A+ LONG | Long | A+ long signal confirmed | Rare — 1-3 per session |
| 🔥 A+ SHORT | Short | A+ short signal confirmed | Rare |
| ⚡ A LONG | Long | A long signal confirmed | Uncommon — 2-5 per session |
| ⚡ A SHORT | Short | A short signal confirmed | Uncommon |
| ✓ A- LONG | Long | A- long signal confirmed | Moderate — 3-8 per session |
| ✓ A- SHORT | Short | A- short signal confirmed | Moderate |
| 💨 B+ LONG SCALP | Long | B+ long scalp confirmed | Frequent — 5-15 per session |
| 💨 B+ SHORT SCALP | Short | B+ short scalp confirmed | Frequent |
| ⚡ B LONG SCALP | Long | B long scalp confirmed | Very frequent |
| ⚡ B SHORT SCALP | Short | B short scalp confirmed | Very frequent |
| 🎯 TP HIT | — | Price crosses locked TP | Once per trade |
| 🛑 SL HIT | — | Price crosses locked SL | Once per trade |
| ⚠ HA MOMENTUM FLIP | — | HA flips against position | During active trades |
| 📊 REGIME CHANGE | — | Consolidation regime shifts | 1-3 per day |
| 🔀 RSI DIVERGENCE | — | Price-RSI divergence detected | Occasional |
| 📊 VOLUME SPIKE | — | Volume > mean + 2σ | Several per session |

### Management v7.5.0 (3 alerts)

| Alert | Direction | Fires When | Frequency |
|-------|-----------|------------|-----------|
| 📈 PYRAMID ADD LONG | Long | Grade upgrade or EMA pullback | During runners |
| 📈 PYRAMID ADD SHORT | Short | Grade upgrade or EMA pullback | During runners |
| 💥 ADX BREAKOUT | — | ADX crosses from low to high | 1-2 per day |

---

## Webhook Integration (Advanced)

For automated logging or broker integration, TradingView alerts support webhook URLs. Set the **Webhook URL** in the alert dialog and use the message field for structured data:

```
Alert Message Example (A+ LONG):
{"signal":"A+","direction":"LONG","instrument":"{{ticker}}","price":"{{close}}","time":"{{time}}"}
```

```
Alert Message Example (TP HIT):
{"event":"TP_HIT","instrument":"{{ticker}}","price":"{{close}}","time":"{{time}}"}
```

```
Alert Message Example (SL HIT):
{"event":"SL_HIT","instrument":"{{ticker}}","price":"{{close}}","time":"{{time}}"}
```

TradingView placeholders like `{{ticker}}`, `{{close}}`, and `{{time}}` are replaced automatically when the alert fires. Use these with services like TradingView-to-Discord bots, Google Sheets loggers, or broker APIs.

---

## Troubleshooting

**"I'm getting too many alerts"**
- Disable B and B+ alerts. Only use A+ and A for entries.
- Increase Signal Cooldown (bars) in Signals settings — 8-10 bars on 1-min reduces frequency.
- Make sure your session config matches your actual trading hours.

**"TP HIT / SL HIT alerts aren't firing"**
- TP and SL levels lock on signal entry. If no grade signal has fired, there's nothing to track.
- Levels reset when grade drops to NONE. If the grade disappeared before price hit TP/SL, the alert won't fire.
- Use "Once Per Bar Close" trigger — "Once Per Bar" may fire and then disappear within the same bar.

**"I get both LONG and SHORT alerts in the same session"**
- This is normal — the indicator can flip direction during a session as confluence shifts.
- If you only want one direction, filter alerts by your pre-session bias (only enable LONG alerts on bullish bias days).

**"Management alerts fire but I don't see the signal on the Signals indicator"**
- Management recalculates the grade independently. The cooldown and persistence settings must match between both indicators.
- Check that all shared settings (preset, RSI, stochastic, ADX, session config) are identical.

**"Alert says A+ LONG but the chart shows B+"**
- The alert fired on bar close when conditions were A+. By the next bar, conditions may have changed.
- This is why "Once Per Bar Close" is important — it ensures the condition was true at the close, not just intrabar.
