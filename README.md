draft: lnproxy spec.
====================

`author: lnproxy` `author: niteshbalusu11` `discussion: https://github.com/niteshbalusu11/lnproxy-ts/issues/2`

---

A user may want to use a proxy destination for an invoice, either because the user needs help finding a path to the original destination, because the original destination is their public lightning network node which they want to keep private, or because they are paying from a custodial service and don't want the custodian to know the destination of their payment.  In order not to have to trust an intermediary with their funds, these users can request that an lnproxy relay generate a "proxy invoice": an invoice with the same payment hash.

Then, once the user verifies the amount and payment hash, the user can use the proxy invoice wherever the original invoice would have been used and know that the only way for the payment to succeed is for the lnproxy relay to pay the original invoice.

## Requesting and verifying a proxy invoice

1. User makes a POST request to a relay like:
   ```Bash
   curl --header "Content-Type: application/json" \
       --request POST \
       --data '{"invoice":"<bolt11 invoice>"}' \
       <relay URL>
   ```
   The invoice must specify an amount, otherwise the relay to take the entire payment.

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

3. User verifies that the payment hash (tag `p`) in the proxy invoice matches the payment hash in the original invoice.

4. User verifies that the description (tag `d`) or description hash (tag `h`) in the proxy invoice matches the description or description hash in the original invoice.

5. User verifies that the amount in the proxy invoice exceeds the amount in the original invoice by an acceptably small amount.

6. User uses the proxy invoice in place of the original invoice to send or receive a payment.

## Requesting a proxy invoice with a different description

A user may want to generate a proxy invoice with a different description (tag `d`) or description hash (tag `h`) than the original invoice.  In this case, the flow is the same as above except steps 1 and 4 which become:

  1. User makes a POST request to a relay like:
     ```Bash
     curl --header "Content-Type: application/json" \
         --request POST \
         --data '{"invoice":"<bolt11 invoice>","description":"<new description>"}' \
         <relay URL>
     ```
     or
     ```Bash
     curl --header "Content-Type: application/json" \
         --request POST \
         --data '{"invoice":"<bolt11 invoice>","description_hash":"<new description hash as 32 bytes of hex>"}' \
         <relay URL>
     ```
and:

  4. User verifies that the description (tag `d`) or description hash (tag `h`) in the proxy invoice matches provided description or description hash provided in the request.

## Compatibility with LNURL

For an lnproxy relay to be used in conjunction with [LUD-06](https://github.com/lnurl/luds/blob/luds/06.md), the amount in the proxy invoice needs to be set by the user.  To accommodate this, a relay may accept an additional parameter `routing_msat` that specifies the millisatoshi amount the relay should use when routing the payment.  Note that the user is responsible for ensuring that the amount in `routing_msat` is enough to pay for the costs of routing the payment; otherwise, the payment will fail.  The flow is the same as above except steps 1 and 5 which become:

  1. User makes a POST request to a relay like:
     ```Bash
     curl --header "Content-Type: application/json" \
         --request POST \
         --data '{"invoice":"<bolt11 invoice>","routing_msat":"<millisatoshi amount used when routing the payment, as a string>"}' \
         <relay URL>
     ```
and:

  5. User verifies that the amount in the proxy invoice is exactly equal to the sum of the amount in the original invoice and the amount in the `routing_msat` request.

## Notes on implementing an lnproxy relay

Proxy invoices are hodl invoices. When an lnproxy relay accepts an htlc for a proxy invoice, it immediately pays the original invoice and uses the revealed preimage to settle the proxy invoice. This can expose the relay to certain risks if the following topics are not accounted for.

### Setting the `min_final_cltv_expiry`

An lnproxy relay needs to ensure that payments to the original invoice expire before payments to the proxy invoice.  Otherwise, an attacker could simply wait for the payment to the proxy invoice to expire before settling the payment from the relay.  This means that the `min_final_cltv_expiry` in a proxy invoice needs be longer than the entire route needed to pay the original invoice.

### Atomic multi-path payments

Relays cannot create proxy invoices for AMP invoices since there is not payment_hash reveal mechanism.
