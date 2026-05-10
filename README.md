# kite-mcp-billing

[![Go Reference](https://pkg.go.dev/badge/github.com/algo2go/kite-mcp-billing.svg)](https://pkg.go.dev/github.com/algo2go/kite-mcp-billing)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Billing engine + Stripe checkout + tier middleware for the algo2go
ecosystem. Provides subscription tier management (Free / Trader /
Premium), Stripe Checkout + Customer Portal flows, webhook handling
(checkout.session.completed, customer.subscription.updated/deleted),
tier-aware MCP middleware for tool gating, and event emission via
`algo2go/kite-mcp-domain.TierChangedEvent`.

Used by [`Sundeepg98/kite-mcp-server`](https://github.com/Sundeepg98/kite-mcp-server)
for tier-gated MCP tools (Telegram trading, trailing stops, MF
orders, native alerts), Stripe billing flows, and admin tier
overrides.

## Why a separate module?

Billing is a substantial commercial-tier surface (~6K LOC) that
unrelated algo2go projects (broker dashboards, premium analytics,
future trading bots) may need independent of `kite-mcp-server`.
Hosting as a module:

- Centralizes the tier definition + Stripe integration across consumers
- Lets billing logic + tier semantics version independently
- Pairs cleanly with `algo2go/kite-mcp-domain` (Money + TierChangedEvent),
  `kite-mcp-alerts` (Store backend), `kite-mcp-oauth` (admin gating),
  `kite-mcp-logger` (structured logging) for the full commercial stack

## Closes original Path A.8 halt

This module's promotion was originally **halted at Path A.8**
(commit 71f17eb in kite-mcp-server) due to a 5+ internal-dep
cluster (templates + domain + alerts + users + oauth all in-tree).
Path A.9 through A.13 unblocked each cluster member sequentially:

- Path A.8' (kc/templates @ 1db565a)
- Path A.10 (kc/domain @ 9ee8212)
- Path A.11 (kc/alerts @ fd9d9fb)
- Path A.12 (kc/users @ e96b1c0)
- Path A.13 (oauth @ 6f2a2b0)

This is **Path A.14** — the FINAL step that closes the chain.

## Stability promise

**v0.x — unstable.** Type signatures may evolve as billing patterns
mature. Pin `v0.1.0` deliberately. v1.0 ships only after the public
API is reviewed for stability and at least one external consumer
ships against it.

## Install

```bash
go get github.com/algo2go/kite-mcp-billing@v0.1.0
```

## Public API (selected)

### Tiers
- `Tier` (int): TierFree, TierTrader, TierPremium
- `TierMonthlyINR(t Tier) domain.Money` — pricing
- `TierName(t Tier) string` — display name

### Store
- `Store` — subscription CRUD with `*alerts.DB` backend
- `NewStore(db *alerts.DB) *Store`
- `Store.SetEventDispatcher(d *domain.EventDispatcher)` — emits
  `domain.TierChangedEvent` on subscription changes

### Stripe integration
- `CheckoutSession(ctx, email, tier) (url, error)` — initiates
  Stripe Checkout for tier upgrade
- `CustomerPortal(ctx, email) (url, error)` — Stripe billing portal
- `WebhookHandler(...)` — verifies + dispatches Stripe webhooks

### Middleware
- `TierGate(minTier Tier) mcp.Middleware` — gates MCP tools by tier

## Dependencies

- `github.com/algo2go/kite-mcp-alerts` v0.1.0 — Store backend
- `github.com/algo2go/kite-mcp-domain` v0.1.0 — Money + TierChangedEvent
- `github.com/algo2go/kite-mcp-logger` v0.1.0 — structured logging
- `github.com/algo2go/kite-mcp-oauth` v0.1.0 — admin gating + JWT context
- `github.com/algo2go/kite-mcp-broker, kite-mcp-isttz, kite-mcp-money,
  kite-mcp-templates, kite-mcp-users` v0.1.0 (transitive)
- `github.com/stripe/stripe-go/v82` — Stripe SDK
- `github.com/mark3labs/mcp-go` — MCP middleware contract

All algo2go deps are published modules; no upstream `replace`
directives needed.

## Reference consumer

[`Sundeepg98/kite-mcp-server`](https://github.com/Sundeepg98/kite-mcp-server)
— consumed by:
- `app/wire.go` — service wiring
- `app/providers/billing.go` — Fx provider for the DI graph
- `mcp/admin/admin_billing_tools.go` — admin tier override tools
- `kc/ops/admin_edge_billing_test.go` — admin dashboard tests
- 37 files total reference kc/billing types

## License

MIT — see [LICENSE](LICENSE).

## Authors

Original design: [Sundeepg98](https://github.com/Sundeepg98) (Zerodha
Tech). Multi-module promotion (2026-05-10): algo2go contributors.
