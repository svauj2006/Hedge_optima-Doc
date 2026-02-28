# Public Share Checklist (Repo Remains Private)

Use this checklist to show Hedge_optima on a profile (Upwork/LinkedIn/Resume) without making the trading source code public.

## 1) Secrets & Credentials (Must Not Leak)
- Never share: `DATABASE_URL`, `REDIS_URL`, `JWT_SECRET_KEY`.
- Never share broker credentials: `api_key/api_secret`, `access_token`, `client_id`, `password/mpin`, `totp_key`.
- Never share Telegram tokens, admin tokens, or callback URLs that include secrets.
- Rotate keys/tokens before any demo recording if they were ever shown in logs/screenshots.

## 2) Logs & Screenshots (Sanitize)
- Remove/blur: user IDs, broker client IDs, phone numbers, tokens, connection strings, internal hostnames.
- Prefer showing: status transitions (`RECEIVED -> PROCESSING -> DONE/FAILED`), token sync counts, idempotency behavior (`duplicate`).

## 3) Performance Proof (Share Safely)
- Always separate: backtest results vs live trading results.
- Include assumptions: brokerage/fees, slippage, latency, sample period, and data source.
- Share metrics instead of guarantees: trade count, net return, max drawdown, and a risk-adjusted ratio (e.g., Sharpe).
- Always add: "Past performance does not guarantee future results."
- If sharing statements/screenshots: mask account numbers, client IDs, and personal details.

## 4) Safe Demo Mode
- Use `PAPER` mode or a dedicated demo environment.
- Keep risk controls enabled (market time, kill switch, circuit breaker).
- Use synthetic signals (dummy order numbers) when possible.

## 5) What You Can Share Publicly (High Value, Low Risk)
- Architecture overview (flow diagram + short narrative).
- Key invariants: deterministic hedge sequencing, idempotent execution, scheduler windowing, token snapshot strategy.
- Sample redacted payload schema (remove IDs/keys).
- A 2-3 minute screen recording (no secrets on screen).

## 6) What To Avoid Sharing
- Full source code, Railway variables output, database exports.
- Raw broker error responses containing account details.

## 7) Case Study Template (Paste Into Profile)
- Problem: manual signals needed reliable execution + audit + safety controls.
- Solution: REST ingestion -> validation -> pair engine -> risk gate -> idempotent queue worker -> broker adapters.
- Reliability: distributed locks, retries, circuit breaker, daily ops automation.
- Security: env-based secrets, log sanitization, role-based admin actions.
- Outcome: reduced operational risk + faster execution with full traceability.

