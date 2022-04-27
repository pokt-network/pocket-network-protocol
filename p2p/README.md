# RainTree: Pocket Network 1.0 Peer-To-Peer Specification - A Fast, Scalable, Reliable and Optionally Redundant Binary Tree Gossip Algorithm <!-- omit in toc -->

<table style="text-align: center;">
<tr style="text-align: center;">
  <td style="border-collapse: collapse;">
    <p align="center">
        Hamza Ouaghad
        @derrandz<br>
    </p>
  </td>

  <td style="border-width: 0px;">
    <p align="center">
        Andrew Nguyen
        @andrewnguyen22 <br>
    </p>
  </td>

  <td style="border-width: 0px;">
    <p align="center">
        Otto Vargas
        @oten91<br>
    </p>
  </td>

  <td style="border-width: 0px;">
    <p align="center">
        Shawn Regan
        @benvan<br>
    </p>
  </td>
</tr>
</table>

<tr>
<td style="border-width: 0px;">
    <p align="center">
        Version 1.0.2<br>
    </p>
</td>
</tr>

# Abstract <!-- omit in toc -->

<!-- Future edit notes:
  * Need to rewrite this whole Abstract (previously overview) to make it clear - see https://arxiv.org/pdf/1803.05069.pdf as a reference.
  * We use the word `structure` way too many times without explaining what that mans.
  * No reference to RainTree - I still have no clue what this whole spec is about.
-->

Choosing the proper data structure to represent the structure of a network's overlay is the main and crucial step to achieving a structured overlay, and a detrimental one for building an efficient and performant network.

We chose to go with a structured overlay approach for our P2P network to be able to both theorize and evaluate the network's performance, while also managing churn and peer discovery

In this paper, we will explain the structure that will power Pocket Network V1's P2P networking layer.

