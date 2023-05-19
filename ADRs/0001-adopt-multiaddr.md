# ADR-0001: Adopt Multiaddr <!-- omit in toc -->

- Status: in review
- Deciders: Pocket Network Protocol Team
- Date: 2023-04-17

**Table of Contents**

- [Summary](#summary)
- [Problem Statement](#problem-statement)
- [Technical Story](#technical-story)
- [Decision Drivers](#decision-drivers)
- [Considered Options](#considered-options)
- [Decision Outcome](#decision-outcome)
  - [Positive Consequences](#positive-consequences)
  - [Negative Consequences](#negative-consequences)
- [Pros and Cons of the Options](#pros-and-cons-of-the-options)
  - [Adopting multiaddr](#adopting-multiaddr)
  - [Custom address convention](#custom-address-convention)
- [References](#references)

## Summary 

In the context of simplifying and consolidating node identity, facing the need for a standardized and extensible address format, we decided for adopting multiaddr and neglected a custom address convention, to achieve a future-proof and composable addressing scheme, accepting the additional dependency and learning curve, because multiaddr provides a standard solution with many benefits.

## Problem Statement 

The current addressing system for node identity lacks standardization and extensibility, creating a demand for a unified and future-proof format.

## Background

A GitHub discussion was initiated to decide on the direction of the address convention for the Pocket Network nodes: [discussion #616: Should we go multiaddr](https://github.com/pokt-network/pocket/discussions/616).

## Decision Drivers

- Need for a standardized serialization format
- Requirement for supporting arbitrarily deeply nested protocols
- Demand for supporting arbitrary protocol values, i.e., extensibility
- Confirming and following the newest & best standards in the industry
- Maintenance by the core team

## Considered Options

1. Adopting multiaddr
2. Custom address convention

## Decision Outcome

Chosen option: 1, because it provides a standardized, extensible, and future-proof solution for addressing nodes, while also being widely accepted in the peer-to-peer ecosystem.

### Positive Consequences

- Implementation of standardized and easily parsable node addresses
- Provision of an extensible and composable address format
- Assurance of a future-proof addressing scheme

### Negative Consequences 

- Inclusion of an additional dependency on the multiaddr library
- Necessity for understanding the new abstraction of multiaddr

## Pros and Cons of the Options 

### Adopting multiaddr

Pros:

- Offers a standardized and extensible address format
- Supports arbitrarily deeply nested protocols
- Is a widely adopted solution in the p2p ecosystem

Cons:

- Brings in an additional dependency
- Demands learning a new abstraction

### Custom address convention

Pros:

- Avoids the need for an additional dependency

Cons:

- Could lead to a non-standard and less extensible solution
- Requires development and maintenance efforts

## References 

- [`go-multiaddr` GitHub repo](https://github.com/multiformats/go-multiaddr)
- [`multiaddr` specification](https://github.com/multiformats/multiaddr)
