---
title: Interledger Connector Identity Infrastructure
type: working-draft
draft: 1
---
# Interledger Connector Identity Infrastructure

### Problem

Connectors can cheat the system. They can steal value in various ways outlined in [#0018](https://github.com/interledger/rfcs/blob/main/0018-connector-risk-mitigations/0018-connector-risk-mitigations.md). If caught, they can easily change their public address, name, IP and associated settlement accounts (e.g wallet addresses) and repeat the process. This makes it impossible for an initial grassroots network to form, because it is exactly these entry points, where the first connector relationships are formed, that are the most vulnerable and exploitable.

### Solution

