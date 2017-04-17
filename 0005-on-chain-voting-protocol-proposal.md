# PeerAssets on-chain voting protocol specification

- Status: proposed
- Type: new feature
- Start Date: 13-03-2017
- Discussion: (fill me in with link to RFC discussion - shepherd will complete this)
- Related components: 0005-on-chain-voting-transaction-specification.proto
- Author: peerchemist, saeveritt

## Summary

PeerAssets on-chain voting protocol.
Aim is to deliver a standardized way to organize and operate peer-to-peer voting and opinion polls in fully decentralized and trustless fashion. Both vote and poll initialization and counting must be done in completely decentralized and trustless fashion without dependency on 3rd party services.
Voting is done in two phases, vote initialization (vote_init) and vote casting (vote_cast). 
Votes or polls can be started by anyone on the deck by creating vote_init transaction and paying to deck_vote_tag address. This transaction is called vote_init.
Deck clients detect new votes or polls in progress by monitoring deck_vote_init address and participate in them by paying to their respective P2TH derived from vote_init. This transaction is called "vote_cast".

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).
- "DECK", "CARD" are used as described in PeerAssets whitepaper.

## Motivation

PeerAssets whitepaper mentions possibility of using PeerAssets as framework for DAO/DAC operation, so protocol should provide standardized specification on how to run decentralized voting and opinion polls. Primary goal of offered protocol is to allow fully decentralized and trustless peer-to-peer operation which does not depend on 3rd party hosts like forums.

_______

# Detailed design

## Deck vote tag

Deck vote tag is deterministic P2TH address derived from sha256 hash of (asset_id + "vote_init") string and used as deck-level tag for all things voting.
For example, for Peercoin-testnet based deck "hopium" asset_id is "d460651e1d9147770ec9d4c254bcc68ff5d203a86b97c09d00955fb3f714cab3" so vote_init address is derived from "c93c3a899f1b488fb972cb70f24dc7d22efddebced941ae3ae82336c3751a7a1" + "vote_init" and equals "mtE3jLc8LtXGhrEQJksbmWd5KH4S5yQVrj".

Python code for deriving deck_vote_tag using pypeerassets:

```
import pypeerassets as pa
from hashlib import sha256
from binascii import unhexlify

asset_id = unhexlify("d460651e1d9147770ec9d4c254bcc68ff5d203a86b97c09d00955fb3f714cab3")
deck_vote_tag_privkey = sha256(asset_id + b"vote_init").hexdigest()

deck_vote_tag_address = pa.Kutil(network="tppc", privkey=deck_vote_tag_privkey).address

assert deck_vote_tag_address == "n116UKstnbrNJBdYHgx4UnAs9xYHVBczFs"
```

## Vote init

Vote init is special transaction sent to deck vote tag which announces the vote to the deck and informs interested parties on nature of the vote with attached information like:

* start block of the vote
* end block of the vote
* vote counting method (weighting with card balance, weighting with card-days, etc.)
* choices of the vote/poll

Vote init transaction pays to deck_vote_tag as first output.

Example of information describing the vote_init:

```
vote_init = {
    "start": blocknum,
    "end": blocknum,
    "count_method": weight_card_days,
    "choices": [
                "putin"
                "merkel",
                "trump"]
}
```

Information describing the vote_init is attached as protobuf in the op_return output.

Vote_init transaction can be sent by any party interested in the deck and and is understood by all clients subscribed to deck.


## Vote casting

Vote is casted by sending sending transaction from address which has positive card balance to address derived from sha256(vote_init + #OPTION).
Vote_cast receiver address is calculated deterministically from vote_init and vote option index.

For example, Bob is voting for on choices offered by vote_init in example above. Bob wants to vote for "trump" option, which is third option of the vote.
Bob calculates vote_cast address in Python using pypeerassets:

```
import pypeerassets as pa
from hashlib import sha256
from binascii import unhexlify

vote_init_txid = unhexlify("b461051e1d9147770ec9d4c254bdd68ff5d203a86b97c09d00955fb3f088cab3")
option_index = vote_init["choices"].index("trump") # 2

vote_cast_privkey = sha256(vote_init_txid + bytes(option_index)).hexdigest()

vote_cast_address = pa.Kutil(network="tppc", privkey=vote_cast_privkey).address

assert vote_cast_address == "msCpEmB3m2sARUikdZSCgyoCokmXykD1de"

```

All clients can deterministically calculate this address and check transactions paying to it to verify if they are legit vote_casts.


## Vote counting

### Simple vote counting

Simplest form of vote counting is counting one vote per transaction, with filtering of double votes and invalid votes. Filtering is done by using "first come, first served" rule, where only the first vote is registered and subsequent ones are dismissed as invalid.
The only requisite of valid vote with this schema is positive card balance. However this schema is easily gamed by buying cards just before casting a vote and selling them after the vote has been casted, so two more complex voting schemes are proposed bellow.

### Weighting votes with PA card balance

This schema allows weighting the vote with PeerAssets card balance. This allows equating casted vote with stake in the PeerAssets deck.
Simplest form of card balance weighted voting is using card balance when summing up votes. Same rule of "first come, first served" is applied to prevent double voting.

Example:

> Voting options are "A" and "B".

    Alice_card_balance = 102.0
    Bob_card_balance = 80.0
    David_card_balance = 27.0

    Alice votes for option "A", while Bob and David vote "B".
    Option A weight is 102
    Option B weight is 107

    Option B wins.


### Weighting vote with PA card days

Voting can also be weighted with so called "card days", as explained in PeerAssets whitepaper <sup>[1]</sup>:
```
Card age is a term derived from "coin age" used in Peercoin and it represents a product of quantity of the tokens and days they have been idle.
In PeerAssets context card age is a product of the card balance and the time which card has spent being immovable with that address.
```
Therefor votes are counted as card days, card balance is multiplied with age of UTXO which was used to deliver the cards.
This schema prevents "last minute" vote manipulations by giving more weight to older card holders.

> Voting options are "A" and "B".

    Alice_card_balance = 102.0
    Alice_card_days = 10
    Bob_card_balance = 80.0
    Bob_card_days = 100
    David_card_balance = 27.0
    David_card_days = 10


    Alice votes for option "A", while Bob and David vote "B".
    Option A weight is 1020.0
    Option B weight is 8270.0

    Option B wins.


## Drawbacks

## Alternatives

## Unresolved questions

<!-- References -->

[1] (https://github.com/PeerAssets/WhitePaper/blob/master/README.md): PeerAssets Whitepaper, 2016
