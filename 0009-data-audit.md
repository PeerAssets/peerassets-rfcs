# PeerAssets Data Audit Protocol

- Status: proposed
- Type: new feature
- Related components: (if any)
- Start Date: 2018-06-16
- Discussion: (fill me in with link to RFC discussion - shepherd will complete this) 
- Supersedes: (fill me in with a link to RFC this supersedes - if applicable)
- Superseded by: (fill me in with a link to RFC this is superseded by - if applicable)
- Author: Frano Perleta

## Summary

This document proposes a protocol for embedding document revision histories in a public blockchain.

## Conventions
- The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and
  "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).

## Motivation

The primary purpose of this protocol is to record cryptographic hashes of successive revisions of single-file documents
in a public blockchain, in a manner which enables thin clients to easily query and verify document histories. Such
histories inherit useful properties from the underlying blockchain, namely immutability and massive replication, and can
therefore serve as proofs of existence.

In addition to cryptographic hashes, users may also record a collection of URIs for each document, facilitating
retrieval of document contents from off-chain sources.

The goal is to impose minimal requirements on the blockchain, in terms of both features and storage space.

## Detailed design

The central notion is one of *documents*. A document is uniquely and permanently identified with a P2TH address. How the
address is derived is irrelevant, provided that it is indeed unique, ie. not already present in the underlying
blockchain at document creation time.

Document *state* has two components:

* the owner -- identified by their address;

* the content -- a blob (ie. a contiguous finite sequence of bytes, without further restrictions), identified by one or
  more cryptographically secure hash digests, optionally carrying additional metadata;

Document *history* is document state as a piecewise constant function of time, represented as a sequence of discrete
state transitions embedded in the underlying blockchain in the form of specially crafted transactions. Notably, all
relevant transactions include the document's P2TH address as the first output, which enables clients to efficiently
retrieve them in order to reconstruct the document's history, including the present state.

The second output of such transactions is an `OP_RETURN` carrying as payload a compact encoding of a state transition.
State transitions often affect only part of the state, thus only the difference is encoded in the payload, considering
the severe constraints on its size. Furthermore, the notion of ownership entails the need for validation, as only the
current owner may initiate the next transition. Consequentially, the transactions must be processed in chronological
order.

* The first transaction which specifies the content determines the initial document state. Any earlier transactions must
  be ignored. The initial transaction is always valid, and establishes the spender as the owner of the document.

* Each subsequent transaction is validated with respect to the state produced by the previous valid transaction, by
  checking whether the spender is the current owner. If a transaction is valid, the next state is computed by applying
  the change encoded in the `OP_RETURN` payload to the current state. This may involve a transfer of ownership onto
  another address.

* Invalid transactions, including those with omitted or malformed payloads, must be ignored.

Obviously, blobs of arbitrary size cannot be contained within transactions. The question of blob storage and retrieval
is entirely outside of the scope of this protocol. Instead of blobs themselves, the content representation includes only
their digests, which are of fixed size. Specifically, the following cryptographically secure hash functions are
supported:

* SHA2-256,
* SHA2-512,
* SHA3-256,
* SHA3-512.

This enables clients which manage to obtain the blobs by some other means to verify their integrity by computing their
digests and comparing them to those recorded in document histories. The possibility of using more than one hash function
at the same time, so that there is effectively a concatenation of multiple digests, should not be misunderstood as
providing additional security. Unfortunately, such a construction is not necessarily any stronger than the strongest
used hash function. Still, clients seeking to verify a blob are advised to check against all recorded digests, to reduce
the (already extremely small) probability of accidental collisions.

In addition to the mandatory digest(s), the content representation may also include one or more URIs, intended to
facilitate blob retrieval. The URIs are treated as opaque bytestrings, although clients are advised to conform to the
URI generic syntax. This is a non-essential feature provided in anticipation of common use of HTTP/S URLs, magnet links,
etc., as means of off-chain blob distribution.

### Transaction decoding and interpretation

A transaction's timestamp is the timestamp of the encompassing block, ie. it is determined by the decentralized process
of extending the blockchain, rather than chosen freely by the creator of the transaction. To the degree that block
timestamps are reliable, they serve as upper bounds on the creation times of blobs whose cryptographic hashes are stored
in the transaction.

The spender's address is extracted from the signature script of the input at index 0, which is a standard P2PKH input.
Naturally, the spender must match the current owner for the transaction to represent a valid document state transition.
Otherwise, the transaction must be ignored.

The document identifier (ie. its P2TH address) is extracted from the output at index 0, where the script has the
standard P2PKH form:

    OP_DUP OP_HASH160 <documentID> OP_EQUALVERIFY OP_CHECKSIG

The output at index 1 contains the Protocol Buffers message `TxPayload` in the standard provably unspendable script:

    OP_RETURN <payload>

The payload must be successfully decoded as an instance of `TxPayload`, otherwise the entire transaction is to be
ignored. Additional outputs are allowed but irrelevant.

The `version` field should be omitted to conserve space, as the default value of zero corresponds to the protocol
defined is this document. Other values are acceptable, and any unknown fields (presumably introduced by some future
version) may be simply ignored.

The `flags` field consists of two bits, `CONTENT_UPDATE` and `OWNERSHIP_TRANSFER`. If the `version` field is zero, no
other bit of `flags` may be set, or the payload is malformed and the transaction therefore ignored.

If the `OWNERSHIP_TRANSFER` bit is set, the input at index 1 must have a P2PKH script containing the public key of the
recipient, which is hashed to produce an address. In any case, additional inputs are allowed but irrelevant.

If the `CONTENT_UPDATE` bit is set, the payload must contain at least one hash digest field, described below. The
obvious implication is that the content of the document is (almost always) a different blob, compared to previous
versions. Therefore all previously mentioned digests are invalidated. However, digests may be added in subsequent
transactions without the `CONTENT_UPDATE` flag, usually simply because all of the digests cannot fit into a single
transaction. If a transaction contains a digest corresponding to a hash function whose value has already been defined
since the last content update, and the values do not match, the transaction is invalid and must be ignored.

The initial transaction must have the `CONTENT_UPDATE` bit set. In other words, transactions tagged with the document's
ID which are not content updates and are not preceded by content updates must be ignored, since they are not considered
to be part of the document's timeline.

The digest fields `sha2` and `sha3` are bytestrings required to be exactly 32 or 64 bytes long, as they contain the
compact binary encoding of the 256-bit and 512-bit variants of those hash functions. Any other length makes the entire
transaction invalid.

The `uri` field may contain general-form URIs. The `http` and `https` fields are similar, but with `http://` and
`https://` prefixes implied, to save space.

The `ipfs` field may contain binary-encoded multihash IDs, equivalent to URIs of the form `ipfs://${MULTIHASH}`, where
the multihash is Base58-encoded.

The `magnet_sha1` and `magnet_btih` fields may contain binary-encoded hash digests, equivalent to URIs of the following
forms:

    magnet:?xt=urn:sha1:${SHA1_HEX}
    magnet:?xt=urn:btih:${BTIH_HEX}

Unlike the blob digests, which are valid until the next content update, the URIs have no invalidation mechanism.

## Drawbacks

## Alternatives

## Unresolved questions

