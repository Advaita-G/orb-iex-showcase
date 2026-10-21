# ORB-IEX

A Python opening-range breakout bot for **Alpaca paper trading**, built around reliable order handling and recovery.

The central engineering problem: **keeping local state and broker state consistent when orders partially fill, requests time out, or the process restarts**.

This repository is an engineering case study. [Portfolio](https://advaitagupta.com)

## How it works

1. Filter market data, generate opening-range signals, and apply sizing and exposure checks.
2. Persist order intent and submit through the paper broker adapter.
3. Reconcile fills and order state, maintaining protective orders or flattening when required.
4. Record execution events and expose heartbeat, health, and circuit-breaker state.

```text
Market data + signals
  -> sizing + durable intent
  -> Alpaca PAPER
  -> reconcile + protect
  -> journal + review
```

## Decisions that matter

- **Order ownership:** deterministic client order IDs distinguish bot orders from unrelated account activity.
- **Ambiguous responses:** after a timeout, inspect the existing order before deciding whether to submit again.
- **Partial fills and cancellations:** protection follows filled quantity, and cancellation requires confirmation.
- **Restart recovery:** reconcile durable SQLite state with broker state rather than assuming the previous process finished cleanly.
- **Operational visibility:** heartbeat, health checks, and circuit breakers support supervised paper operation.

## Tests and current evidence

The focused configuration, account-safety, broker-safety, reconciliation, and recovery suites produced **142 passed, 1 failed** during the 2026-10-20 review.

The failure is a historical reconciliation test that expects a fixed old count while reading an evolving local runtime journal. It needs an immutable fixture. The other checked cases include foreign orders, partial fills, uncertain broker outcomes, and restart recovery.

<details>
<summary>Command used in the application checkout</summary>

The application source is private; this documentation repository contains no runnable bot package. In the application checkout, using its existing environment:

```sh
cd orb_iex
.venv/bin/python -m pytest tests/unit/test_config.py tests/unit/test_safety.py tests/unit/test_account_safety.py tests/unit/test_reconcile.py tests/unit/test_recovery.py tests/unit/test_ambiguous_broker.py tests/unit/test_broker_safety.py -q
```

The package declares Python 3.11+, requests, tzdata, and a pytest development extra. These focused offline tests need no keys. Broker-connected checks require a separate paper account and were outside this review.

</details>

## Scope and attribution

This is a port of the supplied C# reference `reference/Main.cs`, developed with substantial Claude and Codex assistance. The strategy is not an original invention; the work described here is the Python integration and execution infrastructure.

Paper fills differ from real execution. Gross P/L, paper-account realized P/L, and modeled costs are separate quantities. This case study reports engineering behavior, not investment performance. Full-suite and fresh broker/session validation remain outstanding. The in-progress journal dashboard is outside the verified scope.

Application source, broker records, credentials, and research data are not distributed here. Reference redistribution provenance remains unresolved.
