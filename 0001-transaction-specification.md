# Transaction specification

- Status: proposed
- Type: new feature
- Related components: /
- Start Date: 06-10-2016

## Summary

Two basic types and three special types of PeerAsset transactions are specified:
- Deck spawn transaction
  - Deck transfer transaction
- Card transfer transaction
  - Card issue transaction
  - Card burn transaction

With the deck transfer, card issue and card burn transactions being special types of their respective base types transaction.

The deck spawn transaction *registers* a new asset to the network. The owner of `vin[0]` is the only enitity that is allowed to transfer assets with a negative balance, resulting in a card issue transaction.
The deck spawn transaction decides how much the asset can be divided (*number_of_decimals*) and how the asset can be issued (*issue_mode*).

The deck transfer transaction is a special case of the deck spawn transaction. Instead of registering a new asset, the deck transfer transaction transfers ownership from `vin[1]` to `vin[0]`, meaning that both parties are required to sign the transfer transaction for it to be accepted in the blockchain. Next to a transfer of ownership, the transfer transaction makes it possible to upgrade the asset to a newer protocol version and it allows some meta-data to be added or updated. However, the asset's *name* and *issue_mode* are never allowed to change.

The card transfer transaction *transfers* the ownership of assets from one the originating address to one or multiple new addresses.
This works on a first come first serve basis following the serialization order on the blockchain.
Once the balance of an account becomes lower than a transfer transaction it issued (sum of it's outputs), that entire transfer transaction is considered invalid.
Topping up the balance of an account to make that transfer valid, requires that transaction to be resent so it gets serialized in the chain after the incoming transaction on that account.

The card issue transaction is a special case of the card transfer transaction originating from the owner of the deck spawn transaction (or latest deck transfer transaction if one exists).
The asset owner is the only account that is allowed to hold a negative balance.
This balance is used as a checksum to validate a correct computation of all account balances.
If the deck spawn transaction specifies the `ONCE` issue mode, only the first serialized card issue transaction is considered valid.
Just as the card transfer transaction, the issue transaction is allowed to contain multiple outputs.
This means that when the `ONCE` issue mode is specified, cards can be issued to multiple parties directly.

The card burn transaction is a special case of the card transfer transaction sent to the owner of the deck spawn transaction.
This results in the owner's balance to decrease in absolute value (become less negative), meaning that the total amount of issued assets decreases.
This transaction can be used as Proof of Burn.

## Motivation

This RNC is meant to iterate and agree on the PeerAsset transaction protocol.

## Detailed design

A PeerAssets transaction encodes it's information in the transaction's inputs (`vin[]`), outputs (`vout[]`) and a special meta-data output (`OP_RETURN`).

### P2TH tags

To easily query for PeerAsset transactions, the PeerAssets protocol requires a [P2TH][2] output in it's transactions.
The peercoin v0.5 protocol requires every output to at least 0.01PPC of value or to hold no value at all.
To prevent tag spam to slow down PeerAsset clients, a tag output should at least hold 0.01PPC of value before the client starts parsing the entire transaction, this is referred to as the *tag fee*.
Therefore, every PeerAsset transaction spends 0.02PPC of wich 0.01PPC is burned as a transaction fee and 0.01PPC is sent to the tag hash to be claimed as a reward for cleaning up the UTXO table, for example by reusing the tag fee in a future PeerAsset transaction.

### Deck spawn transaction layout

For the deck spawn transaction, the following properties are specified:
* `txnid`: The unique identifier for this asset.
* `vin[0]`: The owner of the Asset. Ownership of the asset is proven by proving ownership over the Public Key Hash (PKH) or Script Hash (SH) originating `vin[0]`.
* `vout[0]`: Deck spawn tag using a [P2TH][2] output. This tag registers the assets as a PeerAsset so that it can be discovered by PeerAsset clients. In case of the card transfer transaction, the card transfer tag is used.
* `vout[1]`: (`OP_RETURN`) Asset meta-data. A [protobuf3 encoded message][1] containing meta-data about the asset.
* all other in and outputs are free to be used in any way. `vout[2]` will typically be used as a change output.

### Deck transfer transaction layout

Although the deck transfer transaction is a special case of the deck spawn transaction, it differs enough to be separately specified.

* `txnid`: No specific meaning, the deck spawn `txnid` remains the unique identifier.
* `vin[0]`: The new owner of the Asset. For safety reasons, a transaction input is used for the new owner. Using an input instead of an output requires the receiver of the deck transfer transaction to sign the transaction making sure that the receiving party is able to sign future transactions. Note that `vin[]` and `vout[]` refer to peercoin transaction in and outputs, which aren't necessarily PeerAsset in and outputs.
* `vin[1]`: The previous owner of the Asset in case of a deck transfer transaction.
* `vout[0]`: Asset tag using a [P2TH][2] output based on the asset's unique identifier instead of a deck spawn tag.
* `vout[1]`: (`OP_RETURN`) Asset meta-data. A [protobuf3 encoded message][1] with the same layout as the deck spawn message.
* all other in and outputs are free to be used in any way. `vout[2]` will typically be used as a change output.

The deck transfer protobuf message has the same layout as the deck spawn message as it overwrites the properties of the deck spawn transaction.
This allows the deck transfer transaction to upgrade an asset to a newer protocol version.
However, changes to `name`, `number_of_decimals` or `issue_mode` are not allowed and will mark the transaction as invalid.

### Deck spawn tags

The deck spawn tag private keys are publicly known so it can be imported in every ppcoin node to easily query for deck spawn transactions without the need for a block explorer and to allow any node to claim the tag fees resulting in an UTXO cleanup.

PPC mainnet:
- `PAprod`: PAprodbYvZqf4vjhef49aThB9rSZRxXsM6 - U624wXL6iT7XZ9qeHsrtPGEiU78V1YxDfwq75Mymd61Ch56w47KE
- `PAtest`: PAtesth4QreCwMzXJjYHBcCVKbC4wjbYKP - UAbxMGQQKmfZCwKXAhUQg3MZNXcnSqG5Z6wAJMLNVUAcyJ5yYxLP

PPC testnet:
- `PAprod`: miHhMLaMWubq4Wx6SdTEqZcUHEGp8RKMZt - cTJVuFKuupqVjaQCFLtsJfG8NyEyHZ3vjCdistzitsD2ZapvwYZH
- `PAtest`: mvfR2sSxAfmDaGgPcmdsTwPqzS6R9nM5Bo - cQToBYwzrB3yHr8h7PchBobM3zbrdGKj2LtXqg7NQLuxsHeKJtRL

### Card transfer transaction layout

For the card transfer transaction, the following properties are specified:
* `vin[0]`: The sending party of the transfer transaction.
* `vout[0]`: Asset tag using a [P2TH][2] output based on the asset's unique identifier, see [test vectors][4]. This tag makes it easy for nodes to follow transactions of a specific asset.
* `vout[1]`: (`OP_RETURN`) Asset transfer data. [protobuf3 encoded message][1] containing the amount of transferred assets and optionally some meta-data (ref. peerassets.proto).
* `vout[n+2]`: The receiving parties of the transfer transaction. vout[n+2] receives the n-th (starting with 0) amount specified in the asset transfer data message.
* all other in and outputs are free to be used in any way. `vout[2+receiver_cnt]` will typically be used as a change output.

### Asset tag generation

The asset tag refers to the asset's unique id, the deck spawn transaction id, as this allows nodes interested in this asset to easily retrieve it's card transfer and deck transfer transactions.
To generate the card transfer tag, the deck spawn transaction id is used as the raw private key.
The code below illustrates how this is done using the [bitcore JavaScript library][3].

```
var deckSpawnTxid = '5faf805821abc7307a9a38d1432521be325bd40cb492742c3164dd34fb78c283';
var privateKey = new PrivateKey(deckSpawnTxid);
console.log("private key WIF: " + privateKey.toWIF());
console.log("tag address: " + privateKey.toPublicKey().toAddress());
```

Any node interested in following a specific asset's transactions can simply import the tag's private key in it's wallet.

### Issue modes

There are five different basic issue modes, `ONCE`, `MULTI`, `MONO`, `UNFLUSHABLE` and `CUSTOM`, which can be used to specify how an asset can be issued.
If it makes sense, these modes can be combined to allow multiple ways to issue an asset.
* `ONCE`: Only one issuance transaction from asset owner allowed.
* `MULTI`: Multiple issuance transactions from asset owner allowed.
* `MONO`: All card transaction amounts are equal to 1.
* `UNFLUSHABLE`: No card transfer transactions allowed except for the card-issue transaction.
* `CUSTOM`: A custom client is implemented with non-standard issuance rules. This allows standard clients to enumerate these assets.
But to compute the asset balances, the custom client is required.
Therefore these assets aren't tradable on standard exchanges.

<!-- References -->
[1]: 0001-peerassets-transaction-specification.proto
[2]: http://peerassets.github.io/P2TH/
[3]: https://github.com/bitpay/bitcore-lib
[4]: 0002-transaction-test-vectors.md
