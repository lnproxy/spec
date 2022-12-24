WIP: lnproxy spec.
==================

`author: lnproxy` `author: niteshbalusu11` `discussion: https://github.com/niteshbalusu11/lnproxy-ts/issues/2`

---

A user may want to use a proxy destination for an invoice, either because the user needs help finding a path to the original destination, because the original destination is their public lightning network node which they want to keep private, or because they are paying from a custodial service and don't want the custodian to know the destination of their payment.  In order not to have to trust an intermediary with their funds, these users can request that an lnproxy relay generate a "proxy invoice": an invoice with the same payment hash.

Then, once the user verifies the amount and payment hash, the user can use the proxy invoice wherever the original invoice would have been used and know that the only way for the payment to succeed is for the lnproxy relay to pay the original invoice.

## Requesting and verifying a proxy invoice

1. User make a POST request to a relay like:
   ```Bash
   curl --header "Content-Type: application/json" \
       --request POST \
       --data '{"invoice":"<bolt11 invoice>"}' \
       <relay URL>
   ```
2. User gets a JSON response from the relay of form:
   ```Typescript
   {
     "proxy_invoice": string // bech32-serialized lightning invoice
   }
   ```
   or
   ```JSON
    {"status":"ERROR", "reason":"error details..."}
    ```

3. User verifies that the payment hash in the proxy invoice matches the payment hash in the original invoice.

4. User verifies that the amount in the proxy invoice exceeds the amount in the original invoice by an acceptably small amount.

5. User uses the proxy invoice in place of the original invoice to send or receive a payment.

## Notes on implementing an lnproxy relay

### Accounting for routing fees

### Setting the `min_final_cltv_expiry`

### Features
