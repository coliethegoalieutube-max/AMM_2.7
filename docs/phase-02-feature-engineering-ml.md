# Phase 2: Feature Engineering & ML Model Build

Predict pregame retail inflow **before** it hits the order book. Prefer tabular ML over latency racing automated market makers.

## 2.1 Universe filter

Exclude low-liquidity small-conference games where whisper risk and wide spreads dominate.

**Include only** games that pass:

- Recognizable programs / power conferences / nationally televised slates
- Strict minimum **volume / depth floor**

Config placeholder: `configs/universe.yaml` (to be defined).

## 2.2 Target label — excess drift

Predict excess pregame drift from **T−30h → T−30m**:

```
excess_drift = ΔKalshi_price − Δsharp_implied_prob
```

- Clip extreme moves **> 10¢** so injury shocks do not distort training
- Stop at **T−30m** to capture the retail peak before late sharp reversion

## 2.3 High-signal features

### Perception vs. analytics gap

- AP Poll rank − KenPom / BartTorvik rank

### Visibility & slate context

- National TV flag (ESPN / CBS / Fox)
- Standalone primetime vs. crowded Saturday slate
- Program prestige / fanbase proxy

### Recency & pricing psychology

- Last-3-game win and ATS streaks
- Ranked-underdog flag (ranked team priced under 50¢)
- Home-underdog status

### Initial market state (at T−30h)

- Kalshi bid–ask spread
- Resting depth
- Initial divergence from sharp-book fair value

## 2.4 Model & validation

| Choice | Spec |
|--------|------|
| Model | Gradient-boosted trees — LightGBM or XGBoost |
| Validation | **Strict walk-forward** — train on earlier seasons/weeks; test only on future slates |
| Forbidden | Random K-fold (leaks future information) |
| Selection | Rank each day's slate by predicted drift; trade only **top 5–10%** conviction |

### Outputs

- Feature matrix → `data/processed/features/`
- Labels → `data/processed/labels/`
- Model artifacts → `src/models/` (or `artifacts/` once introduced)
- Daily ranked slate for backtest / execution

## 2.5 Acceptance criteria (Phase 2 complete when)

- [ ] Universe filter reproducible from config
- [ ] Excess-drift label matches definition with clipping
- [ ] Feature join coverage audited (missingness report)
- [ ] Walk-forward folds defined by time only
- [ ] Top-decile selection wired into backtest entry filter
