# Hedge_optima Architecture Flow (Summary)

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

