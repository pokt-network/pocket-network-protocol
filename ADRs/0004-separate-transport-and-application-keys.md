# Separate of Transport and Application Keys

- Status: in review
- Deciders: Pocket Network Protocol Team
- Date: 2023-05-19

**Table of Contents**

- [Summary](#summary)
- [Problem Statement](#problem-statement)
- [Background](#background--origin-document--technical-story-)
- [Decision Drivers](#decision-drivers-)
- [Considered Options](#considered-options-)
- [Decision Outcome](#decision-outcome-)
    - [Positive Consequences](#positive-consequences-)
    - [Negative Consequences](#negative-consequences-)
- [Pros and Cons of the Options](#pros-and-cons-of-the-options-)
    - [Option 1: Use a single private key for both P2P and other functionalities](#option-1)
    - [Option 2: Use separate private keys for P2P and other functionalities](#option-2)
- [References](#references-)

## Summary

In the context of maintaining a secure and robust network, facing the concern of key compromise, we decided for Option 2, separating keys between P2P and other functionalities (consensus, utility etc.), and neglected Option 1, to achieve better security posture, flexibility, and isolation, accepting the increased complexity in identity management, because this approach provides a more holistic and secure solution for the network.

## Problem Statement

We need to decide whether to use separate private keys for P2P and other functionalities (like consensus and utility related logic), or use a single private key for all purposes. Combining the keys could simplify the identity management, but may also increase security risks and impact system quality.

## Background

[Consolidate and Refactor Node Identity](https://github.com/pokt-network/pocket/issues/348)

## Decision Drivers

- Security: Minimizing the risks associated with key compromise
- Simplification: Reducing complexity in identity management and node configuration
- **Flexibility/Modularity**: Allowing different key management strategies for different modules
- Isolation: Minimizing the impact of compromise on other system components
- Optionality: Enabling future changes and extensions to the protocol

## Considered Options

1. Use a single private key for both P2P and other functionalities (i.e. consensus, utility, etc...)
2. Use separate private keys for P2P and other functionalities

## Decision Outcome

Chosen option: 2, because this approach provides a better security posture, enables greater flexibility in key management strategies, and isolates the impact of potential key compromises.

### Positive Consequences

- Improved security: Compromise of one private key does not directly impact the other functionalities
- Flexibility: Different key management strategies can be applied to each functionality

### Negative Consequences

- Increased complexity: Requires managing separate private keys for different functionalities

## Pros and Cons of the Options

### Option 1: Use a single private key for both P2P and other functionalities

Pros:

- Simplifies identity management

Cons:

- A compromise in one functionality directly impacts the other functionalities
- Limits the ability to apply different key management strategies to each functionality

### Option 2: Use separate private keys for P2P and other functionalities

Pros:

- Improves security by isolating potential compromises
- Allows for greater flexibility in key management strategies

Cons:

- Increases complexity in managing separate private keys for different functionalities

## References

- [Consolidate and Refactor Node Identity](https://github.com/pokt-network/pocket/issues/348)
