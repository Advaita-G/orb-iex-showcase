# ORB-IEX

A Python opening-range breakout bot for **Alpaca paper trading**, built around reliable order handling and recovery.

The central engineering problem: **keeping local state and broker state consistent when orders partially fill, requests time out, or the process restarts**.

This repository is an engineering case study. The full source and tests are public at [Advaita-G/orb-iex](https://github.com/Advaita-G/orb-iex). [Portfolio](https://advaitagupta.com)

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

Verified on **2026-11-09**, using a clean checkout of public source revision [`9f4025516367`](https://github.com/Advaita-G/orb-iex/tree/9f4025516367797200da6d1f0ae2289eecffa4ad), on macOS with Python 3.13.15:

- **145 passed** in the focused configuration, account-safety, broker-safety, reconciliation, and recovery suites. This supersedes the older 2026-10-20 focused result, including its historical reconciliation failure.
- The checked cases cover unrelated account orders, partial fills, uncertain broker responses, and restart recovery using simulated broker state.
- **Full-suite completion is not verified.** Collection found 887 tests; the broader run was interrupted after 764 passed when it stopped making progress. The isolated `test_restart_at_noon_does_not_rescan_or_duplicate` also did not finish within a 15-second check. A test count is not a guarantee of correctness.

<details>
<summary>Reproduce the focused result</summary>

The source is public at [Advaita-G/orb-iex](https://github.com/Advaita-G/orb-iex). The full collection also needs the optional research dependencies:

```sh
cd orb-iex
git checkout 9f4025516367797200da6d1f0ae2289eecffa4ad
python -m pip install -e ".[dev,research]"
python -m pytest tests/unit/test_config.py tests/unit/test_safety.py tests/unit/test_account_safety.py tests/unit/test_reconcile.py tests/unit/test_recovery.py tests/unit/test_ambiguous_broker.py tests/unit/test_broker_safety.py -q
```

These focused offline tests need no keys. The incomplete broader run used `python -m pytest -q`. Broker-connected checks require a separate paper account and were outside this review.

</details>

## Scope and attribution

This is a port of the supplied C# reference `reference/Main.cs`, developed with substantial Claude and Codex assistance. The strategy is not an original invention; the work described here is the Python integration and execution infrastructure.

Paper fills differ from real execution. Gross P/L, paper-account realized P/L, and modeled costs are separate quantities. This case study reports engineering behavior, not investment performance. Full-suite completion and fresh broker/session validation remain outstanding. The journal dashboard is outside the verified scope of this test run.

Application source is available in the linked repository. Broker records, credentials, private research data, and the supplied C# reference are not distributed in this case study.
