# ADR-0001: Adopt Multiaddr <!-- omit in toc -->

- Status: in review
- Deciders: Pocket Network team
- Date: 2023-04-17

**Table of Contents**

- [Problem Statement](#problem-statement-)
- [Technical Story](#technical-story-)
- [Decision Drivers](#decision-drivers-)
- [Considered Options](#considered-options-)
- [Decision Outcome](#decision-outcome-)
    - [Positive Consequences](#positive-consequences-)
    - [Negative Consequences](#negative-consequences-)
- [Pros and Cons of the Options](#pros-and-cons-of-the-options-)
    - [Adopting multiaddr](#adopting-multiaddr)
    - [Custom address convention](#custom-address-convention)
- [References](#references-)

## Problem Statement <!-- required -->

In the context of simplifying and consolidating node identity, facing the need for a standardized and extensible address format, we decided to adopt multiaddr and neglect alternative address conventions, to achieve a future-proof and composable addressing scheme, accepting the additional dependency and learning curve, because multiaddr provides a standard solution with many benefits.

## Technical Story <!-- optional -->

[GitHub discussion #616: Should we go multiaddr](https://github.com/pokt-network/pocket/discussions/616)

## Decision Drivers <!-- optional -->

- Standardized serialization format
- Supporting arbitrarily deeply nested protocols
- Supporting arbitrary protocol values, i.e., extensible

## Considered Options <!-- required -->

1. Adopting multiaddr
2. Custom address convention

## Decision Outcome <!-- required -->

Chosen option: "Adopting multiaddr", because it provides a standardized, extensible, and future-proof solution for addressing nodes.

### Positive Consequences <!-- optional -->

- Standardized and easily parsable node addresses
- Extensible and composable address format
- Future-proof addressing scheme

### Negative Consequences <!-- optional -->

- Additional dependency on the multiaddr library
- Learning curve for understanding multiaddr abstraction

## Pros and Cons of the Options <!-- required -->

### Adopting multiaddr

Pros:

- Provides a standardized and extensible address format
- Supports arbitrarily deeply nested protocols
- Widely adopted solution in the p2p ecosystem

Cons:

- Introduces an additional dependency
- Requires learning a new abstraction

### Custom address convention

Pros:

- Doesn't require an additional dependency

Cons:

- May lead to a non-standard and less extensible solution
- Requires development and maintenance efforts

## References <!-- optional -->

- [`go-multiaddr` GitHub repo](https://github.com/multiformats/go-multiaddr)
- [`multiaddr` specification](https://github.com/multiformats/multiaddr)