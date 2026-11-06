# Security model

## Assets, in order

1. The owner's node funds.
2. The integrity of the shown invoice: the customer pays the shop, not someone else.
3. Owner settings, above all the rate and the node connection.
4. Receipt history (business privacy).
5. Availability of the till.

## Trust boundaries

* The cashier device is semi-trusted: it can create and cancel charges and see its own receipts.
* The owner is trusted. Owner-only actions need the PIN again (step-up) even inside a session.
* The internet is untrusted, and reaches only `/lnurlp` and `.well-known` when the address is on.
* LND is trusted (it is the owner's).

## Threats and answers

| # | Threat | Answer |
|---|---|---|
| T1 | Server compromised, attacker uses the macaroon to spend | Macaroon has only invoices:read/write; startup guard refuses broader ones; optional IP constraint; revoke with `deletemacaroonid` |
| T2 | Attacker swaps the QR for their own invoice (XSS, injected script, malicious proxy) | Strict CSP (`default-src 'self'`, no inline script, no external origins), html/template auto-escaping, HTTPS, QR rendered server-side from the stored payment request; the Pay page also shows the amount and short hash next to the QR |
| T3 | Cashier undercharges by changing the rate | Rate and currency are owner-only (step-up PIN); each receipt snapshots the rate used |
| T4 | Invoice spam through public LNURL bloats the node's DB | LNURL off by default; per-IP rate limit (30/min), global cap on open LNURL invoices (500), short LNURL expiry (10 min), LND invoice GC options documented |
| T5 | Brute-force of owner PIN or pairing code | Argon2id PIN hash; 5 failures mean a 15 min lockout per IP and globally per account; pairing codes single-use, 5 min |
| T6 | Session theft / CSRF | Cookies `HttpOnly; Secure; SameSite=Strict`; CSRF token on every POST; sessions revocable per device; idle expiry 30 days for cashier, 12 h for owner step-up |
| T7 | Wrong network (regtest till pointed at mainnet node or reverse) | Network from invoice prefix stored on first connect; mismatch stops charging until the owner confirms in Settings |
| T8 | Mismatched invoice from LND or a bug (amount or description hash differ) | Decode the returned payment request and compare amount, hash and description hash before showing it; on mismatch refuse and log |
| T9 | Secrets leak in logs or backups | Logs never include macaroon, preimage, PIN, session token; macaroon and DB files must be 0600 or startup warns; backup download is owner-only |
| T10 | Lost or stolen cashier phone | Owner revokes the device; it can only create invoices that pay the owner, so theft gains nothing |
| T11 | Supply chain (malicious dependency) | Short dependency list, `go.sum` pinned, Dependabot for Go modules and Actions, `govulncheck` in CI, no `curl \| sh` in docs |
| T12 | LAN exposure | Default listen `127.0.0.1:8080`; to serve phones, owner sets `--listen` and either provides a cert or uses the self-signed cert Ohala generates (like lnd's tls.cert) |

## The minimum limits (and nothing more)

1. Invoice-only macaroon, enforced at startup.
2. No default password; one-time setup URL.
3. Strict CSP and no third-party origins.
4. Owner step-up for settings; PIN lockout.
5. Rate limits on public LNURL routes only.
6. Maximum charge per invoice (owner-adjustable).
7. Localhost bind by default.

Everything else (Tor, client certs, 2FA, audit logs) waits for an issue that
shows the need. Contributors on regtest see none of this friction: `make dev`
writes a config with a regtest macaroon and a local PIN `000000`, and only
accepts it when the network is regtest.
