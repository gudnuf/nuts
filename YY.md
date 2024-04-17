# Nuts on Nostr

Nostr is a prime use case for Cashu, so there should be native nostr support for mints and wallets.

## Features

- A mint's pubkey should be a from a nostr key pair.
    -  enable mint identification over nostr
- Wallet to mint communication: a wallet should be able to interact with a mint using nostr relays.
- Wallet to wallet communication: wallet A should be able to send tokens to wallet B using nostr
- Wallet should not have to use http!
- Share spent secrets publicly
- 

## Mint Public Key

As far as I can tell, right now the pubkey of a mint is left as a placeholder for something.

Let's use it as a mint's nostr key.

With this, a few things are made possible:

- A mint can advertise itself with a [NIP 87](https://github.com/benthecarman/nips/blob/ecash-mint-discover/87.md) `kind: 38172` info event
- Wallets can query nostr for `38172` or `38000` events to find a mint
- We can build NMC (Nostr Mint Connect)

## NMC

Nostr Mint Connect describe a flow very similar to NWC ([NIP 47](https://github.com/nostr-protocol/nips/blob/master/47.md)).

As a brief summary of NIP 47:

- Request/response events get published to nostr
- Event content is encrypted with a shared key computed through a Diffie Hellman key exchange.
- For example, an app can publish a `pay_invoice` request that their wallet will pick up, process and send back a response with a result or error.

Nostr mint connect is a way for the same request/response model to facilitate communication between cashu mints and cashu wallets.

Event content is encrypted:
- By the mints private key and public key found in request events
- By the wallets private key and the mint's public key found at their `/v1/info` endpoint.

An example would be minting tokens:

Wallet sends a `mint_token` request that commits to the blinded messages they will use.

The encrypted request content might look like:

```json
{
  "unit": <unit_to_mint>,
  "amount": <amount_to_mint>,
  "commitment": <sha256_blinded_msg>
}
```

When a mint receives this request, it will respond with a `request` and the `commitment`.

The wallet then has to pay the invoice. Once the invoice is paid, the wallet can send another request to get the tokens, or the mint should be able to just publish the tokens once it sees that the invoice is paid, but this would require the mint to have the blinded messages ahead of time.

The content of these requests/responses could also be exactly the same as the request responses of the REST calls to the mint.

### NMC Advantages

- no need to have dns records
- asynchronysity: requests/responses can sit on relays if wallet or mint are offline.
- comms are encrypted
- a sea of ideas becomes possible when you're nostr native

### NMC Disadvantages

- Could dox privacy of the wallet if they use the same key everytime they communicate with the mint.
- Mint activity is partially visible by the requests/responses going through the relays
- 
