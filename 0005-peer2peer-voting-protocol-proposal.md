# PeerAssets voting protocol specification

- Status: proposed
- Type: new feature
- Start Date: 13-03-2017
- Discussion: (fill me in with link to RFC discussion - shepherd will complete this)
- Author: peerchemist

## Summary

PeerAssets voting protocol.
Aim is to deliver a standardized way to organize and operate peer-to-peer voting and opinion polls in fully decentralized and trustless fashion.
Voting is done in two phases, vote initialization (vote_init) and vote casting (vote_cast). 
To enable voting on the deck, the deck issuer has to make first blank vote_init to deterministic P2TH address (deck_vote_tag) which will serve as vote tag of the deck in the future.
This transaction is called deck_vote_init.
Clients of the deck detect this and are now able to start votes or polls by tagging vote_inits with this address.
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
For example, for Peercoin-testnet based deck "hopium" asset_id is "d460651e1d9147770ec9d4c254bcc68ff5d203a86b97c09d00955fb3f714cab3" so vote_init address is derived from "c93c3a899f1b488fb972cb70f24dc7d22efddebced941ae3ae82336c3751a7a1" and equals "mtE3jLc8LtXGhrEQJksbmWd5KH4S5yQVrj".

Python code using pypeerassets:

```
import pypeerassets as pa
from hashlib import sha256

asset_id = "d460651e1d9147770ec9d4c254bcc68ff5d203a86b97c09d00955fb3f714cab3".encode()
deck_vote_tag_privkey = sha256(asset_id + "vote_init".encode()).hexdigest()

deck_vote_tag_address = pa.Kutil(network="tppc", privkey=vote_init_privkey).address

assert deck_vote_tag_address == "mtE3jLc8LtXGhrEQJksbmWd5KH4S5yQVrj"
```

## Vote init

Vote init is special transaction sent to deck vote tag which announces the vote to the deck and informs interested parties on nature of the vote with attached information like:

* start block of the vote
* end block of the vote
* vote counting method (weighting with card balance, weighting with card-days, etc.)
if vote count method has cutoff card-days or card balance, it needs to be included here.
* choices of the vote/poll

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

### Deck vote init

To enable voting on the deck, deck issuer has to make first blank vote_init and send it to deck_vote_tag.
This transaction is called deck_vote_init.
This transaction has to originate from the deck spawn address.


## Vote casting

Vote is casted by sending sending transaction from address which has positive card balance to address derived from sha256(vote_init + #OPTION).
Vote_cast receiver address is calculated deterministically from vote_init and vote option index.

For example, Bob is voting for on choices offered by vote_init in example above. Bob wants to vote for "trump" option, which is third option of the vote.
Bob calculates vote_cast address in Python using pypeerassets:

```
import pypeerassets as pa
from hashlib import sha256

vote_init_txid = "b89c3a198f1b488fb972cb70f24dc7d21efbdebked941ae3ae82536c3951c7a1".encode()
option_index = 2#vote_init["choices"].index("trump")

vote_cast_privkey = sha256(vote_init_txid + bytes(option_index)).hexdigest()

vote_cast_address = pa.Kutil(network="tppc", privkey=vote_cast_privkey).address

assert vote_cast_address == "moNEVvdptv3qa2stvSaw48Wr1G9n88QUwR"

```

All clients can deterministically calculate this address and check transactions paying to it to verify if they are legit vote_casts.


## Vote counting

Simplest form of vote counting is vote per transaction, with filtering of double votes and invalid votes.
The only requisite of valid vote is positive card balance with this schema. However this schema is easily gamed, so two more complex schemas are proposed bellow.

### Weighting votes with PA card balance

This schema allows weighting vote with PeerAssets card balance. This allows equating casted vote with stake in the deck.
Simplest form of card balance weighted voting is using card balance when summing up votes.

Example:

> Voting options are "A" and "B".

    Alice_card_balance = 102.0
    Bob_card_balance = 80.0
    David_card_balance = 27.0

    Alice votes for option "A", while Bob and David vote "B".
    Option A weight is 102
    Option B weight is 107

    Option B wins.


#### Weighting vote with PA card balance with "lookback period"

This scheme prevents actor to purchase cards during or just before the vote and them use them to cast weighted vote and then simply sell them.
Lookback period schema can be applied to example above, where balance of cards is checked with lookback period of 10 blocks.

> Voting options are "A" and "B", cut-off time is 10 blocks

    Alice_card_balance = 40.0
    Bob_card_balance = 80.0
    David_card_balance = 27.0

    Alice votes for option "A", while Bob and David vote "B".
    Option A weight is 40
    Option B weight is 107

    Option B wins.


In this example Alice tried to influence the vote by buying more cards after the vote begun, however those cards are not counted due to lookback period.

### Weightnig vote with PA card days

Voting can also be weighted with so called "card days", as explained in PeerAssets whitepaper:
```
Card age is a term derived from "coin age" used in Peercoin and it represents a product of quantity of the tokens and days they have been idle.
In PeerAssets context card age is a product of the card balance and the time which card has spent being immovable with that address.
```

expressing time in blocks or in days?

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


This schema prevents "last minute" manipulations by giving more weight to older card holders.

## Drawbacks

## Alternatives

## Unresolved questions

<!-- References -->

[1]:
[2]:
