# Polymarket automated trading system

This is an automated trading system for Polymarket. It focuses on scanning markets that are about to end and running automated strategies. **Redeem is not implemented**—you need to **claim** winnings yourself. For a new wallet, the first order: **enable all three options** in the web UI when prompted.

## Core features

- **Smart market scan**: Scans markets ending in 1–6 minutes
- **Scheduled runs**: Runs at 10, 25, 40, and 55 past each hour (5 minutes before 15, 30, 45, and 0)
- **Conservative strategy**: Trades only in the 0.90–0.98 price range
- **Fully automated**: No manual steps during normal operation
- **Monitoring**: Full trade logs and stats
- **Config**: Environment variables to keep your private key out of the repo

## Requirements

- Python 3.8+
- [uv](https://github.com/astral-sh/uv) (recommended)
- A Polymarket account
- USDC to trade

## Quick start

### 1. Install dependencies

```bash
# using uv (recommended)
uv sync

# or with pip
pip install -r requirements.txt
```

### 2. Configure environment

Copy the example and set your values:

```bash
cp config.example.env .env
```

Edit `.env`:

```bash
# Required
PRIVATE_KEY=your_private_key_here
FUNDER=your_funder_address_here
RPC_URL=https://polygon-rpc.com

# Trading
MIN_PRICE_RANGE=0.90
MAX_PRICE_RANGE=0.98
TRADE_AMOUNT=2.0
MAX_ORDER_SIZE=2.0
MIN_ORDER_SIZE=0.1
```

### 3. Run

```bash
# Start the auto-trading scheduler
python advanced_scheduler.py

# Or one-off / dry run
uv run main.py --start-minutes 1 --end-minutes 6 --auto-trade --test-only
```

## Configuration

### Testing

For testing, use a **new EVM wallet** and fund it with a **small** amount.

### Environment variables

| Variable | Description | Default |
|----------|-------------|---------|
| `PRIVATE_KEY` | Wallet private key (required) | - |
| `FUNDER` | Funder / funding address (required). Your Polymarket wallet: open the site, click your avatar to find it. | - |
| `RPC_URL` | Polygon RPC endpoint | https://polygon-rpc.com |
| `MIN_PRICE_RANGE` | Min price to trade | 0.90 |
| `MAX_PRICE_RANGE` | Max price to trade | 0.98 |
| `TRADE_AMOUNT` | Notional per trade (USDC) | 2.0 |
| `MAX_ORDER_SIZE` | Max order size | 2.0 |
| `MIN_ORDER_SIZE` | Min order size | 0.1 |

### Scheduler

Edit `scheduler_config.json`:

```json
{
  "interval_minutes": 5,
  "max_trades": 1,
  "scan_start_minutes": 1,
  "scan_end_minutes": 6,
  "min_time_remaining": 1,
  "test_mode": false
}
```

## Trading strategy

### Scanning

- **Time window**: Markets ending in 1–6 minutes
- **Schedule**: 10, 25, 40, 55 past the hour
- **Market type**: Short-term Up/Down style markets

### Execution

- **Price band**: 0.90–0.98 only
- **Size**: 2.0 USDC per trade (configurable)
- **Slippage**: 1% tolerance
- **Risk**: Conservative, targeting higher win rate

## Usage

### Auto mode

```bash
# Start scheduler
python advanced_scheduler.py

# Single scan with auto trade
uv run main.py --start-minutes 1 --end-minutes 6 --auto-trade --max-trades 1

# Dry run (no real orders)
uv run main.py --start-minutes 1 --end-minutes 6 --auto-trade --test-only
```

### Manual mode

```bash
uv run main.py --manual-trade --start-minutes 1 --end-minutes 6
```

### CLI options

| Flag | Description | Example |
|------|-------------|---------|
| `--start-minutes` | Min minutes to resolution | `--start-minutes 1` |
| `--end-minutes` | Max minutes to resolution | `--end-minutes 6` |
| `--auto-trade` | Enable auto trading | `--auto-trade` |
| `--manual-trade` | Manual trading | `--manual-trade` |
| `--test-only` | No real orders | `--test-only` |
| `--max-trades` | Max trades in run | `--max-trades 1` |

## Project layout

```
polymarket/
├── main.py                    # Entry point
├── advanced_scheduler.py     # Advanced scheduler
├── scheduler_config.json     # Scheduler config
├── config.example.env        # Example env
├── src/
│   ├── polymarket_scanner.py # Market scanner
│   ├── polymarket_trader.py  # Order execution
│   ├── auto_trader.py        # Auto trader
│   ├── manual_trader.py      # Manual trader
│   ├── balance_checker.py   # Balance checks
│   └── polymarket_tokenid.py
├── QUICK_START.md
└── README.md
```

## Core components

### PolymarketScanner

- Finds markets about to end
- Pulls order book / market data
- Filters by time window

### PolymarketTrader

- Market orders
- Balance and allowance handling
- Order status

### AutoTrader

- Decides when to trade
- Risk checks
- Execution

### AdvancedScheduler

- Time-based scheduling
- Retries
- Stats

## Example output

```
=== Scanning markets ending in 1–6 minutes ===
Strategy: start analysis 4 minutes before end

Found 2 markets ending in 1–6 minutes:
 1. btc-updown-15m-1761011100 - Bitcoin Up or Down
   Resolves: 2025-10-21T02:00:00Z (3m 22s left)
 2. eth-updown-15m-1761011100 - Ethereum Up or Down
   Resolves: 2025-10-21T02:00:00Z (3m 22s left)

Found 1 opportunity:
1. btc-updown-15m-1761011100 - Bitcoin Up or Down
   Suggestion: BUY_YES
   Reason: YES at 0.94 is in 0.90–0.98; buy Up, ~3.2m left
   Trade executed successfully
```

## Important notes

### Security

1. **Private key**: Keep it secret; never share it.
2. **Funds**: Prefer a dedicated trading wallet.
3. **Test first**: Use `--test-only` before live trading.

### Risk

1. **Trading loss**: You can lose money; use at your own risk.
2. **Volatility**: Short-dated markets can move fast.
3. **Ops**: Network lag or API issues can affect fills.

### FAQ

**Q: Why no trades?**  
A: The bot only trades when a side is in 0.90–0.98. If prices are outside that band, it will not place orders.

**Q: How do I change the strategy?**  
A: Edit `MIN_PRICE_RANGE` and `MAX_PRICE_RANGE` in `.env`.

**Q: How do I stop auto trading?**  
A: Press `Ctrl+C` in the terminal, or set `"test_mode": true` in `scheduler_config.json` to disable real orders.

## Support

If something fails, check:

1. Network / RPC
2. Private key and `FUNDER` address
3. USDC balance
4. `.env` and `scheduler_config.json`

## License

MIT License

---

**Disclaimer**: This project is for learning and research. Trading involves risk; understand it before you use real funds.
