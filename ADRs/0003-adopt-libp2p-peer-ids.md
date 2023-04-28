# Adopting Libp2p PeerIDs <!-- omit in toc -->

- Status: in review
- Deciders: Pocket Network team
- Date: 2023-04-14

**Table of Contents**

- [Summary](#summary-)
- [Problem Statement](#problem-statement-)
- [Technical Story](#technical-story-)
- [Decision Drivers](#decision-drivers-)
- [Considered Options](#considered-options-)
- [Decision Outcome](#decision-outcome-)
    - [Positive Consequences](#positive-consequences-)
    - [Negative Consequences](#negative-consequences-)
- [Pros and Cons of the Options](#pros-and-cons-of-the-options-)
    - [Adopting Libp2p identity/IDs](#adopting-libp2p-identityids)
    - [Custom identity scheme](#custom-identity-scheme)
- [References](#references-)

## Summary <!-- required -->

Facing the need for a unique and verifiable identifier for nodes in the peer-to-peer network, we decided to adopt Libp2p identity/IDs and neglect alternative identity schemes, to achieve a reliable and secure identification system, accepting the additional dependency and integration work, because Libp2p provides a proven solution with robust security features.

## Problem Statement <!-- required -->

At the time of writing, there is no formal specification for identity with respect to pocket network nodes/participants.
Each node MUST be uniquely identifiable/addressable so that it may be correctly routed to.
Identities MUST be provable (e.g. via cryptographic signature).

### Context <!-- optional -->

- We've recently refactored the P2P module to use libp2p for the transport layer. As a result we're already interoperating with libp2p objects and types.
- We're simultaneously exploring consolidating and simplifying the concept of node identity throughout the codebase wherever reasonable/feasible.

## Technical Story <!-- optional -->

[Consolidate and Refactor Node Identity - #348](https://github.com/pokt-network/pocket/issue/348)

## Decision Drivers <!-- optional -->

- Unique and verifiable node identifiers
- Secure and reliable identification system
- Proven solution in the peer-to-peer ecosystem

## Considered Options <!-- required -->

1. Adopting Libp2p identity/IDs
2. Custom identity scheme

## Decision Outcome <!-- required -->

Chosen option: "Adopting Libp2p identity/IDs", because it provides a unique, verifiable, and secure identification system that is proven in the peer-to-peer ecosystem.

### Positive Consequences <!-- optional -->

- Unique and verifiable node identifiers
- Secure and reliable identification system
- Simplified integration with other Libp2p components

### Negative Consequences <!-- optional -->

- Additional dependency on the Libp2p library
- Integration work required to adopt Libp2p identity/IDs
- May not be compatible with other Libp2p components or the refactored P2P module

## Pros and Cons of the Options <!-- required -->

### Adopting Libp2p identity/IDs

Pros:

- Provides unique and verifiable node identifiers
- Offers a secure and reliable identification system
- Proven solution in the peer-to-peer ecosystem
- Aligns with the refactored P2P module, which already uses libp2p for the transport layer

Cons:

- Introduces an additional dependency
- Requires integration work to adopt Libp2p identity/IDs

### Custom identity scheme

Pros:

- Doesn't require an additional dependency

Cons:

- May lead to a less secure and reliable solution
- Requires development and maintenance efforts
- May not be compatible with other Libp2p components

## References <!-- optional -->

- [`go-libp2p` - `peer.ID`](https://pkg.go.dev/github.com/libp2p/go-libp2p@v0.27.1/core/peer#ID)
- [pokt-network/pocket#434](https://github.com/pokt-network/pocket/issue/434)