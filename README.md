# Ohala

A Lightning till for your own LND node. It can create invoices and nothing else.

Status: early. There is no code yet. To help, pick a [good first issue](https://github.com/altradits/ohala/issues?q=is%3Aissue+is%3Aopen+label%3A%22good+first+issue%22).

## What it is

Ohala is a self-hosted till (point of sale) in Go. It runs next to the owner's own LND node and talks to that node only. A cashier types an amount, shows a QR code, and the till watches the invoice settle. Money moves from the payer straight to the owner's node.

* One small binary. One screen. No third-party APIs.
* The till holds an invoice-only macaroon. It cannot spend.
* Every currency. Community translations from day one.

## What it is not

* Not a wallet. It never sends, never opens channels, never holds keys.
* Not multi-merchant. One instance serves one node owner.
* Not an exchange. No mobile money, no buying, no conversion.

Altradits will run one instance for its own sales. It will not take payments for other shops.

## Who it is for

Shops, stalls, cafes, meetups and donation desks that run an LND node. Ohala is built for the world-wide Bitcoin community.

## Tech stack

* Go, the same language as LND.
* LND over gRPC (`lnrpc`, `invoicesrpc`) with an invoice-only macaroon.
* bbolt for local storage, in one file.
* Plain HTML rendered by the same binary, with one small JavaScript file. No frameworks, no build step, works on low-end phones.
* LNURL-pay and Lightning addresses served by Ohala itself.
* Translations in plain JSON files, one per language.
* Local development on regtest with Polar.

The design is in [docs/architecture.md](docs/architecture.md), the LND setup in [docs/lnd.md](docs/lnd.md), and the security model in [docs/security.md](docs/security.md).

## Requirements

Go and an LND node. Exact versions will be pinned when the first code lands.

## Install

There is nothing to install yet.

## Run

There is nothing to run yet. The plan is `ohala init`, then `ohala run`. The design is in `docs/`.

## Develop

There is no code yet. Read the starter issues in [M1: first sale on regtest](https://github.com/altradits/ohala/milestone/1), starting with "Start here".

## Languages

English first, then Kiswahili. See `docs/i18n.md` for how to add a language.

## Contributing

Code, tests and translations must be your own work. Every commit message is one line. Start with [CONTRIBUTING.md](CONTRIBUTING.md).

## Security

See [SECURITY.md](SECURITY.md). Do not open public issues for vulnerabilities.

## Licence

MIT. See [LICENSE](LICENSE).
