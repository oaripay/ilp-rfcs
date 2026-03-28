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

Discovery data must support routing via relay-peers, see [RFC 0041](../0042-identity/0041-identity.md)
