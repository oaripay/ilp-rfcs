---
title: Interledger Discovery Protocol
type: working-draft
draft: 1
---

# Interledger Discovery Protocol

## Problem

## Terminology

- **node**: a network participant that implements the Interledger Protocol (ILP).
- **settlement engine**: see [RFC 0038](../0038-settlement-engines/0038-settlement-engines.md)
- **carrier-peer**: or carrier-node is a node that has a settlement engine
- **relay-peer**: or relay-node is a node that does not have a settlement engine

## Design Goals

### Privacy and Security

Carrier-nodes should not be required to publish direct `ip:port` endpoints to be discoverable by other carrier-peers.
Nodes are unique to a `node_id`, which is issued by a CA upon verification, see [RFC 0042](../0042-identity/0042-identity.md).

### Relay-Nodes

Discovery data must support routing via relay-peers, see [RFC 0041](../0041-relays/0040-relays.md)

### Efficient Peer State

Each node maintains a `peer_table` and `routing_table`. The `routing_table` should contain asset/ledger specific information about its peers,
its state should be efficient wrt. determining the next most efficient hop for ILPv4 packets. The `peer_table`
is a table of all known peers of the node, this table is shared across the network.

## Startup

The process of booting up a new node is the following:

1. Node goes through a list of **bootnodes**

New nodes can join the network by using a list of **bootnodes**. This list can either be hardcoded
or fetched from an external source. For all the available bootnodes the new node

2. sends a peering request

The peering request contains information needed for authentication [RFC 0042](../0042-identity/0042-identity.md),
upon which the two nodes authenticate each other. In the case of a valid authentication the new node

3. establishes a WebSocket connection with its peer.

## Peer State

When a node establishes a WebSocket connection with a peer, they will not automatically exchange information
but rather subscribe to updates of each others stats. The available options are.

- `subscribe_peer_table`: receives updates about the peers `peer_table`
- `subscribe_assets`: receives updates about the available asset and exchange rates of that specific peer
- `subscribe_peer_table_assets`: receives updates about available assets and exchange rates from all the peers in the `peer_table` of that specific peer

The received updates will be then updated in the nodes internal database system.
When receiving information about a new peer not in the local `peer_table`, the node will go through steps 2 and 3 contacting the new peer
