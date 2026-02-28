# PUBLIC_SHARE_CHECKLIST.md - Kem Upyog Karvo (Simple Gujarati Guide)

`PUBLIC_SHARE_CHECKLIST.md` tamara mate chhe (internal). Client ne aa file normally moklvi jaruri nathi. Aa file thi tame ensure karo ke koi secret public ma leak na thay.

## 1) Share Karva Pahela Quick Check (1 minute)
- Screenshot/log ma `DATABASE_URL`, `REDIS_URL`, `JWT_SECRET_KEY`, broker `api_key/api_secret/access_token`, `client_id`, `password/mpin`, `totp_key`, Telegram tokens na dekhay.
- Account number / client id blur/mask kari do.
- Railway variables output screenshot kyarey share na karo.

## 2) Performance Proof (Backtest + Live) Kevi Rite Share Karvu
- Backtest: period, trade count, fees/slippage assumption, net return, max drawdown.
- Live: broker statement screenshot (IDs masked) + summary P&L/drawdown.
- Hamesha line add karo: "Past performance does not guarantee future results."

## 3) Safe Demo Rule
- Best: `PAPER` mode / demo environment.
- Real mode demo karvo hoy to: chhota qty + strict risk on.
- Terminal/screenshot ma secrets hide.

## 4) Copy-Paste NDA Ask (Simple English)
Before sharing code/results, can we proceed under NDA? I will share a redacted demo, architecture overview, and performance proof (account details masked).

