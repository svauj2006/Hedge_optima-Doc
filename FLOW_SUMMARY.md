- # Hedge_optima Architecture Flow (Summary)
-
## Summary
Hedge_optima is a private, production-oriented robotic trading platform focused on hedge-style options execution. It converts strategy signals (e.g., Excel-driven hedge pairs) into validated, idempotent, risk-gated broker orders, with full audit logs and operational automation (token sync, day resets, and scheduled cleanup). This document is portfolio-safe while keeping the source private.

## What It Does
- Ingests structured signals via REST (including combined HDGE pair payloads).
- Normalizes/validates signals into a consistent schema for processing.
- Executes hedge pairs (ENTRY/EXIT) with deterministic leg ordering.
- Enforces money management and risk controls (session checks, kill switch, loss limits, circuit breaker).
- Resolves broker-specific instrument tokens from a DB snapshot.
- Executes via a Redis-backed queue worker with DB idempotency and retries.
- Maintains end-to-end traceability from raw signal to execution status.

## Key Workflows
- Signal ingestion (`/signal`): stores canonical payloads in `signal_logs`.
- Source poller: picks pending signals, applies distributed locks, updates statuses.
- Pair engine: interprets compound HDGE events and routes ENTRY/EXIT correctly.
- Execution engine + order worker:
  - Builds locked execution payload (NIFTY + NFO + MIS + MARKET).
  - Writes idempotency state to `order_execution_log`.
  - Retries failures and can trigger a circuit breaker (global block).
- Token sync (daily snapshot):
  - Downloads broker masters, filters to `NIFTY only`.
  - Filters to `NOW + NEXT expiry` and strike band (NSE NIFTY previous close +/- 20 strikes).
  - Upserts into `instrument_tokens` for fast runtime resolution.
- Morning jobs (IST scheduling):
  - Daily ops: token refresh, mass login, day-lock reset, and controlled cleanup.
  - Cleanup window is bounded (08:00-09:15 IST) and excludes protected tables.

## Tech Stack
- Backend: Python, FastAPI
- Data: PostgreSQL (audit + state), Redis (locks + queues)
- Infra: Railway (service + Postgres + Redis)
- Parsing: pandas (broker master processing)

## Data Model (Selected)
- `signal_logs`: raw intake + parsed payload + processing status
- `instrument_tokens`: canonical symbol -> broker token snapshot
- `order_execution_log`: idempotency + execution audit (`PROCESSING/EXECUTED/FAILED`)
- `hedge_pairs`, `hedge_legs`, `active_trades`: hedge lifecycle and trade state
- `system_config`: ops configuration (e.g., expiry override)

## Reliability & Ops Highlights
- Distributed locks on signal processing (safe under multi-worker scale).
- Idempotency layer to prevent duplicate executions.
- Deterministic hedge sequencing for ENTRY and EXIT.
- Circuit breaker can globally block execution to protect capital.

## Performance Evidence (Private / On Request)
Backtests and live trading results can be shared privately (redacted where needed).
- Backtest: period, assumptions (fees/slippage), trade count, net return, max drawdown, key ratios.
- Live: date range, broker statement screenshots (account IDs masked), and summary P&L/drawdown.
Note: past performance does not guarantee future results.

## Availability
- Private repository (source shared only under NDA or private screening).
- Can provide: architecture diagram, redacted logs, and a demo walkthrough video.
- Can provide (private): backtest pack and live proof pack (redacted).
-


This document explains the end-to-end flow at a high level. The core source code and credentials remain private.

## 1) Signal Ingestion Layer
- Source: Excel sheet / strategy engine generates signals (single leg or HDGE pair).
- Entry point: `POST /signal` (FastAPI).
- Audit: raw + normalized payload is stored in `signal_logs` for traceability.

## 2) Validation & Pair Engine
- Normalization: converts incoming payload into a standard schema the backend can process.
- Distributed locks (Redis): ensures the same signal is not processed twice in parallel.
- Idempotency: checks execution history so duplicate orders are not re-fired.
- Pair handling: for HDGE, ensures both legs exist (BUY + SELL) and sets deterministic sequence for ENTRY/EXIT.

## 3) Risk & Rule Engine (Money Management)
- Circuit breaker: if repeated failures or unsafe conditions occur, execution is globally halted to protect capital.
- Loss limits: enforces per-user risk rules (from settings) and system-level protection.
- Session gate: allows execution only during valid market session windows.
- Kill switch / day lock: user-level disabling or locking can block execution instantly.

## 4) Instrument Token Resolver
- DB lookup: resolves broker-specific instrument tokens from `instrument_tokens`.
- Snapshot model: token sync maintains a filtered snapshot (NIFTY only, NOW + NEXT expiry, strike band).
- Purpose: broker APIs require token/identifier (not just symbol text) for order placement.

## 5) Order Execution & Worker
- Queue: orders are pushed into Redis queue to control load and enable retries.
- Worker: consumes the queue and places broker MARKET orders via broker adapters (Zerodha/Angel/Upstox/Dhan).
- Audit log: each attempt is recorded in `order_execution_log` with status (`PROCESSING/EXECUTED/FAILED`).

