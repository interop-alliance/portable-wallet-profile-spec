# Portable Wallet Profile for Portable Web Spaces -- Specification

This repo is the **Portable Wallet Profile (PWS-WALLET)** specification, a
W3C CCG-style document authored in **ReSpec + Markdown**. It defines the
conventions under which a portable digital wallet keeps an account on a
Portable Web Spaces server, in two halves:

- **Server requirements**: what a portable wallet needs from any PWS server.
  The PWS and companion-profile versions a wallet depends on and, under
  each, the feature tokens it requires; and this profile's persistent
  identifier, under which a server that satisfies them lists a version
  entry in its service description.
- **Wallet data conventions**: how a portable wallet lays out its data on
  that server. The Spaces an account holds and their roles, the system
  collections and records, and the backup bundle an account is exported,
  restored, and moved with.

The halves are separate sections because a server can satisfy only the
first. The second binds wallet implementations, so that a wallet written in
any language can read, restore, and move an account written by another.

This is a **companion spec layered on top of [PWS]** (the storage model and
HTTP API) and a sibling of **[PWS-EC]** (the encrypted-collection
construction a wallet's collections follow) and **[APP-CONNECT]** (the
handshake by which an app obtains capabilities into a wallet's Space;
referenced, not absorbed).

The profile is implemented by **`@interop/wallet-core`** and
**`@interop/wallet-backup`**, and consumed by the **DCW** and
**freewallet** wallets.

This is a **spec repo, not a code repo**. The deliverable is the rendered
HTML document. There is no build/test/lint pipeline -- "correct" means the
prose is accurate, internally consistent, and renders cleanly in ReSpec.

## Files

- `spec.md` -- the entire normative specification (single file). This is what
  you edit 99% of the time.
- `index.html` -- ReSpec shell: config (`specStatus: "unofficial"`, GitHub
  repo, xref to `did-core`, `localBiblio` for `PWS`, `PWS-AUTHZ`, `PWS-EC`,
  `APP-CONNECT`, `DID-KEY`, `DID-WEBVH`) and the
  `<div data-include="spec.md">` include. The `<h1>`, abstract, SotD, and
  conformance sections live here, not in `spec.md`.
- `llms.txt` -- the llmstxt.org index; keep its summary in step with the
  abstract.
- `README.md` -- short; overview, local preview, and publishing.
- `.github/workflows/publish.yml` -- renders a static ReSpec snapshot and
  deploys it to GitHub Pages on every push to `main`. It greps the snapshot
  for a few top-level section titles; update that list when renaming them.

## Preview locally

```
npm i -g http-server
http-server ./
```

Then open the served `index.html`; ReSpec renders client-side. There is no
static build step for local preview; the published site is rendered by the
workflow.

## ReSpec / Markdown authoring conventions

Match the existing style (same conventions as the encrypted-collections-spec
and app-connect-spec repos):

- **Cross-references to sections:** `[[[#section-id]]]` renders as a live
  link with the section's title. Headings carry explicit ids
  (`### Reads {#reads}`) -- use those.
- **Term references:** `[=term=]` links to a `<dfn>` in the Terminology list
  (e.g. `[=portable wallet=]`, `[=account=]`). Define new terms in the
  `## Terminology` `<dl>` with `<dfn data-lt="aliases">`.
- **Bibliography refs:** `[[PWS]]`, `[[PWS-EC]]`, `[[APP-CONNECT]]`,
  `[[DID-KEY]]`, `[[RFC...]]` -- ReSpec auto-resolves (custom keys via
  `localBiblio` in `index.html`).
- **Headings map to numbered sections.** `##` = top-level, `###`/`####` nest.
- Cross-document links into PWS or a sibling profile are absolute URLs into
  the other document, since `[[[#id]]]` only resolves within one ReSpec
  document.
- Unwritten sections carry a `<p class="issue">` placeholder rather than
  empty prose.
- Honor the global rules: use `--` not an em dash, and `to` not an arrow.

## Design invariants (so edits stay consistent)

- **Two halves, two conformance targets.** Server-binding text lives only
  under Server requirements; everything under Wallet data conventions binds
  wallets. Do not let a wallet-shaped requirement leak into the server half,
  and do not let PWS-EC or PWS core absorb wallet-shaped requirements that
  belong here.
