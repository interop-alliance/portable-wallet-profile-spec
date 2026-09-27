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
  <dt><dfn data-lt="backup bundles|bundle|bundles">backup bundle</dfn></dt>
  <dd>
    One file holding an [=account=]'s server-held state: a per-Space export
    archive for every Space the account names, wrapped with a manifest. See
    [[[#backup-bundle]]].
  </dd>
  <dt><dfn data-lt="backup credentials">backup credential</dfn></dt>
  <dd>
    An unlock method an export establishes for its [=backup bundle=]. Its
    unlock secret is 32 random bytes, which the bundle packs. See
    [[[#export-backup-credential]]].
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

This section states how a [=portable wallet=] lays out an [=account=] on a
[[PWS]] server. It binds wallet implementations. A server claims none of it
(see [[[#spec-identifier]]]).

<p class="issue">
  Partially written. The Space roles and their <code>type</code> arrays, the
  system collections, and the record shapes are still to come. The backup
  bundle is specified below.
</p>

### The backup bundle {#backup-bundle}

A [=backup bundle=] is one file holding an [=account=]'s server-held state.
It is what an account is exported, restored, and moved with.

[[PWS]] exports one Space at a time, through the per-Space export operation
(`POST /space/{space_id}/export`), and takes one back through the matching
import operation (`POST /space/{space_id}/import`). That primitive knows
nothing about accounts. A bundle is assembled by the wallet: it calls the
primitive once per Space the account names and wraps the results.

The assembly is wallet work because the server cannot do it. The registry
that names an account's Spaces is sealed to the account's user key, so which
Spaces make up an account is knowledge only the wallet holds. A server-side
bundle route is therefore out of scope for [[PWS]] and for this profile.

#### The Spaces a bundle covers {#bundle-space-set}

A bundle MUST carry every Space the [=account=] names:

* the account Space;
* every unlock Space the unlock-methods registry names, passphrase, passkey,
  and backup credential records alike;
* every recovery-code Space;
* the client annex Space.

The rule is "every Space the account names", with no special case. The annex
is included on the same footing as the rest. Every grant a transient session
minted has the annex DID as the middle link of its chain, resolved only out
of the annex log, and the account log's `#DelegatedClients` pointer names the
annex DID.

A wallet MUST NOT write a bundle that omits a named Space. A bundle missing
one Space reads as complete and is not, which is worse for its holder than no
bundle at all. An export whose per-Space export fails therefore fails whole.

#### Layout {#bundle-layout}

A bundle is a tar archive. Its entries are, in order:

```
manifest.yml           the bundle manifest (see below)
backup-credential.json the packed backup credential, when present
spaces/                the Space archive directory
spaces/<spaceId>.tar   one per-Space export archive, verbatim bytes
```

The Space archives appear in the order the manifest lists them.

The layout is tar-in-tar on purpose. A wallet neither packs nor unpacks a
per-Space archive: an export streams the primitive's response into the outer
entry, and a restore streams the entry into the import operation. The inner
archives are opaque to the outer layer.

Every tar header's `mtime` is pinned to the Unix epoch, which is the value the
per-Space archives already use. A bundle over unchanged inputs is therefore
byte-reproducible, because the inner exports already are.

Nothing server-wide travels in the outer layer. Each per-Space archive is
self-describing: it carries the exporting server's service description beside
its own manifest.

The per-Space archive's internal layout -- its file-name codec, its dot-files,
its manifest, and its entry order -- is specified in a section of its own,
which this version of the profile does not yet carry.

#### The manifest {#bundle-manifest}

The bundle's `manifest.yml` is a FEP-6fcd manifest. It carries four top-level
members.

<dl>
  <dt><code>ubc-version</code></dt>
  <dd>
    The manifest dialect's version, <code>"0.1"</code> for the layout this section
    specifies.
  </dd>
  <dt><code>meta</code></dt>
  <dd>
    The bundle's provenance, using the FEP's own members and no others:
    <code>created</code>, the ISO datetime the bundle was written; <code>createdBy.controller</code>,
    the account's <code>did:webvh</code> string; and <code>createdBy.client</code>, an object
    <code>{ name, url }</code> naming the wallet that wrote the file. There is no version
    member. The server's version travels in each inner archive's service
    description, and the wallet's is not needed for a restore.
  </dd>
  <dt><code>spec</code></dt>
  <dd>
    The one profile version whose bundle layout the file follows, as
    <code>{ id, version, url }</code>. The <code>id</code> is this profile's persistent identifier,
    <code>https://w3id.org/pws/wallet-profile</code>; the <code>version</code> is the profile
    version; the <code>url</code> is where that version is published. This is the one
    top-level member beyond the FEP's, and it is singular. A service
    description's <code>specs</code> lists every specification a server speaks. A bundle
    follows exactly one.
  </dd>
  <dt><code>contents</code></dt>
  <dd>
    The FEP's file tree. Each entry's <code>url</code> names the specification of what
    that entry is.
  </dd>
</dl>

The `contents` tree annotates the manifest itself with FEP-6fcd's manifest-file
section, the packed backup credential with [[[#packed-backup-credential]]], the
`spaces/` directory with [[[#space-archives]]], and each
`spaces/<spaceId>.tar` with the archive role that Space plays. Archive file
names are the bare `<spaceId>.tar`. The role rides the `url` alone.

```yaml
ubc-version: "0.1"
meta:
  created: "2026-09-20T17:04:11.000Z"
  createdBy:
    controller: did:webvh:...
    client:
      name: freewallet
      url: https://github.com/interop-alliance/freewallet
spec:
  id: https://w3id.org/pws/wallet-profile
  version: "0.1"
  url: https://interop-alliance.github.io/portable-wallet-profile-spec/
contents:
  manifest.yml:
    url: https://codeberg.org/fediverse/fep/src/branch/main/fep/6fcd/fep-6fcd.md#manifest-file
  backup-credential.json:
    url: https://interop-alliance.github.io/portable-wallet-profile-spec/#packed-backup-credential
  spaces:
    url: https://interop-alliance.github.io/portable-wallet-profile-spec/#space-archives
    contents:
      - <accountSpaceId>.tar:
          url: https://interop-alliance.github.io/portable-wallet-profile-spec/#account-space-archive
      - <annexSpaceId>.tar:
          url: https://interop-alliance.github.io/portable-wallet-profile-spec/#client-annex-space-archive
      - <unlockSpaceId>.tar:
          url: https://interop-alliance.github.io/portable-wallet-profile-spec/#unlock-space-archive
```

A reader MUST match a role URL as an opaque string. A `contents` entry
carrying an unrecognized role URL is not read as a Space archive of a known
kind, following this profile's fail-closed extensibility rule.

#### Space archives {#space-archives}

The `spaces/` directory holds one per-Space export archive per Space the
bundle covers, at `spaces/<spaceId>.tar`, with the primitive's bytes verbatim.
A bundle re-encodes nothing.

Every archive carries exactly one Space, and every Space's archive carries one
of the three roles the next three sections define. A bundle carries exactly one
account Space archive, at most one client annex Space archive, and one unlock
Space archive per unlock method the account names.

<p class="note">
  The section ids `#space-archives`, `#account-space-archive`,
  `#client-annex-space-archive`, `#unlock-space-archive`, and
  `#packed-backup-credential` are permanent wire text. A written bundle names them
  in its manifest, and nothing else in the bundle marks which archive is
  which. Each id is pinned explicitly in its heading, so rewording a title
  never moves it. They are not renamed.
</p>

#### The account Space archive {#account-space-archive}

Contents: the account Space, whole. Its encrypted collections' envelopes, each
collection's Metadata object and governing history log, the `id` collection
with the account's `did.jsonl`, and the `key-map` collection with the user-key
roster `key-map/user-key.jsonl` and `key-map/keys.json`.

Restore controller: the restoring session's `did:key`. The Space is created at
its original id under that key, which is its controller for the length of the
import. The controller is promoted to the account's `did:webvh` once the
import has landed, so the account log the promotion is checked against is in
place first.

Place in the restore order: second, after the acting credential's own unlock
Space and before the client annex Space.

#### The client annex Space archive {#client-annex-space-archive}

Contents: the client annex Space, whose log records the per-visit verification
methods a transient session signs under. It resolves the middle link of every
grant such a session minted. The account log's `#DelegatedClients` pointer
names the annex DID, and that DID embeds this Space's id, so the annex cannot
be re-created elsewhere without a new pointer entry.

Restore controller: the same shape as the account Space. The Space is created
at its original id under the restoring session's `did:key`, typed as a
delegated-clients Space, imported, and its controller promoted to the account
DID.

Place in the restore order: third, after the account Space and before the
login.

A bundle that carries no annex archive, or an annex import that fails, leaves
the account converging through the wallet's own heal path: a fresh annex Space
and a signed re-point of the account log's pointer. That path is lossy for
live grants. It is the mender for a partial restore rather than the design.

#### An unlock Space archive {#unlock-space-archive}

Contents: one unlock Space, holding the unlock record of one unlock method.
The record locates the account and carries the method's latent authority. It
carries no content key.

A backup credential's unlock Space archive, like a recovery code's, is an
unlock Space archive. Passphrase, passkey, recovery-code, and backup credential
Spaces take the same role URL, so the bundle's manifest exposes no credential
kind. The record inside names the kind, and only a
holder of that record's secret can read it.

Restore controller: the unlock identity `did:key` the method's secret derives,
which is the controller of its own Space. How that controller acts splits the
role across two places in the restore order.

Place in the restore order: the acting credential's own unlock Space is
restored first of all, created and imported under that credential's own
signature. Every other unlock Space is restored last, after the login, by the
logged-in session invoking the management capability the unlock-methods
registry holds for that entry. Each such Space is created at its original id
with `controller` set to the entry's unlock identity, then imported.

The late placement is forced. The management capability is a chain whose
middle link names the account DID, and that link resolves only once the
account log is restored.

Restoring a sibling this way does not need the sibling's secret, which is the
whole point of the management capability: it manages a lost method. If a
sibling archive is missing, or its import fails, that method's next login
refuses and the account's registry walk reports the entry unreachable. The
user re-adds the method.

#### The packed backup credential {#packed-backup-credential}

`backup-credential.json` is an optional entry at the bundle's root, beside the
manifest. It carries the secret of the [=backup credential=] the export
established (see [[[#export-backup-credential]]]). The secret is 32 random
bytes, encoded base64url with no padding.

The document takes one of two forms, distinguished by a `form` member. A reader
MUST refuse a document whose `form` is neither of the two below, and a `secret`
that does not decode to exactly 32 bytes.

The plain form carries the secret as written:

```json
{ "form": "plain", "secret": "<32 random bytes, base64url, unpadded>" }
```

The sealed form carries the secret encrypted to an export passphrase:

```json
{
  "form": "sealed",
  "kdf": {
    "version": 2,
    "algorithm": "Argon2id",
    "memory": 65536,
    "passes": 3,
    "parallelism": 1,
    "salt": "<16 random bytes, base64url, unpadded>"
  },
  "encryption": { "...": "a one-epoch encryption descriptor" },
  "wrapped": { "...": "the envelope of { secret }" }
}
```

<dl>
  <dt><code>kdf</code></dt>
  <dd>
    The derivation from the export passphrase to a 32-byte seed. The
    parameters are the wallet's own passphrase parameter set: Argon2id with
    64 MiB of memory (<code>memory</code> is in KiB, per [[RFC9106]]), 3 passes, and
    parallelism 1. The <code>salt</code> is 16 random bytes, fresh per bundle, encoded
    base64url with no padding.
  </dd>
  <dt><code>encryption</code></dt>
  <dd>
    A [[PWS-EC]] encryption descriptor with one epoch and one recipient: the
    key-agreement key of the standing identity the seed derives. Nothing else
    can unwrap the epoch's secret.
  </dd>
  <dt><code>wrapped</code></dt>
  <dd>
    The envelope of the JSON object <code>{ secret }</code>, sealed under that
    descriptor through the same record construction an unlock record is sealed
    with. Its <code>secret</code> member is the base64url string the plain form
    carries.
  </dd>
</dl>

The per-bundle random salt is a deliberate departure from the fixed, app-wide
salt a wallet passphrase's derivation uses. It keeps one precomputed guess
list from amortizing across bundles. The export passphrase is its own secret.
It does not have to match the wallet passphrase, and it may be reused across
exports without the two bundles sharing a derived key.

A reader given a sealed document and no export passphrase refuses. A reader
given the wrong passphrase derives a different key-agreement key, for which
the sealed epoch holds no wrap, and the unwrap refuses.

The secret derives the credential's unlock seed through HKDF with SHA-256
[[RFC5869]]. The input keying material is the 32 bytes the `secret` string
decodes to. The salt is the UTF-8 string
`freewallet/keyring/backup-credential/v1`, the info is the UTF-8 string
`freewallet/unlock-seed`, and the output is 32 bytes.

The unlock seed derives the unlock identity `did:key` and its unlock Space the
way every other unlock method's seed does. A reader holding the secret
therefore locates the credential's unlock Space archive among the bundle's
archives by its Space id.

The derivation applies no memory-hard stretching, because the 32 bytes are
already uniform key material. Its salt differs from every other unlock
method's, so the secret cannot derive another method's unlock Space. The salt
is permanent wire text. Changing it would orphan every credential a written
bundle packs.

#### The restore order {#restore-order}

A restore runs the Spaces in one order:

1. The acting credential's own unlock Space. Created at its original id under
   that credential's unlock `did:key`, then imported, then read. A wrong
   credential or another account's bundle is refused by the record's own
   binding, which leaves one inert Space the same key can delete.
2. The account Space. Created at its original id under the session's
   `did:key`, imported, and its controller promoted to the account
   `did:webvh`.
3. The client annex Space. The same shape, typed as a delegated-clients Space.
4. An ordinary login, on the restored account.
5. Every remaining unlock Space, as that session, through the management
   capability the unlock-methods registry holds for each entry.

Steps 4 and 5 come last because the management capability's chain links
through the account DID, which resolves only once the account log is restored.

A restore is idempotent by re-run. Import is a merge that skips what already
exists, so re-running a torn restore converges.

#### A restore rolls state back {#restore-is-a-rollback}

A restore is a rollback of the user-key roster and of the account document to
the moment of the export.

A passphrase changed after the backup was written does not open the restored
account. The passphrase that stood at export time does. A client disconnected
after the backup was written is readmitted by the restore, because the
document that lists it is the document the bundle carries.

This is stated rather than mitigated. A bundle is a snapshot, and restoring a
snapshot restores what it holds.

#### The keystore key is replaced on restore {#keystore-key-replacement}

An account whose document publishes an `authentication` key held in a key
management server does not recover that key from a bundle. A keystore lives
outside the Space tree and has no export operation.

`key-map/keys.json` lands through the import's merge like any other resource,
so a restore strips nothing. A same-host restore may find the keystore still
standing, in which case the named key is still usable.

A wallet MUST mint a fresh key and swap the account document's
`authentication` verification method when the restored `key-map/keys.json`
names a key the keystore does not hold, or when no keystore stands under the
account DID. The swap is one document entry.

The replacement is sound because the key signs only ephemeral presentations. A
verifier re-checking an older proof resolves the `did:webvh` at the version
that proof anchors to.

#### The export-established backup credential {#export-backup-credential}

An export establishes a [=backup credential=] before it reads the account's
Space list, and packs that credential's secret into the bundle. The export
mints the secret, 32 random bytes, and establishes the credential exactly as
registering a passkey establishes one.

The order matters. Establishment writes the credential's own unlock Space, a
`keyAgreement` entry and a ladder verification method in the account
document, a roster wrap, and a registry entry. A list read afterwards
therefore already names that Space. A list read first would leave a bundle
whose credential locates a Space the bundle does not hold.

The credential's rung commitment in the client annex log lands before the
client annex Space is exported. An export that cannot commit it refuses.
Without it, the first login on the restored account would find an uncommitted
rung, start a fresh annex generation, and lose every restored grant.

A restore driven by the bundle is then an ordinary login on that credential.
Nothing is retired, no key rotates, and no new passphrase is set. The
restored client annex stays reachable through the credential's sibling
capability, so live grants survive.

The secret is packed in one of the two forms of [[[#packed-backup-credential]]].
Sealed, the bundle carries nothing openable on its own, and the export
passphrase is what opens it. Plain, the bundle is a bearer credential.

The backup credential is an ordinary unlock method in every other respect. It
is labeled with the export date, and it is listed and removable beside the
account's other unlock methods. It is not retired automatically. Backup
credentials accumulate one per export. Every bundle stays self-sufficient
while its own credential stands.

A holder of the file plus its secret, or plus the passphrase that stood at
export time, opens every row of the account Space archive with no server
contact at all. Removing the credential afterwards does not change that, and
neither does changing the passphrase. Rotation bounds what can be done against
the live account. It does not bound what an old file yields.

#### Origin-bound and portable parts {#bundle-portability}

A bundle restored to the host and Space ids it came from needs nothing
rewritten. Restoring it elsewhere does, because some of what it carries names
its origin.

Origin-bound:

* `id/did.jsonl` and the client annex log, since each DID embeds the host and
  the Space id;
* every governed history log's proofs, which name `did:webvh` verification
  methods;
* the unlock records: the account pointer, the binding, and the two
  capabilities each carries;
* the unlock-methods registry's management capabilities and the
  verification-method ids they name;
* `key-map/keys.json`;
* the archived capability revocations;
* the Space Metadata `controller`.

Portable:

* every encrypted envelope;
* the user key and all of its roster wraps;
* the blinded-index key wraps;
* the unlock Space ids, each a hash of its unlock `did:key`;
* a collection's `generator` attribution (its `id`, `origin`, and `url`);
* all of the content.

What a move must redo is the authority layer alone. The logs are minted
portable, so the account DID keeps its identifier across a host or Space id
change, and the governed logs travel verbatim: a log's controller resolves a
proof's key set at the version the entry anchors to, so a pre-move entry still
verifies. The move ceremony itself is not specified in this version of the
profile.

#### No key material in the clear {#bundle-key-material}

A bundle holds no key material in the clear.

The user key is reachable from the account's passphrase, a recovery code, or
a backup credential's secret, plus `key-map/user-key.jsonl`. The secret goes through the unlock
derivation to an unlock seed, and the seed derives the standing identity whose
key-agreement key the roster's current epoch wraps to. Nothing in the bundle
shortens that path.

In an unlock record, every member other than the record frame, its binding,
and its proof is ciphertext. A bundle's plaintext is the routing information a
holder of the right secret needs to find what to decrypt.

An unprotected bundle is the exception worth naming. It is an exception about
an unlock secret rather than about key material. See
[[[#unprotected-bundle-is-a-bearer-credential]]].

## Security considerations {#security-considerations}

<p class="issue">
  Partially written. The considerations below cover the backup bundle.
</p>

### A management capability outlives its Space {#stale-management-capability}

The management capability the unlock-methods registry holds for an entry
authorizes creating that entry's unlock Space at its original id (see
[[[#unlock-space-archive]]]). The consent check a server makes on that path
verifies the chain and has no revocation scope, so the capability outlives the
deletion of the Space it names.

The residual: a compromised session holding a stale management capability and
an old bundle can re-create a retired credential's unlock Space and import its
archive.

Two things bound it. The capability's own `expires` limits the window. The
re-created Space is useless without the retired credential's secret, which is
what unwraps the record inside: that credential's verification method has been
struck from the account document, its bridge capability is dead, and its
registry entry is gone. What the attacker gains is a Space full of ciphertext
they already had in the bundle.

### A history log only grows at its host {#history-log-continuity}

The account's `did:webvh` log and the client annex log are each stored as a
`did.jsonl` resource. Each is the authorization root of the Spaces its DID
controls. A server resolves the controller from that resource on every
invocation, so whoever can rewrite the resource decides who controls the
Space.

Write access to the resource is broader than control of the account. The
generation delegation a wallet's clients hold covers the whole Space subtree,
and the log sits inside it. If the server treats the log as an ordinary
resource, two attacks follow:

* Rollback. Every prefix of a valid `did:webvh` log is itself a valid log with
  the same SCID. A client whose key was retired in a later entry, and which
  still holds a subtree grant, can put back the prefix that lists its key. Its
  key is then current again, and it can take the Space.
* Deletion. Removing the log, or overwriting it with bytes that do not
  verify, leaves the controller unresolvable. Every invocation fails,
  including the controller's own. The only repair is an update of the Space
  Metadata, and that update would have to be authorized by the controller that
  no longer resolves.

The defense is a write rule on the resource itself, applied by the server. A
`PUT` of a `did.jsonl` resource is accepted only when the stored bytes are a
prefix of the incoming bytes. Otherwise it is refused with 412. A `DELETE` of
one is refused with 405. A governed history log carries the same fast-forward
rule, for the same reason: a write grant can add history but cannot erase it.

The rule applies to a `did.jsonl` resource in every collection, not only in
`id`. A self-hosted DID names the collection its log lives in, and the client
annex log does not live in `id`. A rule keyed on the collection name would
leave the annex log, and any account anchored elsewhere, open to both
attacks. The rule is also unconditional. It does not wait until some Space
names the DID as its controller. Checking for such a reference would mean
scanning every Space on each delete, and the check could race a promotion.
Leaving an unreferenced log deletable gains nothing.

So a log goes away only with its collection or its Space. Deleting the
collection that holds a controller's log leaves every Space that DID controls
with no controller that resolves. There is no break-glass path. A wallet
retires an account by deleting its Spaces, not its log.

The rule does not stop an append whose new entries fail verification. Such a
write keeps the stored bytes as a prefix, and it still leaves the controller
unresolvable. It is caught when the controller is resolved, not refused when
it is written.

A restore is unaffected. It re-creates each Space at its original id and
imports the archive into it (see [[[#restore-order]]]), so the log it lands is
a create, not an overwrite. That is how a restore rolls the account document
back (see [[[#restore-is-a-rollback]]]). It replaces the Space, which takes the
controller's own authority, and does not rewrite a log that a subtree grant
can reach. An import into a Space that still holds its log skips the log, and
the document stays as it stands.

### An unprotected bundle is a bearer credential {#unprotected-bundle-is-a-bearer-credential}

A bundle whose [=backup credential=] is packed plain (see
[[[#packed-backup-credential]]]) hands its holder the account. The secret opens a
standing unlock method against the live server. The holder spends nothing and
retires nothing. They simply log in.

A wallet offering an unprotected export MUST say so at the point of export.
The listing of unlock methods is what makes a standing bundle visible
afterwards, since the backup credential is listed and removable like any other
unlock method.

Sealing the secret removes the bearer property from the file and moves it onto
the export passphrase. A weak export passphrase is a bundle's whole strength,
against an attacker who can guess offline at Argon2id cost.

Neither form is bounded by later rotation. A holder of a file plus the secret
that opens it reads that file's contents offline, whatever the live account
has since done (see [[[#export-backup-credential]]]).
