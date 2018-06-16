# PeerAssets PERISH issue mode specification

- Status: proposed
- Type: new feature
- Related components: 0008-peerassets-perish-specification.proto
- Start Date: 26-08-2017
- Discussion: (fill me in with link to RFC discussion)
- author: Peerchemist

## Summary

PeerAssets "PERISH" issue mode allows for creation of decks where cards expire after *n* blocks, as if they are perishable goods. This issue mode will allow issuance of cards which have a strictly defined expiry rate, and last only for *n* blocks.


## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).


## Motivation

There is number of scenarios where we could use a token which expires after some time, most obvious examples being time based subscriptions of pay-walled services, expirable API keys and logins, time limited in-game tokenized power-ups, coupons, tickets for an event and many others.

#### Hypothetical use cases

A local grocery store operates a PeerAssets deck on which they issue coupons which carry discounts. Once a month grocery store issues limited number of discount coupons to it's best paying customers but as limited time offer, only valid for two days. Grocery store can use PERISH issue mode with flag 288. 288 is expected number of blocks in two days.
Grocery store started this deck with following issue modes: `MULTI`, `PERISH`.

* MULTI issue mode makes sure that grocery store can issue the coupons whenever it pleases.
* PERISH issue mode with flag 288 makes sure that all the cards expire after 288 blocks.



PeerAssets token can be used to mark subscription to online magazine, where token is used to unlock access to some premium content.
For any service provider who does not want to deal with databases, backups and other elements of classic web stack this could be a nice solution as it also brings payment processing in a single package. This example somewhat duplicates functionality of the `SUBSCRIPTION` mode explained in the PA-RFC004.
Service provider would spawn a deck with following deck_issue modes: `MULTI`, `UNFLUSHABLE` and `PERISH`.

* MULTI issue mode would allow multiple card issuances on this deck, per individual subscription.
* UNFLUSHABLE would invalidate any customer to customer token tranfers, making the only valid tokens those who are issued by the deck creator (service provider in this case).
* PERISH would invalidate all the cards (tokens) older than n blocks ( 1 block = 10 minutes on average).


## Detailed design

PERISH issue mode is a simple concept, basically card is not valid if more than *n* blocks has passed since it was issued. When parsing, please note that block when card was issued is different from block of last transfer. To find card issue block height you must climb the PeerAssets transaction tree to it's issue block.

For example, if card was issued directly to Alice:

> Alice sends 10 cards to Bob
> Bob checks the deck_issue mode and sees PERISH, checks the rules, number_of_blocks = 100 (cards expire after 100 blocks)
> Bob verifies: if block_height - card_issue block < 100 transfer is valid

### Deck spawn

This is the first deck issue mode which requires extra metadata within the new, optional, rules field in the deckspawn protobuf. If no flags are provided in the rules field, cards expire after 0 blocks which makes them all invalid by default.

```
rules={
	   "number_of_blocks" = n
	  }
```

Other possible rules beside number_of_blocks:

* hops [integer] :

Maximum number of "hops" (transfers) the card can do, counted from the issuer. For example, it can be transfered max 3 times and then it becomes invalid.

Example of deck_spawn for PERISH deck:

```
deck_spawn = {
		"version" : 1,
		"name": "coupon",
		"number_of_decimals": 0,
		"issue_mode": MULTI & PERISH,
		"rules": {"n": 2}
		}
```

## Advantages

* Allowing issuance of perishable time-limited tokens.

## Drawbacks

Depends on introducing new deck-defining information in the deck spawn protobuf, the "rules" field.

## Alternatives

## Unresolved questions

<!-- References -->
