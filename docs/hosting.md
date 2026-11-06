# Hosting

## One instance per node owner

One Ohala instance serves one node owner. It runs next to that owner's LND node
and can only ask that node for invoices.

If one node took payments for other shops, its owner would hold their money.
That is custody and needs a VASP licence. So each shop runs its own instance
against its own node.

## The Altradits instance

Altradits runs one instance at till.altradits.com. It takes only Stanley's own
sales, tips and meetup desks. It is also the reference deployment. It does not
take payments for other shops. It holds no one else's money.

## Running your own

* Bind to localhost by default (`127.0.0.1:8080`). To serve phones on the LAN,
  set `--listen` and either provide a cert or use the self-signed cert Ohala
  generates (like lnd's tls.cert).
* If the node sits behind a tunnel, use a TCP tunnel to the node's RPC port
  (default 10009). Keep the macaroon off shared hosting where you can.
* The Lightning address feature needs a public HTTPS URL. It is off by default.
  The till itself works on a LAN with no public exposure at all.
* Storage is one bbolt file (`ohala.db`, mode 0600) in the data directory
  (`~/.ohala` by default). Settings can export receipts as CSV and download a
  database backup. Back up the file. There is no other state to keep.
