# 0002: A content migration copies content and appends to no governed log

- Status: accepted
- Date: 2026-09-17
- Driving work: the design for migrating a backup bundle's content into
  a new wallet account, on any server, from the bundle file and one old
  secret; the profile owns the claim of what a bundle yields to a holder
  of one credential and no server.
- Affects: this spec (the content-migration section, beside the bundle
  text); freewallet and dcw (the walk writes rows through each wallet's
  ordinary stores and touches no log); `@interop/wallet-backup` (the
  walk issues no HTTP request).

## Context

Every governed log in a bundle anchors at the source account's DID: the
account log `id/did.jsonl`, the user-key roster
`key-map/user-key.jsonl`, and each encrypted collection's `meta/log`.
Their entries are signed by keys the source account's document lists,
and a verifier resolves that document, not the target's. A migration
lands in an account with its own DID, its own roster, and its own
collection epochs. Nothing in the envelope or wrap cryptography names
the account DID, so the content decrypts under the old user key and
re-seals under the new one without either log's involvement.

## Decision

A content migration copies rows. It decrypts each row under the source
account's user key, recovered from the archived roster through the old
secret, and writes the plaintext through the target wallet's ordinary
write path, which seals it under the target's current epoch. It appends
to no governed log, on either account: not the source's (the walk
contacts no server) and not the target's (a row write is not a log
entry). The archived logs are read for their heads and, once provenance
verification lands, verified from their bytes; they are never
continued.

## Rejected Alternatives

- Continuing the source account's logs in the target (a "move" of the
  identity). That is a different ceremony with a different subject, the
  account rather than its content, and needs the source server.
- Re-signing the archived collection logs under the target's keys. It
  would fabricate history the target's document never authorised.

## Consequences

- A migrated account has a fresh log history; the old account's history
  survives only as content rows (the credential activities) and in the
  bundle.
- Public copies come back private, since a public link addresses the
  source Space; shares and grants do not carry across, since their
  capabilities chain to the source's document.
- The one new record a migration writes is an activity row in the
  target's `wallet-activity`, naming the bundle's manifest and the
  per-collection counts.

## Revisit Criteria

1. A profile section defines an account move (the identity continued
   under a new Space); the copy stays a separate operation and this
   record then names the boundary between them.
2. A governed log gains an entry kind that records an import by design;
   the "appends to no log" rule would then be narrowed to "no entry the
   target's document did not authorise".
