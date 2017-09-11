# Deck native fee

- Status: proposed
- Type: new feature
- Related components: 0001-peerassets-transaction-specification.proto
- Start Date: 27-08-2017
- Discussion: (fill me in with link to RFC discussion)
- author: Peerchemist

## Summary

Deck native fee allows for card transfer fee expressed in the deck's native token. Per-deck fee is set upon deck spawn and becomes the requirement of each valid card transfer on the deck.
Procedure is the same as if every card transfer on the deck also included a card burn transaction for some amount of cards (tokens) which means that each fee is deducted from the total supply of the card on the deck.


## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).

## Motivation

There is utility in having a deck native fee, notably as card transfer fees act as deflationary force on the card supply allowing for creation of dynamic economic system revolving around token in question.

## Detailed design

### Defining the fee parameter on deck spawn

Native deck fee is equal to zero by default, and does not have to be specified in the deck spawning protobuf if issuer wants to keep default value of zero.
Deck fee has to be encoded as: `fee * 10**number_of_decimals` to allow for more elaborate fee expressions.

Example of deck metainfo (to be encoded in protobuf):

```
{'issue_mode': ['MULTI', 'MONO'],
 'name': 'deck_testing',
 'number_of_decimals': 2,
 'fee': 1
 'version': 1}
```

### Card transfer

First card transfer output must be addressed to deck issuer and set to amount equal or larger than the deck's native fee.

Card transfer transaction specification:

* `vin[0]`: The sending party of the transfer transaction.
* `vout[0]`: Deck tag (P2TH).
* `vout[1]`: (`OP_RETURN`) Asset transfer data containing the amounts of transferred assets, first amount must be equal or greater than the fee.
* `vout[2]`: The receiving parties of the transfer transaction, first receiving party must be deck issue address.
* other vouts, usually change address.


#### Implementation

For card transfer to be valid on a deck where fee is not equal to zero parser should assert that the first output is addressed to deck issue address and that the first amount is greater or equal to fee parameter of the deck.

## Drawbacks

Why should we *not* do this?

## Alternatives


## Unresolved questions

