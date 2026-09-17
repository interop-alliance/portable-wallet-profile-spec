# 0001: One profile document until a second consumer

- Status: accepted
- Date: 2026-09-16
- Driving work: drafting the server requirements section of this profile.
- Affects: portable-wallet-profile-spec (spec.md, Server requirements
  section); no other repo is bound by this decision.

## Context

Drafting the Server requirements section surfaced a bundle of server
affordances a portable wallet needs: listing, collection and Space
management, policy, export, encryption, the changes feed, blinded-index
query, and governed history logs. That bundle is not wallet-specific. A
local-first synced application that is not a wallet would need the same
set: the same replication, conditional-write, and encryption affordances,
without this profile's wallet data conventions.

The question was where that bundle lives: as its own profile document (a
`local-first-synced-app` profile) that this profile references, or as this
document's Server requirements section, with no separate document.

## Decision

One document. This profile's Server requirements section is the only
specification of the bundle. No separate local-first profile exists, and
this profile references none.

## Rejected Alternatives

- A separate local-first profile now. It would have no consumer today.
  A second identifier and a second version entry with no server or client
  implementing it is a maintenance cost with no interoperability gain.
- Growing the PWS-EC (Encrypted Collections) conformance section into the bundle. Decided
  earlier and recorded in this spec's introduction: PWS-EC would then
  carry wallet-shaped requirements a non-wallet encrypted app does not
  want.
- Advertising the bundle as a single feature token. Also decided earlier:
  a token cannot carry a version or a definition URL, and the bundle
  needs both.

## Consequences

A non-wallet application that wants the same server bundle checks this
profile's identifier, even though it never follows the wallet data
conventions. That is acceptable: listing the identifier claims only the
server requirements section, not the wallet data conventions.

## Revisit Criteria

1. A second consumer that is not a wallet ships against the same server
   bundle and needs to name it.
2. The wallet's server bundle and a local-first application's bundle
   diverge: one needs tokens the other does not.

If revisited: split the bundle out as a new sibling profile with its own
identifier, and have this profile's Server requirements section reference
it. This profile's identifier and version entry stay unchanged.
