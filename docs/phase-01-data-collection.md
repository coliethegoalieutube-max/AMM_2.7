# Phase 1: Data Collection & Storage Pipeline

Backfill historical odds and trades while logging live L2 order books immediately. Historical trade logs do not capture resting depth — split collection into a **historical backfill** (model training) and a **live L2 recorder** (realistic maker backtesting).

## 1.1 Kalshi historical backfill

| Step | Endpoint / action | Notes |
|------|-------------------|--------|
| Cutoff | `GET /historical/cutoff` | Boundary between live and archived records |
| Markets | `GET /historical/markets` | Settled NCAAB markets |
| Candles | `GET /historical/markets/{ticker}/candlesticks` | 1-minute and/or 60-minute |
| Trades | `GET /historical/trades` | Trade prints |
| Fees | Persist per series | Store `fee_type` and `fee_multiplier` for cost model |

### Storage targets

- `data/raw/kalshi/markets/`
- `data/raw/kalshi/candlesticks/`
- `data/raw/kalshi/trades/`
- Series metadata (including fee fields) keyed by market/series ticker

## 1.2 Sportsbook benchmarks (The Odds API)

- Sport key: `basketball_ncaab`
- Historical data available back to ~2020
- Pull time-stamped moneylines and spreads from:
  - **Sharp books** — e.g. Pinnacle
  - **Retail books** — e.g. DraftKings
- Snapshot horizons relative to tip-off:
  - **T−36h, T−24h, T−12h, T−2h, T−30m**

### Storage target

- `data/raw/odds_api/` — event-aligned snapshots keyed by tip time and book

## 1.3 Herding & context data

Pull and store:

- Historical AP Poll rankings
- Efficiency ratings (KenPom or BartTorvik)
- Team win / ATS streaks
- Home / away status
- Conference tier
- National TV broadcast schedules

### Storage target

- `data/raw/context/`

## 1.4 Live L2 order book logger

- Connect to Kalshi production WebSocket: `wss://external-api-ws.kalshi.com/trade-api/ws/v2` (confirm against current Kalshi docs)
- Subscribe: `orderbook_delta`, `trade` for upcoming NCAAB markets
- Snapshot **top 3–5 bid/ask levels** every **1–5 minutes**
- Start logging from **~36 hours pre-tip**
- Purpose: clean dataset for queue-position simulation

### Storage target

- `data/l2_snapshots/` — time-stamped depth + trade events per ticker

## 1.5 Acceptance criteria (Phase 1 complete when)

- [ ] Historical cutoff → markets → candles → trades pipeline runs idempotently
- [ ] Fee metadata stored for every series used in backtest
- [ ] Odds API snapshots exist at all five horizons for training seasons
- [ ] Context tables joinable to markets on tip date / team keys
- [ ] Live L2 logger runs continuously for upcoming NCAAB slate with reconnection
- [ ] Raw schemas documented and versioned under `docs/` or `configs/`
