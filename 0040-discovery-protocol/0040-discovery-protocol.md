---
title: Interledger Discovery Protocol
type: working-draft
draft: 1
---

# Interledger Discovery Protocol

## Context

ILP does not manage discovery of new peers. Also ILP connections are managed
[bilaterally](../0023-bilateral-transfer-protocol/0023-bilateral-transfer-protocol.md),
via BTP connections, this significantly reduces the discovery and peering
potential. Another potential issue are DDoS attacks which can be prevented by
making gateway system for nodes to enter the network only through relay-nodes
[RFC00](../0041-relays/0040-relays.md), which forwards the ILP specific
information of its peers without exposing the public IP address.

## Terminology

- **node**: a network participant that implements the Interledger Protocol (ILP).
- **settlement engine**: see [RFC 0038](../0038-settlement-engines/0038-settlement-engines.md)
- **carrier-peer**: or carrier-node is a node that has a settlement engine
- **relay-peer**: or relay-node is a node that does not have a settlement engine, see [RFC 0041](../0041-relays/0041-relays.md)
- **bootnode**: a well-known node in the network that facilitates initial peer discovery for new nodes
- **peer_table**: a local table containing information about all known peers in the network
- **routing_table**: a local table containing asset/ledger specific information about peers and optimal forwarding paths
- **node_id**: a unique identifier issued by a Certificate Authority (CA) upon verification, see [RFC 0042](../0042-identity/0042-identity.md)
- **TTL (Time-To-Live)**: a mechanism to prevent propagation loops in peer table updates
- **peer score**: a reputation metric based on peer responsiveness and reliability

## Design Goals

### Verification

Carrier-nodes should not be required to publish direct `ip:port` endpoints to
be discoverable by other carrier-peers. Nodes are unique to a `node_id`, which
is issued by a CA upon verification, see [RFC
0042](../0042-identity/0042-identity.md). This verification should be rechecked
upon a certain event to ensure the certificate has not been revoked by the
entity. The nodes should also be able to notify the CA of malicious contempt of
a peer with proof.

### Relay-Nodes, Privacy and Security

Discovery data must support routing via relay-peers, see [RFC
0041](../0041-relays/0040-relays.md). Relay-peers offer a quasi private way to
participate in the network without exposing public IP information and prevent
targeted attacks.

### Efficient Tracking

Each node maintains a `peer_table` and `routing_table`. The `routing_table`
should contain asset/ledger specific information about its peers, its state
should be efficient wrt. determining the next most efficient hop for ILPv4
packets. The `peer_table` is a table of all known peers of the node, this table
is shared across the network.

### Reputation system

A node should be able to conditionally give a reputation to its peers,
evaluating reliability and trustworthiness. This score is shared through the `peer_table`.

## Startup

The process of booting up a new node is the following:

1. **Node goes through a list of bootnodes**

New nodes can join the network by using a list of **bootnodes**. This list can
either be hardcoded or fetched from an external source. For all the available
bootnodes the new node

2. **Sends a peering request**

The peering request contains information needed for authentication [RFC
0042](../0042-identity/0042-identity.md), upon which the two nodes authenticate
each other.

In the case of a valid, mutual authentication the requesting node

3. **Establishes a WebSocket connection with the peer**

## Peer Relationship

### Peer Communication

When a node establishes a WebSocket connection with a peer, they will not
automatically exchange information but rather subscribe to updates of each
others stats. The available options are.

- `subscribe_peer_table`: receives updates about the peers `peer_table`
- `subscribe_assets`: receives updates about the available asset and exchange
  rates of that specific peer
- `subscribe_peer_table_assets`: receives updates about available assets and
  exchange rates from all the peers in the `peer_table` of that specific peer

The received updates will be then updated in the nodes internal database
system. When receiving information about a new peer not in the local
`peer_table`, the node will go through steps 2 and 3 contacting the new peer

tbc...
Specifics of the subscriptions and or more subscriptions

### Response time ( inactive response or no response)

tbc...
Should remove connections and rank connections based on their response or lack of response

### Rechecking verification

tbc...
Should periodically revalidate identity

## Other

tbc...
update mechanism (peer_table, routing_table), peer score?, churn handling, rate limiting, TTL (prevent propagation loops)

## Specifications

tbc...
What kind of websocket connection, routing table format, peering table format, message format, cryptographic details
