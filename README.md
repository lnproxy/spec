WIP: lnproxy base spec.
=======================

`author: lnproxy` `author: niteshbalusu11` `discussion: https://github.com/niteshbalusu11/lnproxy-ts/issues/2`

---

A user may want to change the destination of an invoice, either because the user needs help finding a path to the destination, because the destination is their public lightning network node which they want to keep private, or because they are paying from a custodial service and don't want the custodian to know the destination of their payment.  In order not to have to trust an intermediary with their funds, these users can request that an lnproxy relay generate a "wrapped" invoice: an invoice with the same `payment_hash`.

Then, once the user verifies the amount and hash in the relay's invoice, the user can use the relay's invoice wherever the original invoice would have been used and know that the only way for the payment to succeed is for the relay to pay the original invoice.

## Requesting and verifying a wrapped invoice

1. User...

## Notes on implementing an lnproxy relay

### Accounting for routing fees

### Setting the `min_final_cltv_expiry`
