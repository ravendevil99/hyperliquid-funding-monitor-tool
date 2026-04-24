# Hyperliquid Funding Monitor

> Tracks funding rates across all Hyperliquid perpetual pairs in real time. Alerts on extreme funding deviations, cross-pair funding arbitrage opportunities, and sustained funding trends that indicate crowded positioning.

*Documentation updated: April 2026*

![preview_hyperliquid-funding-monitor](https://github.com/user-attachments/assets/9f9e1d52-e378-42f9-8a4f-5f2802fc11bf)

## What is Hyperliquid Funding Monitor?

Hyperliquid Funding Monitor continuously reads funding rates for every perpetual pair on Hyperliquid and compares each rate against its own historical baseline. When a pair's funding rate deviates significantly from its norm, it means one side of the market has become crowded - longs are paying shorts (positive funding) or shorts are paying longs (negative funding).

Extreme positive funding on a pair means leveraged longs are dominant. If price has not moved proportionally, this often resolves by either price catching up or longs getting squeezed. Extreme negative funding means leveraged shorts are dominant. The monitor tracks both conditions and alerts when deviation crosses a configurable threshold.

The bot also scans for cross-pair funding divergence - cases where two correlated assets have significantly different funding rates. This creates a carry opportunity: long the negative-funding asset, short the positive-funding asset, collecting the funding spread while running a delta-neutral position.

All funding data is read from Hyperliquid's public REST API. No account or private key required.

---

## Download

| Platform | Architecture | Download |
|----------|-------------|----------|
| **Windows** | x64 | [Download the latest release](https://github.com/ravendevil99/hyperliquid-funding-monitor-tool/releases) |

---

## What Gets Monitored

| Metric | Description | Alert Trigger |
|--------|-------------|---------------|
| Funding rate | Current 8h funding for each pair | Deviates > 2x historical baseline |
| Funding trend | Direction of funding over last 6h | Trending positive/negative for 6h+ |
| Cumulative funding (24h) | Total funding paid/collected over 24h | Exceeds configurable threshold |
| Cross-pair spread | Difference between correlated pair rates | Spread > configurable minimum |

---

## Funding Arbitrage Scanner

The scanner compares funding rates across pairs grouped by asset class (BTC, ETH-correlated, SOL-correlated, HYPE). When two correlated pairs show a funding spread above the minimum threshold, it outputs the long/short pair combination, the spread in annualized percentage, and the estimated daily carry per $1,000 notional.

---

## How It Works

1. **Poll** - fetches current funding rates for all Hyperliquid perp pairs on configurable interval
2. **Baseline** - maintains rolling 7-day baseline for each pair's funding rate
3. **Evaluate** - calculates deviation score and flags pairs above threshold
4. **Scan** - checks cross-pair funding spreads within correlated asset groups
5. **Alert** - sends Telegram alert with funding context, deviation, and carry details if applicable

### Config Reference

```toml
[monitoring]
poll_interval_seconds = 60
deviation_threshold = 2.0      # Alert when funding is 2x the 7-day baseline
trend_window_hours = 6         # Hours of consistent direction to trigger trend alert
cumulative_24h_threshold = 0.3 # Alert when 24h cumulative funding exceeds 0.3%

[arbitrage]
enabled = true
min_spread_annualized_pct = 50 # Minimum annualized spread to flag as opportunity

[assets]
monitor = []                   # Empty = all pairs

[alerts]
telegram_bot_token = ""
telegram_chat_id = ""

[api]
rest_endpoint = "https://api.hyperliquid.xyz"
```

---

## Alert Log Format

```json
{
  "event": "funding_extreme",
  "timestamp": "2026-03-19T16:00:00Z",
  "asset": "SOL",
  "funding_rate_8h": 0.0041,
  "baseline_7d": 0.0011,
  "deviation_multiple": 3.7,
  "direction": "positive",
  "annualized_rate_pct": 222.3,
  "trend_hours": 8,
  "alert_type": "extreme_positive"
}
```

---

## Verified on Hyperliquid Mainnet

**Configuration used:**
* Deviation threshold: 2x baseline
* Arb scanner: enabled, min 50% annualized

**Extreme funding detected:**

| | Details |
|---|---|
| Asset | SOL |
| Funding rate | 0.41% per 8h |
| Baseline | 0.11% per 8h |
| Deviation | 3.7x baseline |
| Annualized rate | 222.3% |

---

## Bot Preview

![funding rate deviation scan](https://github.com/user-attachments/assets/60b6fd96-66e7-466a-9d3f-befcac62c720)

---

<img width="1920" height="1080" alt="funding monitor alert feed" src="https://github.com/user-attachments/assets/d1e2f3a4-b5c6-7890-def0-712345678900" />

---

## Frequently Asked Questions

**What is Hyperliquid Funding Monitor?**
It tracks funding rates for all Hyperliquid perp pairs and alerts when rates deviate significantly from historical norms. Extreme funding often precedes directional price moves as the crowded side gets squeezed.

**What does extreme positive funding mean?**
Leveraged longs are paying shorts. This means LONG positions are dominant. If price has not moved upward proportionally, the crowded longs may get squeezed, causing a sharp downward move.

**What is the funding arbitrage scanner?**
It looks for correlated pairs with significantly different funding rates. Going long the negative-funding pair and short the positive-funding pair collects the spread while maintaining approximate delta neutrality.

**How is the baseline calculated?**
A 7-day rolling average of the funding rate for each pair. The deviation multiple is the current rate divided by this baseline.

**How often does it poll?**
Every 60 seconds by default. Funding rates update every 8 hours on Hyperliquid, but the bot catches the change at once when it occurs.

**Does it track cumulative funding?**
Yes. It tracks the total funding paid or collected over 24 hours per pair and alerts when cumulative funding exceeds the configured threshold.

**Can I monitor specific pairs only?**
Yes. Populate the `monitor` list in config with specific pair tickers. Leave empty to monitor all pairs.

**Does it need a private key?**
No. The monitor is read-only and uses Hyperliquid's public API.

**What is the trend alert?**
When funding for a pair has been moving in the same direction (increasing or decreasing) for a configurable number of hours, it triggers a trend alert indicating sustained crowded positioning.

**How do I use the funding data for trading?**
Extreme positive funding = potential short setup (longs overpaying, squeeze risk). Extreme negative = potential long setup. The arbitrage scanner output is useful for funding carry strategies.

---

## Use Cases

- **Funding extreme alerts** - catch pairs with unsustainably high or low funding rates before the squeeze
- **Carry arbitrage** - identify correlated pairs with spread large enough to collect as carry
- **Crowded trade detection** - monitor which assets have the most lopsided funding as a sentiment indicator
- **Trend tracking** - follow funding trends to see if crowded positioning is increasing or resolving
- **Portfolio hedge** - use funding data to decide which perps to hold vs avoid based on carry cost

---

## Repository Structure

```
hyperliquid-funding-monitor/
+-- hyperliquid-funding-monitor-v.1.4.14.exe
+-- config.toml
+-- data/
|   +-- logs/
|   +-- baseline/
|   +-- history/
+-- python/
|   +-- src/
|   |   +-- poller.py
|   |   +-- baseline.py
|   |   +-- arb_scanner.py
|   |   +-- alerter.py
|   +-- requirements.txt
+-- README.md
```

---

## Requirements

```
httpx, toml, python-dotenv, rich, pandas
```

* Python 3.10+
* Telegram bot token for alerts
* No Hyperliquid account required (read-only)

---

*Automate smarter. Trade sharper.*
