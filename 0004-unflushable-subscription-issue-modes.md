# UNFLUSHABLE & SUBSCRIPTION Issue Modes

- Status: proposed
- Type: new feature
- Related components: Transaction specification
- Start Date: 01-03-2017
- Discussion: (fill me in with link to RFC discussion)
- Author: hrobeers

## Summary

Two new issue modes are proposed to allow registering subscriptions using PeerAssets.
These modes allow validation of membership by third parties without having to provide any personal information.
Decentralized service archtectures may use this feature to validate subscription to premium services without having access to a centralized user database.

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).

## Motivation

In order to use PeerAsset tokens for managing subscriptions to premium services, new issue modes are required.
The issue modes proposed in this document, `UNFLUSHABLE` & `SUBSCRIPTION`, allow clients to validate time bound subscriptions issued by an entity.

Using PeerAssets for subscription tracking allows service providers to check for a client's subscription without the need of accessing a centralized user database.
This allows a more decentralized service architecture to be implemented by the service provider.
Or allows service providers to check for a subscription to an external entity they do not control or have access to.

To illustrate the latter, imagine *organization A* that tracks time bound membership (subscriptions) using PeerAssets.
*Organization A* may hold a user database that links PeerAssets addresses to a real life identity.
*Organization B* may decide to offer extra services or discounts to members of *organization A*.
However, *organization A* wishes not to give *organization B* access to their user database in any way to protect the personal information of their members.
Since *organization A* uses PeerAssets to track membership, members of *organization A* can prove their membership without having to provide any personal details, not even their username.

## Detailed design

In order to create the `SUBSCRIPTION` issue mode, an additional basic issue mode `UNFLUSHABLE` needs to be specified.

### Unflushable mode
The `UNFLUSHABLE` issue mode invalidates any card transfer transaction except for the card issue transaction.
Meaning that only the issuing entity is able to change the balance of a specific address.
To correctly calculate the balance of a PeerAssets address a client should only consider the card transfer transactions originating from the deck owner.

##### bitflags
As specified in [0001-peerassets-transaction-specification.proto](0001-peerassets-transaction-specification.proto), the `UNFLUSHABLE` issue mode is activated by setting the `0x10` bitflag.

### Subscription mode
The `SUBSCRIPTION` issue mode marks an address holding tokens as subscribed for a limited timeframe.
This timeframe is defined by the balance of the account and the time at which the first cards of this token are received.

To check validity of a subscription one should take the timestamp of the first received cards and add the address' balance to it in hours.
Hours is chosen as the unit of time to limit the size of the varint representing the balance
If a higer time resulotion is needed, one can choose to increase the number of decimals.

##### bitflags
The `SUBSCRIPTION` mode is a combinatory mode, meaning that it automatically enables some basic issue modes that are required for operation.
This has the advandtage of simplifying possible client implementations as the basic issue mode logic is automatically enforced.

`SUBSCRIPTION` combines `MULTI`, `UNFLUSHABLE` and it's own `0x20` bitflags resulting in the combined `0x34` value (`0x34 = 0x04 | 0x10 | 0x20`) as specified in [0001-peerassets-transaction-specification.proto](0001-peerassets-transaction-specification.proto).

### Implementation

Implementation suggestions.

### Use Case

Usage example.

## Drawbacks

Why should we *not* do this?

## Alternatives

What other designs have been considered? What is the impact of not doing this?

## Unresolved questions

What parts of the design are still to be done?