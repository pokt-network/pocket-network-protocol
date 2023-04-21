# ADR-0001: Adopt Multiaddr

# Adopt Multiaddr for Addressing and Protocol Handling

* Status: in review
* Deciders: Pocket Network team
* Date: 2023-04-17

Technical Story: GitHub discussion [#616: Should we go multiaddr](https://github.com/pokt-network/pocket/discussions/616)

## Context and Problem Statement

In the context of simplifying and consolidating node identity, facing the need for a standardized and extensible address format, we decided to adopt multiaddr and neglect alternative address conventions, to achieve a future-proof and composable addressing scheme, accepting the additional dependency and learning curve, because multiaddr provides a standard solution with many benefits.

## Decision Drivers

* Standardized serialization format
* Supporting arbitrarily deeply nested protocols
* Supporting arbitrary protocol values, i.e., extensible

## Considered Options

* Adopting multiaddr
* Custom address convention

## Decision Outcome

Chosen option: "Adopting multiaddr", because it provides a standardized, extensible, and future-proof solution for addressing nodes.

### Positive Consequences

* Standardized and easily parsable node addresses
* Extensible and composable address format
* Future-proof addressing scheme

### Negative Consequences

* Additional dependency on the multiaddr library
* Learning curve for understanding multiaddr abstraction

## Pros and Cons of the Options

### Adopting multiaddr

* Good, because it provides a standardized and extensible address format
* Good, because it supports arbitrarily deeply nested protocols
* Good, because it is a widely adopted solution in the p2p ecosystem
* Bad, because it introduces an additional dependency
* Bad, because it requires learning a new abstraction

### Custom address convention

* Good, because it doesn't require an additional dependency
* Bad, because it may lead to a non-standard and less extensible solution
* Bad, because it requires development and maintenance efforts

### Links

* [`go-multiaddr` GitHub repo](https://github.com/multiformats/go-multiaddr)
* [`multiaddr` specification](https://github.com/multiformats/multiaddr)
