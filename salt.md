# Introduction

Salt is a trust infrastructure service operated by Salt Ltd. It provides verifiable, persistent identifiers for natural persons, organisations, and other entities. The `salt` DID method defines how these identifiers are created, resolved, updated, and deactivated using a web-hosted registry backed by AWS (S3, CloudFront, Lambda, KMS). This specification details how DIDs under the `salt` method are managed by the Salt service.

# Status of This Document

This is a draft document. It may be periodically updated, replaced or removed by other documents at any time.

# Namespace Specific Identifier (NSI)

The Salt DID scheme is defined by the following ABNF:

```
salt-did = "did:salt:" type ":" id
type     = 1*63( %x61-7A / DIGIT / "-" )
id       = 16*64( %x61-7A / DIGIT )
```

The `type` segment identifies the category of subject (e.g. `natural-person`, `organisation`, `agent`). Lowercase letters, digits, and hyphens are permitted. No leading or trailing hyphen and no double hyphens.

The `id` segment is a high-entropy, opaque identifier with no personally identifiable information. Lowercase alphanumeric only, between 16 and 64 characters.

Both segments are case-insensitive and normalised to lowercase.

Examples of valid `salt` DIDs:

```
did:salt:natural-person:xyc123abc456def789
did:salt:organisation:m7kwqr4znb2xp9ghvt3c
did:salt:agent:f8hn2kd4wp6xjm9qvr3b
```

# CRUD Operations

The following operations are supported:

## Create

DIDs and DID Documents are created via the Salt service. An authorised caller submits a request specifying the entity type and initial document content. The Salt service generates a high-entropy opaque identifier and creates the DID Document. Any DID Documents generated as a result of this process are signed using AWS KMS and stored in the Salt registry.

## Read (Resolve)

A registered DID may be resolved by an HTTP GET request against `https://salt.music/salt/<type>/<id>/did.json`. The result of the request is the DID Document that matches the resolved DID, if it exists. Resolution is permissionless.

## Update

DIDs themselves are immutable and cannot be updated. However, as part of routine key rotation or service endpoint changes, a DID Document may be updated. These updates are validated and signed by the Salt service using KMS.

## Deactivate

At an authorised caller's request, a DID may be deactivated. Once processed, the DID Document is marked as deactivated in its metadata. The DID still resolves but returns deactivated status. Deactivation is permanent.

# Security Considerations

This section provides the security considerations for the DID method implementation.

- In case of a key compromise, the controller MUST deactivate the affected DID immediately.
- All communication MUST be encrypted using TLS 1.2 or above.
- DID Document signing is performed by AWS KMS using asymmetric secp256k1 keys. Private key material never leaves KMS.
- Only authorised callers may perform create, update, or deactivate operations. The Salt service maintains an allow list enforced at the API layer.
- S3 write permissions are restricted to the Salt service role only. DID Documents served via CloudFront are read-only to the public.
- The Salt service MUST implement rate limiting to prevent abuse.
- The Salt service SHOULD implement security auditing and logging for all write operations.
- The Salt service MUST ensure that no exploitable vulnerabilities are present in its dependencies.
- Cryptographic keys SHOULD NOT be reused for multiple purposes.

# Privacy Considerations

This section provides the privacy considerations for the DID method implementation.

- No personally identifiable information is stored in a DID or DID Document.
- Identifiers are opaque and high-entropy; they cannot be reverse-engineered to reveal the identity of the subject.
- The Salt service MUST comply with all applicable privacy and data protection laws and regulations.
- The Salt service MUST ensure that no personal data is written to system or application logs.
- DID Documents are publicly resolvable. Controllers should ensure that only information intended for public disclosure is included in a DID Document.
- The Salt service protects against denial-of-service attacks and other intrusion threats.
