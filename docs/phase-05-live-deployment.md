# Phase 5: Live Deployment & Risk Controls

Start with small sizing and monitor post-fill mark-outs. Guardrails protect against the negative skew of a passive drift strategy.

## 5.1 Daily execution loop

1. At **T−30h to T−24h**, run ML inference on the upcoming slate
2. Filter to **top-decile** games meeting the liquidity floor
3. Post **passive maker bids** at the top of the book
4. On bid fill → immediately post resting **target ask**
5. Run glide path + hard flatten + kill switches until flat

## 5.2 Automated defenses

### Sharp-line kill switch

Continuously monitor sharp-book consensus. Immediately cancel **unfilled** bids if sharp implied probability drops by **≥2¢**.

### Glide path & hard exit

- Step down resting ask on the backtested schedule
- Flatten remaining inventory by hard cutoff (e.g. T−15m to T−30m)
- **Never** carry tip-off risk

### Exposure limits

| Limit | Example starting point |
|-------|------------------------|
| Per-game position | 1–2% of bankroll |
| Concurrent open contracts | Cap total inventory |
| Daily max loss | Hard stop for the day |

Config placeholder: `configs/risk.yaml`.

## 5.3 Post-trade mark-out audit

After every live fill, log sharp-book price at:

- **+5 minutes**
- **+30 minutes**

If post-fill mark-outs are **consistently negative**, sharps are picking off resting bids — tighten liquidity floor or kill-switch sensitivity **before** scaling capital.

## 5.4 Acceptance criteria (Phase 5 ready when)

- [ ] Live loop runs unattended for a full slate day in small size
- [ ] All kill switches and hard exits fire in production (tested)
- [ ] Mark-out audit dashboard / log reviewed daily
- [ ] Scale-up criteria documented and met before size increases
