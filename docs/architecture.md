# Architecture

Ohala is a modular monolith with ports and adapters, kept light. The domain
knows nothing about gRPC, HTTP or the database. That keeps the money maths and
the charge state machine testable with plain `go test`. Only one adapter talks
to LND, so all LND learning happens in one package.

## Shape

```text
browser (cashier/owner) --HTTPS--> internal/web --> internal/till (domain) --> ports
                                                                 |-- Invoices port ---> internal/lnd (gRPC + macaroon) ---> owner's LND
                                                                 |-- Receipts port ---> internal/store (bbolt file)
payer wallet --HTTPS (optional)--> internal/lnurl ----------------/
```

## Module layout

```text
github.com/altradits/ohala
  cmd/ohala/main.go            flags, subcommands: run, init, version, check-macaroon
  internal/config/             flags + env (OHALA_*) + optional ohala.conf; validation; defaults
  internal/till/               domain: Charge, Receipt, state machine (open, paid, expired, cancelled), numbering
  internal/money/              ISO 4217 table (embedded), Rate, fiat to msat integer conversion, formatting
  internal/lnd/                adapter: dial (TLS + macaroon creds), AddInvoice, SubscribeSingleInvoice,
                               LookupInvoiceV2, CancelInvoice, macaroon permission check
  internal/lnurl/              LUD-01 bech32, LUD-06 payRequest, LUD-16 address, LUD-12 comments, metadata + description hash
  internal/store/              bbolt: receipts, settings, devices, sessions; schema version + migrations
  internal/web/                handlers, html/template, sessions, CSRF, CSP, rate limits, pairing
  internal/i18n/               catalog loader, lookup with fallback, plural rules, locale negotiation
  internal/qr/                 QR SVG rendering (wraps one vendored, pinned encoder)
  web/templates/, web/static/  embedded with go:embed (one CSS file, one JS file, no CDN)
  locales/en.json, sw.json ..  one flat JSON catalog per locale (BCP 47 tag)
  tools/i18ncheck/             CI tool: keys, placeholders, plural forms
  tools/aitrailers/            CI helper used by the review-routing workflow
  itest/                       integration tests (build tag itest) against regtest lnd
  dev/                         regtest scripts, Polar how-to, sample ohala.conf
  docs/                        install, lnd setup, macaroon, translating, architecture, ADRs
```

## Dependencies

The list stays short. Each entry needs a reason in `docs/dependencies.md`:

* `github.com/lightningnetwork/lnd/lnrpc` and `lnrpc/invoicesrpc` (LND's own interfaces)
* `github.com/lightningnetwork/lnd/macaroons` (macaroon credentials and ID decoding)
* `google.golang.org/grpc`
* `gopkg.in/macaroon.v2`
* `go.etcd.io/bbolt` (the same store LND grew up on)
* `golang.org/x/text` (plural rules and number formatting)
* `golang.org/x/crypto` (argon2id for the PIN)
* one QR encoder, pinned and vendored

Nothing else without an ADR.

## ADR-001: import lnd/lnrpc directly

Import `lnd/lnrpc` rather than generating from the .proto files or using
`lightninglabs/lndclient`. Gained: contributors read the same types LND uses,
version pinning is simple, and there is no protoc toolchain. Given up: a larger
go.sum (the lnd module graph). The fallback, generating from the protos, stays
open and is easy to switch to later.
