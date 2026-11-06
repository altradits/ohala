# LND setup

Ohala talks to the owner's own LND node over gRPC. It uses an invoice-only
macaroon. This page shows how to bake that macaroon and why it matters.

## Transport

gRPC on the node's RPC port (default 10009) with the node's `tls.cert`, and
macaroon credentials attached per call. REST is not used. One transport keeps
the code small. For a node behind a tunnel, use a TCP tunnel to 10009.

## The RPCs the till uses, and nothing else

| RPC | Service | Permission | Used for |
|---|---|---|---|
| `AddInvoice` | lnrpc.Lightning | invoices:write | Create a charge (`value_msat`, `memo`, `expiry`; `description_hash` for LNURL; `private: true` when the owner enables route hints) |
| `SubscribeSingleInvoice` | invoicesrpc.Invoices | invoices:read | Watch one open charge until it is settled, cancelled or expired |
| `LookupInvoiceV2` | invoicesrpc.Invoices | invoices:read | Reconcile open charges after a restart or a dropped stream |
| `CancelInvoice` | invoicesrpc.Invoices | invoices:write | Cashier taps Cancel |
| `ListInvoices` (`num_max_invoices=1`) | lnrpc.Lightning | invoices:read | Settings Test button (proves the connection and the macaroon) |

## Bake the macaroon

Run these on the node. They use a separate root key ID so the owner can revoke
the till without touching admin.macaroon or any other app.

```sh
lncli bakemacaroon --root_key_id 7 --save_to ./ohala.macaroon invoices:read invoices:write
lncli printmacaroon --macaroon_file ./ohala.macaroon     # check it lists only those two
# optional: lock it to the till's IP
lncli constrainmacaroon --ip_address 203.0.113.10 ./ohala.macaroon ./ohala-ip.macaroon
# revoke later (kills every macaroon with root key id 7)
lncli deletemacaroonid 7
```

`invoices:read` lets the till watch invoices and check the connection.
`invoices:write` lets it create and cancel invoices. Nothing else.

## Why invoice-only

A till that cannot spend cannot lose the owner's money. If the server is
compromised, the attacker holds a macaroon that can only make invoices that pay
the owner. That is the most important limit in this project.

## Permission guard

At startup and on Settings Test, Ohala decodes the macaroon ID locally (the
same data `lncli printmacaroon` shows) and refuses to start if it grants
anything beyond `invoices:read` and `invoices:write`. Pointing it at
admin.macaroon gives a clear error naming the extra permissions. The flag
`--unsafe-allow-broad-macaroon` exists for odd setups and logs a warning on
every start.

Known limit: `invoices:read` lets the till read every invoice on the node, not
only its own. LND has no per-app invoice scoping. Ohala only ever queries
hashes it created. The docs state plainly that the till can see the node's
invoice history.

## Network detection

Network detection needs no extra permission. The `payment_request` prefix from
the test invoice (`lnbc`, `lntb`, `lntbs`, `lnbcrt`) tells mainnet, testnet,
signet or regtest. The till stores it and refuses to run if the network changes
under it.

## Recommended lnd.conf lines

These keep expired and cancelled charges from piling up. They are documented,
not enforced:

```text
gc-canceled-invoices-on-startup=true
gc-canceled-invoices-on-the-fly=true
```
