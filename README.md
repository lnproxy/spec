WIP: lnproxy spec.
==================

`author: lnproxy` `author: niteshbalusu11` `discussion: https://github.com/niteshbalusu11/lnproxy-ts/issues/2`

---

A user may want to use a proxy destination for an invoice, either because the user needs help finding a path to the original destination, because the original destination is their public lightning network node which they want to keep private, or because they are paying from a custodial service and don't want the custodian to know the destination of their payment.  In order not to have to trust an intermediary with their funds, these users can request that an lnproxy relay generate a "wrapped" invoice: an invoice with the same payment hash.

Then, once the user verifies the amount and payment hash, the user can use the wrapped invoice wherever the original invoice would have been used and know that the only way for the payment to succeed is for the relay to pay the original invoice.

## Requesting and verifying a wrapped invoice

1. User...

## Notes on implementing an lnproxy relay

### Accounting for routing fees

### Setting the `min_final_cltv_expiry`
