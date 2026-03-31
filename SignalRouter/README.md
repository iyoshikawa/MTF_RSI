# Signal Router

Multi-confluence signal routing strategy for QuantLynk/Tradovate integration.

## Version: 2.3.0

## Features

### Dual Independent Confluence Stacks
- **Long Stack:** 6 gates (L1-L6) with independent signal type + price relation conditions
- **Short Stack:** 6 gates (S1-S6) with independent signal type + price relation conditions
- **Universal Gates:** 3 toggleable gates (U1-U3) that apply to both directions

### Gate Conditions
Each gate has two conditions that must BOTH pass:

**Signal Type:**
- any, cross, cross_up, cross_down, gt_zero, lt_zero, rising, falling, ignore

**Price Relation:**
- ignore, price_above, price_below, price_cross_up, price_cross_down

### Filters
- **News Blocker:** Time-based US red folder news avoidance (8:30, 10:00, 10:30, 14:00, 14:30 ET)
- **Session Filter:** Asian, London AM, NY AM, NY PM windows

### Trade Execution
- Single contract trailing stop entry
- QuantLynk JSON payload format
- Configurable stop loss in ticks

## QuantLynk Payload Format

```json
{
  "qv_user_id": "<your_id>",
  "alert_id": "<your_alert>",
  "quantity": "{{strategy.order.contracts}}",
  "order_type": "trailingstop",
  "action": "{{strategy.order.action}}",
  "price": "{{close}}",
  "stop_price": "<calculated from SL ticks>"
}
```

## Setup

1. Add strategy to chart
2. Configure gates, sessions, news blocker
3. Enter `qv_user_id` and `alert_id` in QuantLynk settings
4. Set `Stop Loss (ticks)` — default 40
5. Set tick size (YM=1.0, ES/NQ=0.25)
6. Create alert → Message: `{{strategy.order.alert_message}}`
7. Webhook URL: QuantLynk endpoint

## Files

- `SignalRouter_v2.3.0.pine` — Main strategy
- `ConfluenceOutput_Template_v1.0.0.pine` — Template indicator showing how to structure plot outputs for the router

## Companion Indicator

The Confluence Output Template shows how to structure indicator plots for the router:
- `SIG:` prefix → Entry signals
- `L:` prefix → Long gates
- `S:` prefix → Short gates
- `U:` prefix → Universal gates

Output `1` for TRUE, `0` for FALSE.
