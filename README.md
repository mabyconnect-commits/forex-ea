# TradahEA — ICT / Smart Money MT5 Expert Advisor

> Full-stack ICT/SMC strategy EA for MetaTrader 5.  
> Instruments: XAUUSD · BTCUSD · USOIL · GBPUSD  
> Strategy: Order Blocks · Fair Value Gaps · Breaker Blocks · 4H Trend Bias

---

## Table of Contents

1. [Strategy Overview](#strategy-overview)
2. [Installation (Windows + MetaEditor)](#installation)
3. [Input Parameters Reference](#input-parameters)
4. [Telegram Bot Setup](#telegram-setup)
5. [VPS Setup Guide](#vps-setup)
6. [Live vs Prop Account Modes](#account-modes)
7. [Remote Commands](#remote-commands)
8. [FAQ / Troubleshooting](#faq)

---

## Strategy Overview

### Market Structure Logic

| Component | Timeframe | Purpose |
|-----------|-----------|---------|
| Trend bias | 4H | Higher Highs + Higher Lows = bullish; Lower Highs + Lower Lows = bearish |
| Entry confirmation | 15M / 30M | Order Block, FVG, or Breaker Block |
| SL placement | Per POI candle | Below/above wick of OB/FVG candle |
| TP target | 4H | Most recent swing resistance (bull) or support (bear) |

### Entry Concepts

**Order Block (OB)**  
- Bullish OB: Last bearish candle immediately before a strong bullish impulse  
- Bearish OB: Last bullish candle immediately before a strong bearish impulse  
- Price entering the OB zone = entry signal in direction of trend

**Fair Value Gap (FVG)**  
- 3-candle pattern with a price gap between candle[0] and candle[2]  
- Bullish FVG: `candle[0].low > candle[2].high`  
- Bearish FVG: `candle[0].high < candle[2].low`  
- Price filling the FVG = entry signal

**Breaker Block (BB)**  
- A previous OB that price has since broken through  
- Now acts as support (was resistance) or resistance (was support)  
- Retests of Breaker Blocks = high-probability entries

### Trade Management

| Rule | Trigger |
|------|---------|
| Move SL to breakeven | When trade reaches **2:1 R:R** |
| Close 50% of position | When trade reaches **3:1 R:R** |
| Let remainder run | Until full TP is hit |

### Session Filter

The EA only trades during:
- **Monday–Friday**
- **07:00 – 22:00 UTC** (London + New York sessions)
- Sydney session (22:00–07:00 UTC) is excluded

### News Filter

- If a high-impact news event is scheduled within **±1 hour**, the EA will not open new trades
- Existing trades are NOT closed; only new entries are blocked
- The EA uses MT5's built-in Economic Calendar (`CalendarValueHistory`)

---

## Installation

### Step 1 — Copy EA file

1. Open your MT5 terminal
2. Click **File → Open Data Folder**
3. Navigate to `MQL5/Experts/`
4. Copy `TradahEA.mq5` into this folder

### Step 2 — Compile

1. Open **MetaEditor** (press F4 in MT5, or Tools → MetaQuotes Language Editor)
2. In the Navigator panel on the left, find `Experts → TradahEA`
3. Double-click to open the file
4. Press **F7** (or click the Compile button)
5. Confirm: "0 errors, 0 warnings" in the Errors tab at the bottom

### Step 3 — Enable WebRequests (REQUIRED for Telegram)

1. In MT5: **Tools → Options → Expert Advisors**
2. Tick: ✅ **Allow WebRequests for listed URLs**
3. Click the **+** button and add:
   ```
   https://api.telegram.org
   ```
4. Click **OK**

### Step 4 — Enable Automated Trading

1. In MT5 toolbar, ensure **AutoTrading** is enabled (green button)
2. In **Tools → Options → Expert Advisors**:
   - ✅ Allow automated trading
   - ✅ Allow DLL imports (if prompted)

### Step 5 — Attach EA to Chart

1. Open a chart — **XAUUSD M15** is recommended
2. In the Navigator panel: **Expert Advisors → TradahEA**
3. Drag it onto the chart (or double-click)
4. The EA parameters window will appear

### Step 6 — Configure Parameters

| Parameter | Recommended Value |
|-----------|------------------|
| `AccountType` | `PROP` (for prop firms) or `LIVE` |
| `PropRiskPercent` | `1.0` (1% per trade) |
| `LiveRiskPercent` | `10.0` (10% per trade) |
| `TelegramBotToken` | Your bot token from @BotFather |
| `TelegramChatID` | Your chat/channel ID |
| `MagicNumber` | Any unique number (default: 202401) |
| `TradeXAUUSD` | `true` |
| `TradeBTCUSD` | `true` |
| `TradeUSOIL` | `true` |
| `TradeGBPUSD` | `true` |

> ⚠️ The EA is chart-attached and will trade ALL configured symbols from a single chart. You do NOT need to attach it to each symbol's chart separately.

---

## Input Parameters

### Account & Risk

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `AccountType` | Enum | `PROP` | LIVE = 10-20% risk / PROP = 1-2% risk |
| `LiveRiskPercent` | double | 10.0 | Risk % per trade in LIVE mode |
| `PropRiskPercent` | double | 1.0 | Risk % per trade in PROP mode |

### Instruments

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `TradeXAUUSD` | bool | true | Enable XAUUSD trading |
| `TradeBTCUSD` | bool | true | Enable BTCUSD trading |
| `TradeUSOIL` | bool | true | Enable USOIL trading |
| `TradeGBPUSD` | bool | true | Enable GBPUSD trading |
| `MaxPairsOpen` | int | 2 | Max simultaneous pairs |
| `MaxTradesPerPair` | int | 2 | Max trades per symbol |

### Strategy Tuning

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `MagicNumber` | int | 202401 | Unique EA identifier |
| `Lookback4H` | int | 10 | 4H candles for trend detection |
| `LookbackEntry` | int | 20 | Candles to scan for POI |
| `FVGMinPips` | double | 5.0 | Minimum FVG size to qualify |
| `OBBufferPips` | double | 2.0 | Entry buffer inside OB zone |
| `BreakevenRR` | double | 2.0 | R:R to trigger breakeven |
| `PartialCloseRR` | double | 3.0 | R:R to trigger 50% close |

### Session

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `SessionStartHour` | int | 7 | Session open (UTC) |
| `SessionEndHour` | int | 22 | Session close (UTC) |

### Notifications

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `TelegramBotToken` | string | `` | From @BotFather |
| `TelegramChatID` | string | `` | Your Telegram chat ID |
| `EmailAddress` | string | `` | Email for notifications |

---

## Telegram Setup

### Create a Bot

1. Open Telegram and search for **@BotFather**
2. Send `/newbot`
3. Follow prompts to choose a name and username
4. BotFather will give you a **Bot Token** like:  
   `123456789:ABCdefGHIjklMNOpqrSTUvwxYZ`
5. Copy this token → paste into `TelegramBotToken` parameter

### Get Your Chat ID

**Option A — Personal chat:**
1. Search for **@userinfobot** on Telegram
2. Send `/start`
3. It replies with your chat ID (e.g. `987654321`)

**Option B — Channel:**
1. Add your bot to the channel as admin
2. Send a message to the channel
3. Visit:  
   `https://api.telegram.org/bot<TOKEN>/getUpdates`
4. Look for `"chat":{"id":` — this is your channel ID (starts with `-100...`)

### Available Bot Commands

Once running, you can send these to your bot:

| Command | Action |
|---------|--------|
| `/pause` | Stop new trade entries |
| `/resume` | Re-enable trade entries |
| `/status` | Current EA status, balance, open trades |
| `/report` | Generate on-demand weekly report |

---

## VPS Setup

Running the EA on a VPS ensures 24/7 operation even when your PC is off.

### Recommended VPS Providers

| Provider | Min Spec | Monthly Cost |
|----------|----------|-------------|
| Contabo | 4GB RAM, 2 vCPU | ~$5 |
| Vultr | 2GB RAM, 1 vCPU | ~$6 |
| ForexVPS.net | MT5 optimised | ~$18 |
| AWS Lightsail | 2GB RAM | ~$10 |

### Setup Steps (Windows VPS)

1. **RDP into your VPS** (Remote Desktop)
2. **Download MT5** from your broker's website
3. **Login** to your trading account
4. **Install EA** as per the Installation steps above
5. **Configure MT5 to run minimized at startup:**
   - Create a shortcut to `terminal64.exe`
   - Add argument: `/portable`
   - Put in Windows Startup folder (`shell:startup`)
6. **Disable screensaver** and set **Power Plan → High Performance**
7. **Verify AutoTrading is ON** every time you reconnect via RDP

### Keeping EA Running

- Enable **Stay Connected** in MT5 preferences
- Set **Charts → Properties → Keep Data** to prevent chart reload issues
- Consider **ForexVPS Keep Alive** plugins if using a dedicated Forex VPS

---

## Account Modes

### PROP Firm Mode

Configure: `AccountType = PROP`, `PropRiskPercent = 1.0`

- 1–2% risk per trade keeps you within prop firm daily/total drawdown limits
- EA respects max 2 pairs simultaneously to avoid correlation risk
- Recommended for: FTMO, MyForexFunds, The5%ers, etc.

### LIVE Account Mode

Configure: `AccountType = LIVE`, `LiveRiskPercent = 10.0`

- 10–20% risk per trade — aggressive compounding
- Only suitable for accounts you can afford to lose
- Consider starting at 5% until you validate the strategy

---

## Remote Commands

The EA polls Telegram every 30 seconds for commands. Only messages from the configured `TelegramChatID` are processed.

```
/pause   — Stops new trade entries (existing trades continue)
/resume  — Allows new trade entries again
/status  — Shows: mode, paused state, session, open positions, balance/equity
/report  — Instant weekly performance summary
```

Weekly reports are automatically sent every **Friday at 16:00 UTC**.

---

## FAQ

**Q: EA is attached but not trading?**
- Check AutoTrading is enabled (green button in toolbar)
- Check the session filter — EA only trades 07:00–22:00 UTC, Mon–Fri
- Check if `/pause` was sent to the bot
- Check if `MaxPairsOpen` limit was hit

**Q: Telegram messages not sending?**
- Verify WebRequests are enabled for `https://api.telegram.org`
- Confirm `TelegramBotToken` and `TelegramChatID` are correct
- Check MT5 Experts tab for error messages

**Q: EA opened a trade but SL seems too wide?**
- The OB/FVG candle wick determines SL placement
- Adjust `OBBufferPips` to add/reduce buffer
- For tighter SL on volatile pairs (BTCUSD), consider reducing `LookbackEntry`

**Q: Can I run it on multiple accounts simultaneously?**
- Yes — use a different `MagicNumber` per account to avoid trade tracking conflicts

**Q: How does the news filter work?**
- It uses MT5's built-in Economic Calendar
- Ensure your broker provides calendar data (most do)
- If no calendar data is available, the filter returns `false` (does not block)

**Q: What happens if MT5 restarts?**
- `OnInit()` runs again automatically
- Stats are reloaded from `TradahEA_stats.csv`
- Open positions are still managed (trailing stop, partial close)
- Telegram polling resumes from the last processed update ID (resets to 0 on restart — recent commands may be re-processed once)

---

## File Structure

```
forex-ea/
├── TradahEA.mq5          ← Main EA file
├── README.md             ← This file
└── backtest_guide.md     ← Backtesting instructions
```

After compilation, MT5 will create:
```
MQL5/
├── Experts/
│   ├── TradahEA.mq5
│   └── TradahEA.ex5      ← Compiled binary
└── Files/
    └── TradahEA_stats.csv ← Performance tracking
```

---

*Built for Maby — TradahEA v1.0 | ICT Smart Money Strategy*
