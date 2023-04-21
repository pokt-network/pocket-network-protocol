# ADR-0003: Adopt Libp2p PeerIDs

* Status: in review
* Deciders: Pocket Network team
* Date: 2023-04-14

Technical Story: [Consolidate and Refactor Node Identity - #348](https://github.com/pokt-network/pocket/issue/348)

## Context and Problem Statement

In the context of simplifying and consolidating node identity, facing the need for a unique and verifiable identifier for nodes in the peer-to-peer network, we decided to adopt Libp2p identity/IDs and neglect alternative identity schemes, to achieve a reliable and secure identification system, accepting the additional dependency and integration work, because Libp2p provides a proven solution with robust security features.

## Decision Drivers

* Unique and verifiable node identifiers
* Secure and reliable identification system
* Proven solution in the peer-to-peer ecosystem

## Considered Options

* Adopting Libp2p identity/IDs
* Custom identity scheme

## Decision Outcome

Chosen option: "Adopting Libp2p identity/IDs", because it provides a unique, verifiable, and secure identification system that is proven in the peer-to-peer ecosystem.

### Positive Consequences

* Unique and verifiable node identifiers
* Secure and reliable identification system
* Simplified integration with other Libp2p components

### Negative Consequences

* Additional dependency on the Libp2p library
* Integration work required to adopt Libp2p identity/IDs

## Pros and Cons of the Options

### Adopting Libp2p identity/IDs

* Good, because it provides unique and verifiable node identifiers
* Good, because it offers a secure and reliable identification system
* Good, because it is a proven solution in the peer-to-peer ecosystem
* Bad, because it introduces an additional dependency
* Bad, because it requires integration work to adopt Libp2p identity/IDs

### Custom identity scheme

* Good, because it doesn't require an additional dependency
* Bad, because it may lead to a less secure and reliable solution
* Bad, because it requires development and maintenance efforts
* Bad, because it may not be compatible with other Libp2p components

### Links

* [`go-libp2p` - `peer.ID`](https://pkg.go.dev/github.com/libp2p/go-libp2p@v0.27.1/core/peer#ID)
* [pokt-network/pocket#434](https://github.com/pokt-network/pocket/issue/434)