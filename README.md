# Awesome LNURLcash [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

> Bearer notes on Lightning, built on plain LNURL.

**LNURLcash** makes a bearer instrument out of an ordinary LUD-03
withdrawRequest link, by treating its `k1` as the asset itself:

```
lnurlw://mint.example/w?k1=<secret>&amount=<msat>
```

Whoever knows the `k1` controls the sats behind it, like a banknote. It can be
handed over offline as a QR code, rotated onto a fresh secret, split, merged,
or melted back into a BOLT-11 payment. No new endpoint and no new encoding: a
wallet that has never heard of LNURLcash sees a normal withdraw link and can
still cash it out.

## Contents

- [Specification](#specification)
- [Wallets](#wallets)
- [Mints](#mints)
- [Live services](#live-services)
- [Hardware](#hardware)
- [Libraries](#libraries)
- [Testing and conformance](#testing-and-conformance)
- [Related LUDs](#related-luds)
- [Understanding it](#understanding-it)

## Specification

- [LUD-25 draft](https://github.com/lnurl/luds/pull/301) - The specification,
  under review. Comments belong on the PR.
- [LUD-25 rendered](https://github.com/lnurl/luds/blob/lnurlcash/25.md) - The
  same text on its branch, for reading rather than reviewing.
- [The LUD index](https://github.com/lnurl/luds) - Every LNURL specification.

## Wallets

- [lnurl-wallet](https://github.com/dni/lnurl-wallet) - The reference wallet,
  by dni, and the place to look first when a question is really about what the
  protocol means. A single static page with no backend and no accounts; notes
  live encrypted in the browser.
- [sattle](https://github.com/tompro/sattle) - An installable PWA, by tompro,
  with an Android wrapper. Vue 3 and Quasar. Pins each mint's signing key and
  asks before accepting a change, moves funds between mints, keeps an
  encrypted backup locally or on Nostr, and runs an NWC service so other apps
  can spend from it while it is open.
- [notecase](https://github.com/forgesworn/notecase) - A CLI and a static
  PWA over one engine, in TypeScript. Writes every replacement secret to disk
  before its hash goes on the wire, parks a timed-out mutation as `ambiguous`
  and settles it with `reconcile`, and can pay mint invoices through NWC.

## Mints

- [lnurl-mint](https://github.com/dni/lnurl-mint) - The reference service, by
  dni. Python and FastAPI, backed by lnd or cln. Mint, rotate, split, merge,
  melt, optional fees, optional offline verification, and a sunset switch.
- [moneyer](https://github.com/forgesworn/moneyer) - An independent
  TypeScript implementation, sharing no code with the reference. Backed by
  cln or lnd, stores notes in SQLite by `sha256(k1)` so the database never
  holds a spend secret, and runs the conformance grader in its own test suite.

## Live services

Hosted instances you can point a wallet at today. A mint holds the sats
behind every note it issues, so treat each as an experiment and keep amounts
small.

- [lnurlcash.com](https://lnurlcash.com) - The project site, by dni. A
  one-page explanation of the protocol and where the pieces live.
  [Source](https://github.com/dni/lnurlcash.com).

### Hosted wallets

- [wallet.lnurlcash.com](https://wallet.lnurlcash.com) - The reference wallet,
  hosted. Works against any compliant mint.
- [wallet.moneyer.dev](https://wallet.moneyer.dev) - The notecase PWA,
  hosted. PIN or passkey unlock, and a Service Worker that never caches a
  protocol call.

### Public mints

Each is payable at `mint@<host>` and the bare `_@<host>`. All but moneyer run
lnurl-mint.

- [mint.lnurlcash.com](https://mint.lnurlcash.com) - The reference mint,
  hosted by dni, on the `LNURLmint` node. 17 sat to mint the smallest note,
  9,974 sat cap per note.
- [moneyer.dev](https://moneyer.dev) - The moneyer reference mint. Evaluation
  only, 10k sat cap per note, fee set at the melt routing floor rather than a
  margin. The page is itself a wallet-grade client: mint a note in the
  browser, check one, or melt it, without installing anything. Run to
  demonstrate the protocol and to grade implementations against - not offered
  as a place to keep sats.
- [mint.600.wtf](https://mint.600.wtf) - "600 billion mint", on the `dnilabs`
  node.
- [lnurl.21mint.me](https://lnurl.21mint.me) - On the Azzamo node.
- [lnurl.21linz.at](https://lnurl.21linz.at) - On the `21linz` node, which
  peers over Tor.
- [minty.exe.xyz](https://minty.exe.xyz) - Backed by Core Lightning.
- [mint.forgesworn.dev](https://mint.forgesworn.dev) - **Sunsetting.** Minting
  and splitting have been refused since 2026-08-27. Notes already issued stay
  redeemable until 2026-11-25 - rotate one to keep holding it, merge notes
  together, then melt into a Lightning invoice. Do not point a wallet at it
  for a new note.

## Hardware

- [vault.lnurlcash.com](https://vault.lnurlcash.com) - The vault's web
  installer. Flashes lnurl-vault firmware onto a LilyGo T-Display S3 or the
  classic T-Display from the browser, with no PlatformIO or ESP-IDF install,
  and gives an already-flashed board a device console for the wire protocol
  (`get_info`, `list_notes`, `export_secret`). Needs a Web Serial browser, so
  Chrome or Edge - Safari and Firefox cannot talk to the board.
- [lnurl-vault](https://github.com/dni/lnurl-vault) - An ESP32-S3 hardware
  vault. Generates note secrets from a hardware RNG, discloses only their
  hashes until a mint confirms, and gates every plaintext export behind a
  physical button press. Works fully offline: hold both buttons to unveil a
  note as an on-screen QR code.

## Libraries

- [lnurlcash-kit](https://github.com/TheCryptoDonkey/lnurlcash-kit) - TypeScript. Extracted from the reference wallet's protocol layer.
- [lnurlcash-py](https://github.com/TheCryptoDonkey/lnurlcash-py) - Python,
  sync and async, with a no-I/O protocol layer for other HTTP stacks.
- [lnurlcash-core](https://github.com/TheCryptoDonkey/lnurlcash-core) - Rust,
  with UniFFI bindings for Kotlin and Swift.
- [lnurlcash-kotlin](https://github.com/TheCryptoDonkey/lnurlcash-kotlin) - Kotlin and JVM, over the Rust core.
- [lnurlcash-go](https://github.com/TheCryptoDonkey/lnurlcash-go) - Go.

## Testing and conformance

- [lnurlcash-conformance](https://github.com/TheCryptoDonkey/lnurlcash-conformance) - Language-neutral test vectors, an adversarial mock mint that can be told to
  drop a connection mid-mutation or sign in the wrong byte order, and a grader
  that exits non-zero on a non-compliant service. Run these before you run real
  sats through anything.

## Related LUDs

The specifications LNURLcash is assembled from, rather than replacing.

- [LUD-01](https://github.com/lnurl/luds/blob/luds/01.md) - bech32 encoding.
- [LUD-03](https://github.com/lnurl/luds/blob/luds/03.md) - `withdrawRequest`.
  A note *is* one of these.
- [LUD-06](https://github.com/lnurl/luds/blob/luds/06.md) - `payRequest`.
  Paying one mints a note; its preimage is the secret.
- [LUD-11](https://github.com/lnurl/luds/blob/luds/11.md) - `disposable`.
- [LUD-16](https://github.com/lnurl/luds/blob/luds/16.md) - Lightning
  Addresses, including the bare-domain `_` convention.
- [LUD-17](https://github.com/lnurl/luds/blob/luds/17.md) - The `lnurlw://`
  scheme prefixes.
- [LUD-21](https://github.com/lnurl/luds/blob/luds/21.md) - `verify`. For
  LNURLcash the preimage it returns is bearer material, which changes the
  calculus considerably.

## Understanding it

The parts that are easy to get wrong, and where each is explained.

- [Who generates a replacement secret](https://github.com/lnurl/luds/pull/301/files) - The wallet, never the service. A service-issued replacement has,
  structurally, already been seen by that service.
- [Ambiguous mutations](https://github.com/TheCryptoDonkey/lnurlcash-kit#the-five-things-that-will-cost-you-money) - A rotate that times out may already have burned the input, which makes the
  fresh secret the only copy of the money.
- [HTTP retries](https://github.com/TheCryptoDonkey/lnurlcash-conformance/blob/main/vectors/lifecycle.json) - Every mutation is a GET, and GET is meant to be idempotent. This one is not.
  Named here as a scenario, having caught it in two implementations.
- [Melt semantics](https://github.com/dni/lnurl-mint#readme) - `OK` means the
  payment is in flight, not that the note is spent. See the reserved and
  `pending` states.
- [Persist before disclose](https://github.com/forgesworn/notecase#the-safety-design) - A wallet's ordering rules, written down: the fresh secret hits disk before
  its hash leaves the machine, and an uncertain outcome is a state to
  reconcile, never a guess.
- [Why phoenixd cannot back a mint](https://github.com/forgesworn/moneyer#what-it-is) - Minting needs a funding source that accepts a caller-supplied preimage.
  cln and lnd do; phoenixd and NIP-47 `make_invoice` do not.
- [Offline verification](https://github.com/TheCryptoDonkey/lnurlcash-conformance/blob/main/vectors/signature.json) - The exact signing scheme, including which end of the signature carries the
  recovery id.

## Contributing

Additions welcome, particularly implementations in languages not listed here,
and hosted mints or wallets in the wild. Open a pull request; see
[CONTRIBUTING.md](CONTRIBUTING.md).

Entries need a one-line description saying what the thing *is*, not what it
aspires to be. Working software before announcements.