- **Byte-level values are permanent.** Space `type` arrays, system
  collection and resource names, record shapes, bundle manifest layout, and
  this profile's persistent identifier `https://w3id.org/pws/wallet-profile`
  (the `specs` key a server lists it under) are baked into stored artifacts
  and served documents. Transcribe them from the implementing code and verify
  against it; never re-derive or rename without the user's sign-off.
- **Altitude rule:** state invariants, not current implementations.
- **Fail-closed extensibility** throughout: anything unrecognized is
  rejected, not ignored.
- Wallet terminology: follow the global `clientId` / `writerId` rules --
  never "device" for either concept.

## Parties to this contract

Every repo that implements or consumes this profile, with the specific
modules that speak it. **The maintenance rule: a normative change's checklist
is a walk of this table** -- for each row, resolve the impact as shipped
(naming what landed, including the row's ARCHITECTURE/AGENTS docs) or
explicitly waived (`unaffected: <repo> (<why>)`).

| Repo | Modules speaking the contract |
| --- | --- |
| wallet-core | The account's Space layout and roles (`src/space/`), the user-key roster and unlock records (`src/keyring/`, `src/enrollment/`, `src/recovery/`), and the permanent-constants rows in its ARCHITECTURE.md. |
| wallet-backup | The backup bundle codec and the export, restore, and move ceremonies. |
| was-client | The per-Space export and import bindings the bundle wraps; a connect-time check for this profile's service-description entry. |
| freewallet | The web wallet: account creation, unlock methods, backup and restore UI. |
| dcw (private) | The mobile wallet, over the same wallet-core modules. |
| was-teaching-server | Lists this profile's identifier under `specs` once it satisfies the server requirements. |
| was-conformance-suite | A suite verifying that a server listing this profile's entry advertises every token the server requirements name. |

## Ecosystem conventions

- Cross-repo lessons (invariants, gotchas, and process recipes that span
  repos) live in the ecosystem learnings file,
  [byoe-ecosystem/LEARNINGS.md](https://github.com/interop-alliance/byoe-ecosystem/blob/main/LEARNINGS.md)
  (usually checked out beside this repo as `../byoe-ecosystem`); read it at
  the start of any cross-repo task.
- Decisions about the contract this spec owns (profile and wire-contract
  decisions) are recorded in this repo's [decisions/](decisions/) directory,
  one `NNNN-slug.md` file per decision; the convention and template are
  canonical in
  [isomorphic-lib-template's `decisions/`](https://github.com/interop-alliance/isomorphic-lib-template/tree/main/decisions).
- Open work items for this spec live in [_spec/ROADMAP.md](_spec/ROADMAP.md)
  as `PWP-N` items; shipped items move to the archive beside it so ids
  keep resolving. An item is filed in the roadmap of the repo whose document
  it changes: work on PWS core, its authorization profile, Encrypted
  Collections, or App Connect goes in those repos' roadmaps, not here. An
  item moved between roadmaps keeps its old id in the source archive with a
  pointer to the new one. The item format and the `touches:` rule are
  canonical in
  [isomorphic-lib-template's AGENTS.md](https://github.com/interop-alliance/isomorphic-lib-template/blob/main/AGENTS.md)
  ("Roadmap & Task Conventions").

## Reference material (read-only, outside this repo)

These are separate repositories. Ground spec prose against real behavior --
check with the user before editing anything in them.

- [wallet-attached-storage-spec](https://github.com/w3c-ccg/wallet-attached-storage-spec)
  -- the PWS spec this profile layers on; its `spec.md` is the source of
  truth.
- [encrypted-collections-spec](https://github.com/interop-alliance/encrypted-collections-spec)
  -- the sibling profile a wallet's encrypted collections follow.
- [app-connect-spec](https://github.com/interop-alliance/app-connect-spec)
  -- the sibling handshake profile.
- [wallet-core](https://github.com/interop-alliance/wallet-core),
  [wallet-backup](https://github.com/interop-alliance/wallet-backup),
  [freewallet](https://github.com/interop-alliance/freewallet), dcw
  (private repo) -- the wallet layers implementing and consuming this
  profile.
