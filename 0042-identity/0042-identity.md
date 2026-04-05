---
title: Interledger Connector Identity Infrastructure
type: working-draft
draft: 1
---

# Interledger Connector Identity Infrastructure

### Problem

Connectors can cheat the system. They can steal value in various ways outlined
in [#0018](https://github.com/interledger/rfcs/blob/main/0018-connector-risk-mitigations/0018-connector-risk-mitigations.md).
If caught, they can easily change their public address, name, IP and associated
settlement accounts (e.g wallet addresses) and repeat the process. This makes
it impossible for an initial grassroots network to form, because it is exactly
these entry points, where the first connector relationships are formed, that
are the most vulnerable and exploitable.

### Solution

We utilize zero-knowledge-proofs through [W3C Verifiable
Credentials](https://www.w3.org/TR/vc-data-model-2.0/) data model deployed on a
decentralized EVM based ledger, the EBSI ecosystem [European Blockchain
Services Infrastructure](https://hub.ebsi.eu). EBSI serves a decentralized
registry, which mitigates a single point of failure in comparison to
traditional CA systems. Additionally allowing node operators to prove their
data validity (e.g. identity, credentials) without revealing sensitive
information. The fundamental components are [Verifiable
Credentials](https://www.w3.org/TR/vc-data-model/#verifiable-credentials) VC(s)
and [Verifiable
Presentations](https://www.w3.org/TR/vc-data-model/#verifiable-presentations)
(VPs).

### EBSI Onboarding

A prerequisite for a node provider willing to participate in the network and
take part in successful handshakes, is a Verifiable Credential (VC) for the
Interledger Schema. This allows the node operator to register its [Digital
Identifier](https://www.w3.org/TR/did-1.0/) (DID) into the registry and present
a VP (Verifiable Presentation), associated with the issued VC, to other node
operators and facilitate handshakes. In order for a node operator to get a VC
and register a DID, an appropriated issuing entity with allowance to issue VC(s)
for the Interledger Schema is needed. The associated entity can be a Root
Trusted Authority Organization (RTAO), Trusted Authority Organization (TAO)
or a Trusted Issuer (TI), see [Issuer Trust
Model](https://hub.ebsi.eu/vc-framework/trust-model/issuer-trust-model-v3).
In the following section, we refer to this entity simply as an Issuer and a
node operator as the Subject. The steps of registering a DID are the following

1. Subject generates a ES256 and a ES256K key-pair and
   sends the derived DID to the Issuer.
2. The Issuer crafts [Credential
   Subject](https://www.w3.org/TR/vc-data-model/#credential-subject) payload.
   with the subject's information obtained through KYC/KYB.
3. Issuer encrypts all fields, except the `id`, of the `credentialSubject` with.
   the RTAO's public key (?).
4. Issuer creates a VC payload with the `credentialSubject` payload.
5. Issuer signs the VC payload with the ES256 private key and hands it to the
   subject.
6. Subject uses the signed VC to register the DID document.

For ease of use we refer to the [ilp-trust](https://github.com/oaripay/ilp-trust) cli.

### Resolution & Handshakes

When establishing a peer relationship, the nodes mutually prove their
identities and eligibility by presenting a VP to the counterparty.

The verifier resolves and validates keys and credentials in the following
order:

1. **Verify the VP (Subject authentication)**
   1. Resolve the Subject DID in the EBSI DID Registry (DID Document lookup).
   2. Verify the VP signature using a public key referenced from the Subject
      DID Document.
   3. Extract the embedded VC(s) from the VP.

2. **Verify the VC (Issuer authentication)**
   1. From the VC, resolve the Issuer DID in the EBSI DID Registry.
   2. Verify the VC signature using the Issuer DID Document key from the DID Registry.
   3. Verify the `credentialStatus` and `credentialsSchema` of the Issuer (?).

3. **Check revocation status (Credential validity)**
   1. Use the Issuer `service` entries in the DID registry or `proxies` from
      the Trusted Issuers Registry to locate the [Legal Entity Credentials
      Registry (LECR)](https://hub.ebsi.eu/vc-framework/guidelines/le-credential-registry).
   2. Confirm the VC is not revoked in LECR.

4. **Verify the Issuer trust chain (Recursive trust validation)**
   1. Validate that the Issuer is permitted to issue for the Interledger schema
      per the EBSI issuer trust model.
   2. Recursively verify the Issuer’s own credential(s) and their status
      (including revocation/status checks) up to the applicable trust anchor(s)
      (e.g., TAO/RTAO), failing the handshake if any link in the chain is
      invalid.

Only if all checks pass does the verifier accept the Subject’s resolved
identity and proceed with the connector handshake (binding the connection to
the validated DID/keys).

### Legal Entity Credentials Registry

EBSI defines a [Legal Entity Credential Registry
(LECR)](https://hub.ebsi.eu/vc-framework/guidelines/le-credential-registry)
that provides a standardized mechanism to publish and query the status of
issued VCs (e.g., issued/revoked), allowing verifiers to perform revocation
checks during VP/VC verification.

In this proposal:

- **Issuers MUST provide** an LECR for the Interledger-schema credentials they
  issue, so verifiers can reliably check VC status through the registry defined
  in the Issuers DID document as a `service` or a `proxylist`.

- The registry is also used for network discovery metadata: **bootnodes can be
  found there**, allowing new nodes to bootstrap by fetching the bootnode list
  and validating it through the same issuer trust-chain verification described
  above. TODO: specs
