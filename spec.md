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

<p class="issue">
  To be written. This section names the [[PWS]] and [[PWS-EC]] versions a
  portable wallet requires and, under each entry, the feature tokens a
  wallet depends on; and declares this profile's persistent identifier,
  under which a server that satisfies them lists a version entry in its
  service description.
</p>

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