- [1. Introduction](#1-introduction)
- [2. Requirements](#2-requirements)
- [3. RainTree Specification](#3-raintree-specification)
  - [3.1 Definitions](#31-definitions)
  - [3.2 Network View](#32-network-view)
    - [3.2.1 Global Address Book](#321-global-address-book)
    - [3.2.2 Partial Address Book](#322-partial-address-book)
  - [3.3 RainTree Network Structure](#33-raintree-network-structure)
    - [3.3.1 Maximum Number of Layers](#331-maximum-number-of-layers)
    - [3.3.2 Node Layer](#332-node-layer)
  - [3.4 Message Propagation](#34-message-propagation)
    - [3.4.1 RainTree Gossip Algorithm](#341-raintree-gossip-algorithm)
    - [3.4.2 RainTree Redundancy Layer](#342-raintree-redundancy-layer)
    - [3.4.3 Daisy-Chain Clean Up Layer: Failure Detection & Recovery Layer](#343-daisy-chain-clean-up-layer-failure-detection--recovery-layer)
    - [3.5 Network Churn & Discovery](#35-network-churn--discovery)
      - [3.5.1 Requirements](#351-requirements)
      - [3.5.2 Peer Discovery - Joining a Network](#352-peer-discovery---joining-a-network)
      - [3.5.3 Peer Heartbeat - Leaving a Network](#353-peer-heartbeat---leaving-a-network)
      - [3.5.4 Membership Pings](#354-membership-pings)
- [4. Evaluation & Results](#4-evaluation--results)
  - [4.1 Parameters](#41-parameters)
  - [4.2 Evaluation Results](#42-evaluation-results)
- [5 Transport Layer Protocol And Security](#5-transport-layer-protocol-and-security)
  - [5.1. Message Types](#51-message-types)
  - [5.2 Connection Lifecycle](#52-connection-lifecycle)
  - [5.3 Handshake protocol draft](#53-handshake-protocol-draft)
  - [5.4 Connections Pooling](#54-connections-pooling)
  - [5.5 Protocol](#55-protocol)
  - [5.6 Security](#56-security)
- [References](#references)

# 1. Introduction

<!-- Future Edit Notes:
* The introduction needs to be rewritten altogether.
* Definitions:
  * Need to define what an overlay is.
  * Need to define what a structure is
* Need to add a "Related Work" section including:
  * Tendermint Gossip
  * One-Hop Algorithms
  * PRR
  * Kelpis
  * Gemini
    * Need to provide a background section on Gemini
    * Need to link to results of simulations on Gemini
    * How does this work?
    * Why is it not good enough
    * What are the scalability issues?
-->

During the core Pocket team's research, several structures/algorithms were identified to serve as an overlay for Pocket's V1 P2P networking layer. One of these was [Gemini][1], which builds a network topology using robust estimations of future traffic. However, simulations and projections using Gemini showed that scalability challenges could still be present when the peer count beings to exceed 10,000. As of April 2022, Pocket Network size of more than 40,000 nodes already exceeds the theoretical limits of Gemini.

One of the approaches explored involved using a [Proportional Rate Reduction(PRR)][2] algorithm that fallbacks back to Gemini when a certain threshold is exceeded.

Other approaches explored included One-Hop or Constant-Hops algorithms, such as [Kelips][3]. However, to avoid file-lookup-specific complexities, they would have been needed to be heavily modified.

This paper introduces **RainTree**, a structured tree-based gossip algorithm inspired by [Tendermint's P2P Gossip Algorithm][4].

# 2. Requirements

The following list documents the requirements that the Peer-to-Peer networking layer of Pocket 1.0 must adhere to:

- The network should scale to support at least 1M nodes.
- Network peers must be able to communicate and discover one another deterministically and with low latency.
- One must be able to reason about the network's performance and capacity both theoretically and empirically.
- The network should remain partially asynchronous (i.e. an upper bound on message delays exist) regardless of the payload size.
- The goal of this specification is to guarantee that P2P layer does not act as a limiting factor for Pocket's Consensus and Utility specifications.
- The network should be fault tolerant and self-healing.
- The network should be able to distinguish, organize and prioritize peers based on role or priority.
- The network should be partition tolerant per the [CAP][5] theorem.

# 3. RainTree Specification

<!-- Future Edit Notes:
* Need to explain how IDs are assigned to nodes.
* Things to add to the definition / types section:
  * Numerical ID of a peer: what is it and how it's computed
  * Originator node:
-->

## 3.1 Definitions

- **Global Address Book**: The list of addresses of all participating network peers / actors in the network. See Pocket's [Utility Module][6] for a full list of types of actors.
- **Partial Address Book**: The list of address that a network peer / actor is connected to and is always a strict subset of the Global Address Book.
- **Dead Node**: A network peer / actor that is expected to be online
- **Originator Node**: The first node to send a certain message. It is either the creator for the message or a proxy to an external source that created the message.
- **Layer**: TODO

## 3.2 Network View

In most traditional networking schemes, each node has a partial view of the network (i.e. a fog-of-war like view) and is dependant on a mechanism similar to [Tendermint's Gossip][4] to disseminate a message throughout the network.

Pocket Network is an application specific blockchain where all the actors (fisherman, validators, servicers) are staked and therefore part of the blockchain state. All of these actors are both readers & writers of the state. In turn, every peer participating in message propagation has a full view of the global address book - all the other actors participating in RainTree.

Full nodes (i.e. readers only such as block explorers) can connect to these actors to sync a local state of the blockchain but do not participate in the process of message propagation.

Clients (i.e. wallets) will need to send a message to an Originator Node in order for it to be propagated to the rest of the network.

### 3.2.1 Global Address Book

The global address book is a lexicographically ordered list of addresses of all the actors in the Pocket Network staked actors that participate in message propagation. Address are similar to UUIDs, but are are represented by 42 hex digits. See the [Utility Spec][6] for details on how it is computed.

The ID associated with each node corresponds to its index in this list.

### 3.2.2 Partial Address Book

A partial address book is a strict subset of the global address book that a particular peers / node / actor is connected to at the network layer.

## 3.3 RainTree Network Structure

whose right and left subtrees are the 33rd and the 66th percentiles in the ordered list of peers relative to the root's position in the list, and in and of themselves, these rights and lefts pick their own binary branches using the same logic. So in short, the sorted list has been transformed into a binary tree whose root is the originator's node and whose rights and lefts are always the 33th% and the 66th% of the immediate root and so on and so forth.

The maximum number of layers

- Max possible Layers = log3 of List Size
- Layer of a peer = Round up of the count of the exponents of 3 in the peers ID
- Right branch target = Node position + targetListSize/3 (roll over if needed)
- Left branch target = Node position + targetListSize/ 1.5 (roll over if needed)
- targetListSize = (topLayer + currentLayer) x 0.666 x Size of full list

In such a tree/list, we make use of the concept of right and left branch targets, tree layers and max possible tree layers. We codify these concepts as follows:

You can learn more about how we've come to these conclusions by reading the original presentation document.

![](./raintreediagram.png)

RainTree requires that the peer list / neighbors list be sorted based on the distance metric.

### 3.3.1 Maximum Number of Layers

### 3.3.2 Node Layer

## 3.4 Message Propagation

During message propagation, an originator node is designated as the root of a binary tree

To gossip a message M, RainTree uses a distance-based metric to build a binary tree view of the network and propagates information down the tree. This traversal algorithm follows a tree reconstruction algorithm such that the resulting tree is one of three branches and not two.

The root of the tree is message originator Node S, where the immediate left and right branches are the %33th and %66th positions in the peer list respectively. Node S is said to have `level`, which is the starting layer for the gossip, and it's referred to as the `top layer`.

To determine Node S' layer, we use the following formula:

As mentioned before, `S` will follow a tree reconstruction algorithm to achieve full message propagation, such that when it has delivered message `M` to its left and right, `S` "demotes" itself one level down, and picks a new left and right using the same logic. This demotion goes on until the bottom-most layer (layer 0)

![](raintreedmotion.png)

Each node receiving message `M` will follow the same logic..

### 3.4.1 RainTree Gossip Algorithm

The originator of the message propagates the message as follows:

Let S be the source node of message M, whose peer list is of size N. To propagate a message, S does the following:

1. Determine max possible layers of the network (using current peer list size)
2. Determine own layer (top layer)
3. Determine current layer (top layer - 1)
4. Pick Right and Left branch targets belonging to current layer and send message to them and to self
5. Go to next layer (decrement current layer)
6. Repeat 4 and 5 until current layer is 0

Each peer receiving the message M at a given layer L will replay this procedure.

Origin nodes expect ACKs to be sent back from their lefts and right. Failure to receive an ACK within time-out causes the sender to select next 2 nodes (+1 & -1) in list and resend with decremented level number.

Due to the tricky nature of time-outs, Adjust/Resend only happens once per target. Replacement nodes do not ACK.

![](./sendackreadjust.png)

We call the process of going to the next layer "demotion Logic" as the originator leaves his actual original layer and "demotes" himself to "lower" layers all the way to layer 0.

The algorithm in pseudo-code:

![](raintreealgorithm.png)

### 3.4.2 RainTree Redundancy Layer

An optional redundancy layer can be added to RainTree, such that the originator sends message M to the full list on level 0.

![](raintreeredundanctalgo.png)

So, the algorithm becomes:

![](raintreeredundancyalgofull.png)

This redundancy layer insures against non-participation and incomplete lists without the ACK/RESEND overhead, whereas the reliability layer (Daisy Chain clean-up layer) ensures 100% message propagation in all cases. (See next segment)

### 3.4.3 Daisy-Chain Clean Up Layer: Failure Detection & Recovery Layer

Networks are prone to failure and partitions, so RainTree offers a clean up layer (_a reliability layer_) that ensures that every node has successfully received the message M.

The Daisy Chain Clean Up layer kicks in at level 0, such that nodes receiving messages from level 1 will go asking sequentially every other node of whether they have or want the just-received.

![](igyw.png)

This message is denoted as a `IGYW` message, and it propagates following this algorithm:

a IGYW ( I GOT, YOU WANT?) mechanism is put in use such that a given peer does not fully send a message until the receiving part signals that it did not receive it before.

This is achieved by level 1 nodes, such that once they have received a message, they do the following:

- Send IGYW to immediate left neighbor:
  - If answer is Yes, send full message and go to step 2
  - If answer is No, go to step 2
  - If no answer, increment left counter and go to step 1
- Send IGYW to immediate right neighbour:
  - If answer is Yes, send full message
  - If answer is No,
  - If no answer, increment right counter and go to step 2.

In pseudo-code:

![](igywalgo.png)

Thus, the full algorithm becomes as follows:

![](igywfullalgo.png)

This process is an ACK/ADJUST/RESEND mechanism, for if no ACK was received, an ADJUST instruction takes place, which right after a RESEND instruction is initiated.

### 3.5 Network Churn & Discovery

As with any DHT-like network, some level of network maintenance (also known as membership maintenance/protocol or churn management) is required to keep the network connected.

RainTree is different in that it's similar to Constant Hop networks, in that its churn management process is minimal to non-existent. RainTree requires every member to have a close-to-full view of the network.

#### 3.5.1 Requirements

Any new peer should be able to join the network and participate in it seamlessly. To ensure that our Join/Discovery process achieves this, we would like to answer the following requirements:

Any given random peer should be able to discover other peers in the network from their given current perspective of the network (either their existing state their seed adresse(s))
Any given peer can perform basic discovery and can safely fallback to such a procedure in the absence or presence of specialized peers in the network with no problem at all.

To answer these requirements efficiently, we baked the discovery process into the join process.

#### 3.5.2 Peer Discovery - Joining a Network

When a new peer X joins the network:

It first contacts an existing bootstrap peer(s) E.
Peer(s) E will answer with their peer lists.
Peer X retrieves the lists and performs a raintree propagation of a Join Message with its Address in it denoted as the new joiner.
ACKs can be enforced to keep peers from being filtered from peer XÕs peer list due to lack of response.

This way, when a peer joins, it is immediately given at least one peer list it can start working with, and can by itself clean it up using ACKs and timeouts.

#### 3.5.3 Peer Heartbeat - Leaving a Network

A peer that wants to leave the network basically just disconnects and relies on the maintenance routine to "discover" and broadcast its unavailability.

#### 3.5.4 Membership Pings

RainTree per design can perform flawlessly without a periodic pings protocol, as the Gossip algorithm comes with enough to inform it about the recipients state, however we are interested in implementing a churn management protocol in separation of raintree that further enhances failure detection.

For the time being, this is a work-in-progress.

# 4. Evaluation & Results

## 4.1 Parameters

This will scale! If you tripple the node counts, the only increase is ticks=+2

| Nodes   | Comms   | ACKs    | Ticks |
| ------- | ------- | ------- | ----- |
| 27      | 107     | 56      | 11    |
| 81      | 323     | 164     | 13    |
| 243     | 971     | 488     | 15    |
| 729     | 2,915   | 1,460   | 17    |
| 2,187   | 8,747   | 4,376   | 19    |
| 6,561   | 26,243  | 13,124  | 21    |
| 19,683  | 78,731  | 39,358  | 23    |
| 59,049  | 236,195 | 118,100 | 25    |
| 177,147 | 708,587 | 354,296 | 27    |

## 4.2 Evaluation Results

We will be looking to add some interesting results from a scientific simulation of rain tree available in [rain-tree-simulation](https://github.com/pokt-network/rain-tree-sim) repository.

# 5 Transport Layer Protocol And Security

Transport logic and security are key elements in the inner working of the p2p network. Here we try to outline the general properties and specifications that our network should have and comply with. We also detail some possible attacks that we may be susceptible to.

## 5.1. Message Types

Each protocol will define its messages. Take the following starting index of messages per protocol:

- Membership Protocol
  - Ping
  - Pong
  - Join
  - Leave
- Gossip Protocol (_RainTree)_
  - Gossip
  - GossipACK
  - GossipRESEND
- Daisy-Chain Protocol (_Gossip Reliability Protocol_)
  - IGYW
  - IGYW_AFF
  - IGYW_NEG

## 5.2 Connection Lifecycle

A connection is initiated by the peers
Handshake protocol is initiated and peers exchange secrets to establish a secure encrypted channel.
Messages are then sent on-demand while the connection is alive.
The connection uses a default timeout to ensure that if idle for x amount of time resources are freed and no unnecessary allocations happen.

## 5.3 Handshake protocol draft

1. Perform Diffie-Hellman handshake:
   1. Peers generate ephemeral Ed25519 public and private keys
   2. Peers sign a nonce message and send it with their public key to the other party
      1. Define nonce to be (_for instance_): `_p2p_pokt_network_handshake_`
   3. The bytes order is important and can be as follows: `[pubkey... , 0, signature...]`
2. Peers convert the public keys they received into Curve25519 public keys,
3. Peers convert their ephemeral Ed25519 private keys into Curve25519 private keys,
4. Peers establish a shared secret by performing ECDH with their private Curve25519 private key and their peers Curve25519 public key.
5. Peers exchange the produced shared key as follows:
   1. Peer **A** constructs a message of bytes as follows: `[peer.persistentPubkey..., sharedKey...]`
   2. Peer **A** signs it with its persistent private key and sends it to **B**
   3. Peer **B** decrypts and the message and sends back its public key and shared key in the same format: `[peer.persistentPubkey..., sharedKey...]`
   4. Peer **A** upon receiving the response reconstructs the message with peer **B**'s publickey and the shared secret it produced earlier and verifies it using **B**'s persistent Publickey.
6. Peers use the shared secret as a symmetric key and communicate from then on with messages encrypted/decrypted via AES 256-bit GCM with a randomly generated 12-byte nonce.

## 5.4 Connections Pooling

Connection pooling is required to recycle existing connections and properly utilize the available bandwidth.

The network parameters are theorized for a network bandwidth capacity minimum of 500Mbps (_for both upload and download_).

- Max size of a message is 4MB+DataHeaderSize (_a completely full block_)
- DataHeaderSize describes metadata about the data transmitted, primairly:
  - Size: 4 bytes
  - _insert others if needed_
- Max number of inbound connections is 125 (_each connection consuming 4Mbs_)
- Max number of outbound connections is 125 (_each connection consuming 4Mbs_)

We can possibility open these parameters for external configuration to allow for robust servers to utilize their maximum capacity, but stick to a minimum acceptable network capacity such as the one stated above.

This is a basic bounded connection pool for regular operation with persistent peering options.

We intend to add a specialized bounded pools for application use cases:

- Syncing allowance
- Consensus tasks (validators)
- Others.

## 5.5 Protocol

We will rely on TCP/IP with a handshake for direct communications and Gossip where as we intend to use UDP for churn management communication.

## 5.6 Security

Peer connections could be encrypted using AES 256-bit Galois Counter Mode (GCM) with a Curve25519 shared key established by an Elliptic-Curve Diffie-Hellman Handshake.

Very similar to TLS handshakes.

# References

[1]: Gemini: Practical Reconfigurable Datacenter Networks with Topology and Traffic Engineering (https://arxiv.org/abs/2103.03391)

[2]: Proportional Rate Reduction for TCP (https://research.google/pubs/pub37486/)

[3] Kelips : Building an Efficient and Stable P2P DHT
Through Increased Memory and Background Overhead (https://www.cs.cornell.edu/home/rvr/papers/Kelips.pdf)

[4] The latest gossip on BFT consensus (https://arxiv.org/abs/1807.04938)

[5] CAP theorem (https://en.wikipedia.org/wiki/CAP_theorem)

[6] Pocket 1.0 Utility Module (https://github.com/pokt-network/pocket-network-protocol/tree/main/utility)
