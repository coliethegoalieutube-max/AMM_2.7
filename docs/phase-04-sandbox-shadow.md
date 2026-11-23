# Phase 4: Sandbox Testing & Live Shadow-Trading

Separate **technical verification** (demo) from **strategy validation** (live paper). Demo books do not contain real retail flow or sharp adverse selection.

## 4.1 Demo API verification

Connect the execution bot to Kalshi **demo**:

- WebSocket: `wss://external-api-ws.demo.kalshi.co/trade-api/ws/v2` (confirm against current docs)
- Demo REST endpoints

### Plumbing checklist

- [ ] RSA request signing
- [ ] Order creation
- [ ] Order amendment
- [ ] Kill-switch cancellations
- [ ] Automatic WebSocket reconnection with exponential backoff

## 4.2 Live shadow-trading

- Switch to **production** WebSocket feed in **paper-trading** mode
- Run **2–4 weeks** of live college basketball slates
- Log every simulated:
  - Bid placement
  - Trade-through fill
  - Decaying ask step-down
  - Emergency / hard-cutoff exit
- All marks against **real-time** order books (not demo)

## 4.3 Go-live gate

Move to live capital **only when** shadow-trade:

- Fill rates
- Adverse-selection drag
- Net PnL

…stay consistent with backtest expectations.

Detailed checklist: [go-live-checklist.md](go-live-checklist.md).

## 4.4 Acceptance criteria (Phase 4 complete when)

- [ ] Demo suite passes all plumbing checks above
- [ ] Shadow logger writes a complete audit trail per game
- [ ] Side-by-side shadow vs. backtest report reviewed
- [ ] Explicit go / no-go decision recorded before capital
