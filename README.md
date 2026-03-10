# 🌤 Weather Trading Bot — Polymarket

Automated weather market trading bot for Polymarket. Finds mispriced temperature outcomes using real station data from NWS.

No SDK. No black box. Pure Python.

---

## Versions

### `bot_v1.py` — Base Bot (current)

The foundation. Scans 6 US cities, fetches forecasts from NWS using airport station coordinates, finds matching temperature buckets on Polymarket, and enters trades when the market price is below the entry threshold.

No math, no complexity. Just the core logic — good for understanding how the system works.

### `bot_v2.py` — Kelly + EV Edition (coming soon)

Everything in v1, plus:

- **Expected Value** — skips trades where the math doesn't work
- **Kelly Criterion** — sizes positions based on edge strength, not a flat %
- **Auto-exit** — closes positions when price hits the exit threshold
- **Live dashboard** — updates `simulation.json` so the dashboard stays current

---

## How It Works

Polymarket runs markets like "Will the highest temperature in Chicago be between 46–47°F on March 7?" These markets are often mispriced — the forecast says 78% likely but the market is trading at 8 cents.

The bot:

1. Fetches forecasts from NWS using airport coordinates
2. Combines real station observations with hourly forecast to get the true daily maximum
3. Finds the matching temperature bucket on Polymarket
4. Enters the trade if market price is below the entry threshold
5. Runs a full $1,000 simulation against real market prices before you risk anything

---

## Why Airport Coordinates Matter

Most bots use city center coordinates. That's wrong.

Every Polymarket weather market resolves on a specific airport station. NYC resolves on LaGuardia (KLGA), Dallas on Love Field (KDAL) — not DFW. The difference between city center and airport can be 3–8°F. On markets with 1–2°F buckets, that's the difference between the right trade and a guaranteed loss.

| City | Station | Airport |
|------|---------|---------|
| NYC | KLGA | LaGuardia |
| Chicago | KORD | O'Hare |
| Miami | KMIA | Miami Intl |
| Dallas | KDAL | Love Field |
| Seattle | KSEA | Sea-Tac |
| Atlanta | KATL | Hartsfield |

---

## Installation

```bash
git clone https://github.com/alteregoeth-ai/weatherbot
cd weatherbot
pip install requests
```

Create `config.json` in the project folder:

```json
{
  "entry_threshold": 0.15,
  "exit_threshold": 0.45,
  "max_trades_per_run": 5,
  "min_hours_to_resolution": 2,
  "locations": "nyc,chicago,miami,dallas,seattle,atlanta"
}
```

---

## Usage

```bash
python bot_v1.py          # paper mode — shows signals, no trades
python bot_v1.py --live   # simulates trades with $1,000 balance
python bot_v1.py --reset  # reset balance back to $1,000
python bot_v1.py --positions  # show open positions and PnL
```

---

## Configuration

| Parameter | Default | Description |
|-----------|---------|-------------|
| `entry_threshold` | `0.15` | Buy below this price |
| `exit_threshold` | `0.45` | Sell above this price |
| `max_trades_per_run` | `5` | Max new trades per scan |
| `min_hours_to_resolution` | `2` | Skip if resolves too soon |
| `locations` | `nyc,...` | Cities to scan (comma separated) |

---

## APIs Used

| API | Auth | Purpose |
|-----|------|---------|
| NWS (api.weather.gov) | None | US city forecasts + station observations |
| Polymarket Gamma | None | Market data |
| Polymarket CLOB | Wallet key | Live trading (optional) |

---

## Live Trading

The bot runs in simulation mode by default. To execute real trades, add Polymarket CLOB integration:

```bash
pip install py-clob-client
```

Then replace the paper mode block in `bot_v1.py` with your CLOB buy function. Full guide in the article linked below.

---

## Disclaimer

This is not financial advice. Prediction markets carry real risk. Run the simulation thoroughly before committing real capital.
