## Introduction {#introduction}

This document defines the Portable Wallet Profile for Portable Web Spaces
([[PWS]]): the conventions under which a portable digital wallet keeps an
account on a PWS server.

Portability has two halves, and this profile holds both, in separate
sections because a server can satisfy only the first.

* [[[#server-requirements]]] states what a portable wallet needs from any
  PWS server. It names the [[PWS]] and companion-profile versions a wallet
  depends on and, under each, the feature tokens a wallet requires. A
  server that satisfies them may advertise this profile in its service
  description, and a wallet checks for it before trusting the server with
  an account.
* [[[#wallet-data-conventions]]] states how a portable wallet lays out its
  data on that server: the Spaces an account holds and their roles, the
  system collections and records, and the backup bundle an account is
  exported, restored, and moved with. These conventions bind wallet
  implementations, so that a wallet written in any language can read,
  restore, and move an account written by another.

| Specification | Relationship |
|---------------|--------------|
| [[PWS]]       | The storage substrate: Spaces, Collections, Resources, the service description this profile's entry is listed in, and the per-Space export and import primitives the backup bundle wraps. |
| [[PWS-AUTHZ]] | The baseline PWS authorization profile a wallet's capabilities follow. |
| [[PWS-EC]]    | The encrypted-collection construction a wallet's collections follow: the key-epoch roster, the envelope format, and the resource log profile under which a wallet's clients co-manage key resources. |
| [[APP-CONNECT]] | The handshake by which an application obtains capabilities into a wallet's Space. Referenced, not absorbed: it applies whether or not the wallet is portable. |

## Terminology {#terminology}

<dl>
  <dt><dfn data-lt="portable wallets">portable wallet</dfn></dt>
  <dd>
    A digital wallet that keeps its account on a [[PWS]] server under this
    profile's conventions, so that another conforming wallet can read,
    restore, or move the account.
  </dd>
  <dt><dfn data-lt="accounts">account</dfn></dt>
  <dd>
    The set of Spaces a [=portable wallet=] holds for one user on a server,
    together with the identity that controls them.
  </dd>
</dl>

## Server requirements {#server-requirements}

This section states what a [=portable wallet=] needs from any [[PWS]]
server. A server that satisfies every requirement here lists this profile
in its service description under the identifier
[[[#spec-identifier]]] declares.

### This profile's identifier {#spec-identifier}

The persistent identifier of this specification is
`https://w3id.org/pws/wallet-profile`.

That string is the `specs` key a server lists this profile under (see the
[Service Description](https://w3c-ccg.github.io/wallet-attached-storage-spec/#service-description)
of [[PWS]]). It names the specification, not the document's current
location, and it never carries a version. The version entries under the key
carry those. Clients compare the key as an opaque string, with no URL
normalization.

Listing the identifier claims this section alone. A server never claims
[[[#wallet-data-conventions]]]; those conventions bind wallets.

### Required specifications {#required-specifications}

A server that satisfies this profile lists, in its service description's
`specs` object, a version entry for each specification below, and that
entry advertises every feature token named.

| Specification | Identifier | Version | Required feature tokens |
|---|---|---|---|
| [[PWS]] | `https://w3id.org/pws` | `0.5` | `listing`, `collection-management`, `space-management`, `policy`, `export`, `encryption`, `changes-query` |
| [[PWS-EC]] | `https://w3id.org/pws/encrypted-collections` | `0.1` | `blinded-index-query`, `governed-history-logs` |

Under the [[PWS]] rule for a version entry's `features` array, an absent
array or an absent token means the affordance is not served. A conforming
server therefore MUST carry every token above.

Each token group serves a distinct part of a wallet's work against the
server:

* `listing`: enumerating an account's collections and their resources.
* `collection-management` and `space-management`: creating and describing
  the account's Space and its system collections by id, rather than
  through a Spaces Repository. The `spaces` URL member of the [[PWS]]
  entry (see the
  [profile table](https://w3c-ccg.github.io/wallet-attached-storage-spec/#scope-and-conformance-profiles))
  is not required.
* `policy`: reading and setting the access policy on a collection.
* `export`: the per-Space export and import archive the backup bundle
  wraps.
* `encryption`: declaring the encryption scheme of a collection in its
  description.
* `changes-query`: the ordered change feed a wallet's clients replicate
  from.
* `blinded-index-query`: server-side query over blinded index tokens, with
  the unique constraint enforced.
* `governed-history-logs`: the resource log profile under which a
  wallet's clients co-manage key epochs.

See the [[PWS]]
[service description](https://w3c-ccg.github.io/wallet-attached-storage-spec/#service-description)
and
[profile table](https://w3c-ccg.github.io/wallet-attached-storage-spec/#scope-and-conformance-profiles)
sections for the first seven tokens, and the [[PWS-EC]]
[feature tokens](https://interop-alliance.github.io/encrypted-collections-spec/#feature-tokens)
section for the last two.

A wallet relies on two further guarantees that need no token, because
[[PWS]] `0.5` makes them baseline server requirements rather than
advertised affordances: conditional writes and key-epoch stamping.

### Version coupling {#version-coupling}

Each version of this profile pins the versions of [[PWS]] and [[PWS-EC]]
it requires; this version (`0.1`) pins [[PWS]] `0.5` and [[PWS-EC]] `0.1`.
A later version of this profile that requires a different companion
version, or a different token set, is a new version entry. A server MAY
list several versions of this profile at once. Versions follow the
`major.minor` rule of the [[PWS]]
[service description data model](https://w3c-ccg.github.io/wallet-attached-storage-spec/#service-description-data-model).

### The profile's version entry {#version-entry}

The entry a server lists under `https://w3id.org/pws/wallet-profile`
carries exactly two members: `version` (required, `"0.1"` for this
document) and `url` (optional, the version-frozen URL of this document,
under the [[PWS]] rule for a version entry). The entry carries no
`features` array and no other member.

The entry is a conformance claim. The affordances it claims are listed
under the entries named in [[[#required-specifications]]], so the entry
does not become a second route table. A client MUST treat an entry that
carries any other member as absent, following this profile's fail-closed
extensibility rule.

The following shows the `specs` object of a service description that
satisfies this profile:

```json
{
  "specs": {
    "https://w3id.org/pws": [
      {
        "version": "0.5",
        "features": ["listing", "collection-management", "space-management",
                     "policy", "export", "encryption", "changes-query"]
      }
    ],
    "https://w3id.org/pws/encrypted-collections": [
      {
        "version": "0.1",
        "features": ["blinded-index-query", "governed-history-logs"]
      }
    ],
    "https://w3id.org/pws/wallet-profile": [
      {
        "version": "0.1",
        "url": "https://interop-alliance.github.io/portable-wallet-profile-spec/"
      }
    ]
  }
}
```

### The entry is a shorthand {#entry-as-shorthand}

The profile entry is a shorthand for the requirements of
[[[#required-specifications]]]. A client MUST work against a server whose
[[PWS]] and [[PWS-EC]] entries carry every required token, even when the
profile entry is absent.

When the entry is present, a client MAY audit the parts: verify that the
[[PWS]] and [[PWS-EC]] entries each carry every required token. A client
that finds a mismatch MUST treat the server as not satisfying this
profile.

A server MUST NOT list the profile entry unless every requirement of this
section is met.

## Wallet data conventions {#wallet-data-conventions}

<p class="issue">
  To be written. This section covers the Spaces an [=account=] holds and
  their roles, the system collections and records, and the backup bundle an
  account is exported, restored, and moved with.
</p>

## Security considerations {#security-considerations}

<p class="issue">
  To be written.
</p>
