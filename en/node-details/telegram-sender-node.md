# Telegram Sender Node

Telegram Sender Node sends notification messages to Telegram chats or channels.

---

## Node Information

| Property | Value |
|----------|-------|
| **Node Type** | `telegram_sender_node` |
| **Display Name** | Telegram Sender |
| **Category** | Output (Notification) |
| **Icon** | ✈️ Paper Plane Icon |
| **Handle Color** | Blue |

---

## Functionality

Telegram Sender Node integrates with Telegram Bot API to send automated notifications, alerts, and trading updates to specified chats or channels.

**Main Uses:**
- Send trading alerts and notifications
- Report workflow execution results
- Send error and warning messages
- Automated status updates

**Core Features:**
- 📱 **Telegram Integration**: Direct bot API access
- 📝 **Rich Formatting**: Markdown and HTML support
- 🔔 **Real-Time Delivery**: Instant message delivery
- 👥 **Multi-Target**: Send to users or channels

---

## 📖 Full Documentation

For detailed documentation including complete parameter descriptions, usage examples, and advanced features, please refer to:

- [Chinese version](../../zh/node-details/telegram-sender-node.md) - Complete documentation (~11,600 words)
- Frontend configuration: `1_weather_frontend/src/pages/flow/components/TFNode/instances/outputs/TelegramSenderNode.tsx`
- Backend implementation: `3_weather_cluster/tradingflow/station/nodes/telegram_sender_node.py`

---

**Maintained by:** TradingFlow Development Team  
**Version:** 1.0.0
