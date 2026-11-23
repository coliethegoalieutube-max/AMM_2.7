# AMM 2.7 — Kalshi NCAAB Pregame Herding Strategy

ML-driven pregame herding on Kalshi college basketball markets: predict retail inflow 24–36 hours ahead, execute via passive Maker–Maker limit orders, and flatten before tip-off for zero game-outcome variance.

## Strategy in one line

Identify games that will attract lopsided retail volume, post passive bids early, exit into the drift with decaying asks, and never carry tip-off risk.

## Pipeline phases

| Phase | Focus | Status |
|-------|--------|--------|
| 1 | [Data collection & storage](docs/phase-01-data-collection.md) | Planned |
| 2 | [Feature engineering & ML](docs/phase-02-feature-engineering-ml.md) | Planned |
| 3 | [Microstructure-aware backtesting](docs/phase-03-backtesting.md) | Planned |
| 4 | [Sandbox & live shadow-trading](docs/phase-04-sandbox-shadow.md) | Planned |
| 5 | [Live deployment & risk controls](docs/phase-05-live-deployment.md) | Planned |

See also: [architecture overview](docs/architecture.md), [fee & fill model](docs/fee-and-fill-model.md), [go-live checklist](docs/go-live-checklist.md).

## Repository layout

```
AMM_2.7/
├── docs/                 # Strategy & pipeline documentation
├── configs/              # Strategy, universe, and risk configs
├── data/
│   ├── raw/              # Kalshi, Odds API, context pulls
│   ├── processed/        # Feature tables & labels
│   └── l2_snapshots/     # Live L2 recorder output
├── src/
│   ├── collect/          # Historical backfill + live L2 logger
│   ├── features/         # Feature engineering
│   ├── models/           # Training, inference, walk-forward
│   ├── backtest/         # Maker fill sim, PnL, metrics
│   ├── execution/        # Order placement, amend, WS reconnect
│   └── risk/             # Kill switches, exposure limits
├── scripts/              # CLI entrypoints
├── notebooks/            # Exploratory analysis
└── tests/
```

## Design principles

1. **Split history vs. L2** — Historical trades/candles train the model; live L2 snapshots drive realistic maker backtests.
2. **No mid-price fantasy** — Fills require trade-through or queue depletion evidence.
3. **Zero tip-off variance** — Hard flatten before tip; never hold into game outcome.
4. **Demo ≠ edge** — Demo verifies plumbing; only live shadow-trading validates strategy.
5. **Small capital first** — Scale only after shadow metrics match backtest.

## Next steps

Documentation and workspace layout only. Implementation starts with Phase 1 data collection when ready.
