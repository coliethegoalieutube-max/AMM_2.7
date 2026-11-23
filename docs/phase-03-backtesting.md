# Phase 3: Microstructure-Aware Backtesting

Model maker fill realism, exact fees, and decaying exits. A 4¢–5¢ pregame drift can go negative if adverse selection, fees, and pre-tipoff slippage are ignored.

## 3.1 Conservative fill logic

**Never** assume mid-price fills.

Mark a passive maker bid as filled **only when**:

1. Subsequent prints trade **strictly through** your bid price, **or**
2. L2 snapshots show the resting queue ahead of you is **depleted**

Use live L2 recorder data (`data/l2_snapshots/`) for queue-position simulation.

## 3.2 Exact fee schedule

Apply Kalshi’s quadratic fee formula. Full detail: [fee-and-fill-model.md](fee-and-fill-model.md).

| Role | Cost (standard) | Notes |
|------|-----------------|--------|
| Taker | `round_up(M × 0.07 × C × P × (1 − P))` | Up to ~$1.75 per 100 contracts at 50¢ |
| Maker (standard series) | $0 | Confirm per series metadata |
| Maker (`quadratic_with_maker_fees`) | 25% of taker rate | Up to ~$0.44 per 100 contracts |
| Cancel | Free | |

Always use stored `fee_type` / `fee_multiplier` from Phase 1.

## 3.3 Parameters to tune

| Parameter | Candidates |
|-----------|------------|
| Entry window | T−36h vs. T−24h |
| Initial ask target | +3¢, +4¢, or +5¢ above entry fill |
| Ask glide path | Step down at T−2h, T−1h, T−30m |
| Hard flatten cutoff | T−30m, T−15m, T−5m (**avoid T−1m**) |
| Sharp-line kill switch | Cancel unfilled bids if sharp consensus drops ≥2¢ |

Config placeholder: `configs/strategy.yaml`.

## 3.4 Core metrics

Evaluate:

- Net PnL after fees and slippage
- Fill rate on **winning vs. losing** predictions (adverse selection)
- Sharpe ratio
- Max drawdown
- Capital turnover

## 3.5 Acceptance criteria (Phase 3 complete when)

- [ ] Fill engine uses trade-through / queue depletion only
- [ ] Fees match series `fee_type` and documented formula
- [ ] Parameter grid searchable from config
- [ ] Metrics report includes adverse-selection split
- [ ] Backtest results reproducible from a fixed data snapshot
