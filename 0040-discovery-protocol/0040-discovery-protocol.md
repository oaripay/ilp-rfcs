---
title: Interledger Discovery Protocol
type: working-draft
draft: 1
---

# Interledger Discovery Protocol

## Terminology

- **node**: a network participant that implements the Interledger Protocol (ILP).
- **settlement engine**: see [RFC 0038](../0038-settlement-engines/0038-settlement-engines.md)
- **carrier-peer**: or carrier-node is a node that has a settlement engine
- **relay-peer**: or relay-node is a node that does not have a settlement engine

## Design Goals

### WebSocket

Discovery must operate over WebSocket sessions.

### Bootstrapable

New nodes can join the network by using a list of bootnodes. The bootnodes can either be hardcoded
or be fetched from an external source.

### Privacy

Carrier-nodes should not be required to publish direct `ip:port` endpoints to be discoverable by other carrier-peers.
Nodes are unique to a `node_id`, which is issued by a CA upon verification, see [RFC 0042](../0042-identity/0042-identity.md).

### Relay Compatibility

Discovery data must support routing via relay-peers, see [RFC 0041](../0041-relays/0040-relays.md)

## Overview

The process of a new node is the following:

1. Node boots
2. Node goes through the list of bootnodes
3. Node establishes a connection to a peer node through a WebSocket
4. Peer authentication occurs using [RFC 0042](../0042-identity/0042-identity.md)
5. Node subscribes to discovery topics and exchange announcements
6. Node maintains a routing/discovery table keyed by `node_id` with a possible expiry deadline

Upon establishing a WebSocket connection with its bootnodes the new node will
receive subscribed updates through the WebSocket connection. The following are viable options

1. `subscribe_routing_table`: receives updates about the peers routing table
2. `subscribe_assets`: receives updates about the available asset and exchange rates of specific the peer
3. `subscribe_routing_table_assets`: receives updates about available assets and exchange rates from all the peers in the routing table of that specific peer

The received updates will be then updated in the nodes internal database system.
Upon receiving a peers `routing_table`, if a new peer is available, the node will try and establish a WebSocket connection with authentication and subscription.
