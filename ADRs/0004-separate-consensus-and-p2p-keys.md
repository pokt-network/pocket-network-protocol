# Separate Consensus and P2P Private Keys

* Status: in review
* Deciders: Pocket Network Team
* Date: 2023-04-17

Technical Story: [Consolidate and Refactor Node Identity](https://github.com/pokt-network/pocket/issues/348)

## Context and Problem Statement

In the context of maintaining a secure and robust network, we need to decide whether to use separate private keys for P2P and consensus modules, or use a single private key for both purposes.
Combining the keys could simplify the identity management, but may also increase security risks and impact system quality.

## Decision Drivers

* Security: Minimizing the risks associated with key compromise
* Simplification: Reducing complexity in identity management
* Flexibility: Allowing different key management strategies for different modules
* Isolation: Minimizing the impact of compromise on other system components

## Considered Options

* Option 1: Use a single private key for both P2P and consensus modules
* Option 2: Use separate private keys for P2P and consensus modules

## Decision Outcome

Chosen option: "Option 2: Use separate private keys for P2P and consensus modules", because this approach provides a better security posture, enables greater flexibility in key management strategies, and isolates the impact of potential key compromises.

### Positive Consequences

* Improved security: Compromise of one private key does not directly impact the other module
* Flexibility: Different key management strategies can be applied to each module
* Isolation: Minimizes the impact of compromise on other system components

### Negative Consequences

* Increased complexity: Requires managing separate private keys for different module

## Pros and Cons of the Options

### Option 1: Use a single private key for both P2P and consensus modules

* Good, because it simplifies identity management
* Bad, because a compromise in one module directly impacts the other module
* Bad, because it limits the ability to apply different key management strategies to each module

### Option 2: Use separate private keys for P2P and consensus modules

* Good, because it improves security by isolating potential compromises
* Good, because it allows for greater flexibility in key management strategies
* Bad, because it increases complexity in managing separate private keys for different modules
