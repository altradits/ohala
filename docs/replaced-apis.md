# What the replaced APIs would need

Ohala talks to no outside service. The pay path is payer wallet to Lightning to
the owner's node. If an owner wants to automate any of the services below later,
this page states the contract their own self-hosted piece must meet. Ohala
never depends on it being up.

## Fiat rate

Default: the owner types the price of 1 BTC in Settings. A timestamp is kept.
After the maximum age the till falls back to sats.

Optional file source: a local file `rates.json`, written by whatever the owner
trusts (their own script, an exchange account they hold, a price agreed
offline):

```json
{"currency":"KES","btc_price_minor":1300000000,"as_of":"2026-11-06T09:00:00Z","source":"owner-script"}
```

Rules:

* Ohala reads the file on change.
* It rejects non-positive prices, unknown currencies and `as_of` in the future.
* It ignores a file older than the maximum age.
* It rejects a jump of more than 20% from the last accepted rate until the owner confirms in Settings.
* It never fetches anything itself.

A future self-hosted `ohala-price` module (separate repo, separate binary) may
write this file from the owner's own sources. Required inputs: list of sources
the owner configures; method: median of at least 3; output: the file above; it
must sign nothing and hold no keys.

## Fiat cash-out

Not part of Ohala. A till that converts to mobile money is a custodial or
payment-processor service under Kenya's VASP Act. If ever built, it must be a
separate service holding its own licence.

Its interface would be: input an amount in msat and a destination (phone number
with country code); output a BOLT11 invoice whose payment delivers an exact fiat
amount, plus a status endpoint (pending, completed, failed, with a provider
receipt code).

An owner can still point their own wallet at any cash-out they like. Ohala does
not know or care.

## Buying sats

Out of scope: Ohala only receives. A buy service would need input of a fiat
amount and a payer reference, and output of a Lightning payment to the buyer's
own invoice or address, with a published rate and fee. That is a licensed
exchange function and does not belong in a till.

## LNURL and Lightning address hosting

Built in-house. It needs only a public HTTPS URL for the instance and the same
invoice-only macaroon. No wallet-provider host is used.
