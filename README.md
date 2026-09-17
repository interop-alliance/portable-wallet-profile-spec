# Portable Wallet Profile for Portable Web Spaces

> The Portable Wallet Profile (PWS-WALLET) is a companion profile to the
> [Portable Web Spaces spec](https://w3c-ccg.github.io/wallet-attached-storage-spec/).
> It defines what a portable digital wallet needs from a PWS server (the
> server requirements a wallet checks before it trusts a server with an
> account), and how a portable wallet lays out its data on that server (the
> Space roles, system collections, record shapes, and the backup bundle a
> wallet can be moved with), so that a wallet written in any language can
> read, restore, and move an account written by another.

This repository contains the Portable Wallet Profile specification, in
[ReSpec Markdown](https://respec.org/docs/#markdown) format. Its persistent
identifier is `https://w3id.org/pws/wallet-profile`. It is a sibling
of the [Encrypted Collections](https://interop-alliance.github.io/encrypted-collections-spec/)
and [App Connect](https://interop-alliance.github.io/app-connect-spec/)
profiles.

For LLM consumption, [`llms.txt`](llms.txt) summarizes the spec and links to
its full text, following the [llmstxt.org](https://llmstxt.org/) convention.

## Table of Contents

- [Background](#background)
- [Code of Conduct](#code-of-conduct)
- [Contributing](#contributing)
- [Status](#status)
- [Usage](#usage)
  - [Editing](#editing)
  - [Testing](#testing)
  - [Publishing](#publishing)

## Background

You can access the latest version of this specification at:

https://interop-alliance.github.io/portable-wallet-profile-spec/

## Code of Conduct

This repository functions under the W3C
[code of conduct](https://www.w3.org/Consortium/cepc/).

## Contributing

Contributions are welcome as issues and pull requests in the GitHub
repository.

## Status

This document is an experimental draft specification, undergoing regular
revisions. Feedback is welcome via the
[issue tracker](https://github.com/interop-alliance/portable-wallet-profile-spec/issues).

## Usage

### Editing

The specification source is in [`spec.md`](./spec.md), in
[ReSpec Markdown](https://respec.org/docs/#markdown) format. The `<h1>`,
abstract, status, and conformance sections live in the ReSpec shell,
[`index.html`](./index.html).

### Testing

For testing locally, you can `npm i -g http-server`, and then:

```
http-server ./
```

Then open the served `index.html`; ReSpec renders client-side.

### Publishing

The published site is built by [`.github/workflows/publish.yml`](.github/workflows/publish.yml)
on every push to `main`, and deployed to GitHub Pages. The workflow runs the
[ReSpec CLI](https://respec.org/docs/#respec-cli) over `index.html` and
publishes:

| Path        | Contents                                                                                                                           |
|-------------|------------------------------------------------------------------------------------------------------------------------------------|
| `/`         | A static, pre-rendered snapshot of the spec. Readable without JavaScript, so search engines, `curl`, and agents see the full text. |
| `/live/`    | The original ReSpec page, rendered client-side from `spec.md` at view time.                                                        |
| `/spec.md`  | The Markdown source.                                                                                                               |
| `/llms.txt` | The [llmstxt.org](https://llmstxt.org/) index.                                                                                     |

Pull requests build the snapshot as a check but do not deploy it. To render a
snapshot yourself:

```
npx respec@latest --src index.html --out snapshot.html --localhost
```

The repository's Pages source must be set to **GitHub Actions** (rather than
"Deploy from a branch") for the workflow to publish.
