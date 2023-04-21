# ADR-0002: Separate consensus and P2P IDs

* Status: in review
* Deciders: Pocket Network team
* Date: 2023-04-17

Technical Story: [Review Hotstuff Node IDs](https://github.com/pokt-network/pocket/issue/434)

## Context and Problem Statement

In the context of simplifying and consolidating node identity, facing the concern of multiple ID definitions, we decided to clarify the purpose of consensus NodeId and not consolidate it with P2P identity, to achieve a clearer separation of concerns, accepting potential confusion between different IDs, because this will prevent unintended interference between consensus and P2P layers.

## Decision Drivers

* Clear separation of concerns between consensus and P2P layers
* Preventing unintended interference between different node identity types

## Considered Options

* Consolidate consensus NodeId with P2P identity
* Keep consensus NodeId and P2P identity separate

## Decision Outcome

Chosen option: "Keep consensus NodeId and P2P identity separate", because it prevents unintended interference between consensus and P2P layers while maintaining a clear separation of concerns.

### Positive Consequences

* Clear separation of concerns between consensus and P2P layers
* Avoids unintended interference between different node identity types

### Negative Consequences

* Potential confusion between different node identity types
* Maintaining multiple ID definitions

## Pros and Cons of the Options

### Consolidate consensus NodeId with P2P identity

* Good, because it simplifies node identity management
* Bad, because it may lead to confusion and unintended interference between consensus and P2P layers

### Keep consensus NodeId and P2P identity separate

* Good, because it maintains a clear separation of concerns between consensus and P2P layers
* Good, because it avoids unintended interference between different node identity types
* Bad, because it may lead to potential confusion between different node identity types
* Bad, because it requires maintaining multiple ID definitions

### Links

* [pokt-network/pocket#348](https://github.com/pokt-network/pocket/issue/348)