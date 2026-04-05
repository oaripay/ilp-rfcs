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
settlement accounts (e.g., wallet addresses) and repeat the process. This makes
it impossible for an initial grassroots network to form, because it is exactly
these entry points, where the first connector relationships are formed, that
are the most vulnerable and exploitable.

### Solution

We utilize zero-knowledge proofs through [W3C Verifiable
Credentials](https://www.w3.org/TR/vc-data-model-2.0/) data model deployed on a
EVM-based ledger, the EBSI ecosystem [European Blockchain Services
Infrastructure](https://hub.ebsi.eu). EBSI provides a decentralized registry
solution, which mitigates a single point of failure in comparison to
traditional CA systems. Additionally, it allows node operators to prove their
data validity (e.g. identity, credentials) without revealing sensitive
information. The fundamental components are [Verifiable
Credentials](https://www.w3.org/TR/vc-data-model/#verifiable-credentials) VCs
and [Verifiable
Presentations](https://www.w3.org/TR/vc-data-model/#verifiable-presentations)
VPs.

### EBSI Onboarding

A prerequisite for a node provider willing to participate in the network and
take part in successful verification handshakes, is a Verifiable Credential
(VC) for the Interledger Schema. This allows the node operator to register its
[Digital Identifier](https://www.w3.org/TR/did-1.0/) (DID) into the registry
and present a VP (Verifiable Presentation), associated with the issued VC, to
other node operators and facilitate handshakes. In order for a node operator to
obtain a VC and register a DID, an appropriate issuing entity with allowance to
issue VCs for the Interledger Schema is needed. The associated entity can be
a Root Trusted Authority Organization (RTAO), Trusted Authority Organization
(TAO) or a Trusted Issuer (TI), see [Issuer Trust
Model](https://hub.ebsi.eu/vc-framework/trust-model/issuer-trust-model-v3). In
the following section, we refer to this entity simply as an Issuer and a node
operator as the Subject. The steps of registering a DID are the following

1. Subject generates an ES256 and an ES256K key pair and
   sends the derived DID to the Issuer.
2. The Issuer crafts [Credential
   Subject](https://www.w3.org/TR/vc-data-model/#credential-subject) payload
   with the subject's information obtained through KYC/KYB.
3. Issuer encrypts all fields, except the `id`, of the `credentialSubject` with
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
   3. Extract the embedded VCs from the VP.

2. **Verify the VC (Issuer authentication)**
   1. From the VC, resolve the Issuer DID in the EBSI DID Registry.
   2. Verify the VC signature using the Issuer DID Document key from the DID Registry.
   3. Verify the `credentialStatus` and `credentialSchema` of the Issuer (?).

3. **Check revocation status (Credential validity)**
   1. Use the Issuer `service` entries in the DID registry or `proxies` from
      the Trusted Issuer's Registry to locate the [Legal Entity Credentials
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
identity and proceeds with the peering relationship.

### Legal Entity Credentials Registry

EBSI defines a [Legal Entity Credential Registry
(LECR)](https://hub.ebsi.eu/vc-framework/guidelines/le-credential-registry)
that provides an API to publish and query the status of issued VCs (e.g.,
issued/revoked) allowing verifiers to perform revocation checks during VP/VC
verification. Additionally an API endpoint for the entry node list is provided.

**Issuers MUST provide** a LECR for the Interledger-schema credentials they
issue, and an API endpoint providing the entry node list, so verifiers can
reliably check VC status through the registry defined in the Issuers DID
document as a `service` or a `proxylist` and verified subjects can enter into
the network reliably.

The Endpoints are found in the `service` section under the `serviceEndpoint` of
the issuers DID document, the LECR endpoints are of `type`
`InterledgerIdentityProviderV1`, the entry node list endpoint of `type`
`InterledgerEntryNodeListV1`. Example

```
{
  "id": "io,oari,trust.ilp.identity",
  "serviceEndpoint": "https://trust.oari.io/",
  "type":"InterledgerIdentityProviderV1"
  }
```

```
{
  "id": "io,oari,trust.ilp.nodes",
  "serviceEndpoint": "https://trust.oari.io/nodes",
  "type":"InterledgerEntryNodeListV1"
}
```

For ease of use we refer to the
[ilp-trust](https://github.com/oaripay/ilp-trust) cli to add or remove the defined services
services.

# TODO: Discuss

3. Issuer encrypts all fields, except the `id`, of the `credentialSubject` with
   the RTAO's public key (?).

4. Verify the `credentialStatus` and `credentialSchema` of the Issuer (?).
