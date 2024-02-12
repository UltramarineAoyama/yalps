# yalps

Yet Another Linux Packaging System, this time around written in Rust.

## Objective in Project Aoyama (PA)

- we need some sort of rolling release packaging system
- we want very much of the package maintenance process to be automated as we lack maintainers and
  developers who are willing to do exactly that
- just like [Terra], the build process should be transparent and the build sources should be public
- packages should be designed in a way "containerizable", a bit like Flatpaks
  - if possible, we can do it like Nix so we can minimize pain when different packages want
    different dependency versions
- OpenSUSE-like package groups, e.g. `@kde_plasma` = minimal KDE Plasma setup

## Package Manager `yal`

- have _actual_ transactions
  - use btrfs / LVM if available
  - keep snapshots until quota / limit is reached
  - (we'll need to see if we support immutable)
- show warnings, error messages, etc. after the entire transaction (like `brew`)
- show package recommendations (like `pacman`), but allow dependencies to be specified to be
  installed by default (`dnf`)
- if we use a model similar to Nix, we could install packages in parallel

## Metadata

See [rust benchmark comparisons](https://github.com/djkoloski/rust_serialization_benchmark?tab=readme-ov-file).

### Format requirements

- efficiently transfer and decode metadata
- consistent format, not platform/device-dependent
- can partially update (no need to download the entire repo metadata)
- strict schema

### [Cap'n Proto] / [Flatbuffers]

- fast encoding / decoding schemes
- supports easy schema configs
- reasonable size after serialization
  - a bit worse than alkahest
  - usually significantly better if compressed with zlib/zstd
- RPC protocol (capnp)
- supports many proglangs (capnp)
- ❌ not really fault-tolerant
  - maybe only need some checksums?
- ❌ how to partial update?
  - some way to generate a cksum for each pkg and somehow compare from a list
- ❌ not human-readable (but who cares)

### [alkahest]

- pretty much the fastest serializing and deserializing library in rust (other than unsafe `rykv`)
- reasonable size after serialization
  - usually significantly better if compressed with zlib/zstd
- platform-independent
- ❌ a bit difficult to port to other programming languages
  - is this an important factor?
- ❌ not really fault-tolerant
- ❌ how to partial update?
- ❌ not human-readable (but who cares)

### [rkyv]

- pretty fast indeed (the unsafe one is faster than other solutions significantly)
- platform-independent
- built-in byte check
- serde-like macros

## Build reproducibility

We might need to store old packages for reproducibility, but we need to keep package sizes to a
minimum (which means really high compression).

We probably want to store sources too (except repos that stores packages indefinitely like
https://crates.io). However, we definitely want to download the fonts (because remember how Manrope
and Seto got taken down?).

## Minimize package size

- prefer static linking
- split packages into subpackages
- split docs and manpages from the main package
- we might want to reference a license instead of including a copy
  - e.g. an MIT-licensed package should include the dependency `license(MIT)`

## [HeadPAT]

HeadPAT (HEAD of Package/Project Automatically Tracked) is a planned microservice for updating
packages in Terra. The project is currently stalled.

HeadPAT is not a project originally planned with PA in mind but we probably still want to use it.
This is of low priority and we should only work on this later. (Internal docs have more details.)

## Tags

- used for labelling packages
- e.g. `broken:`, `old:`, `legacy:`
- `unofficial:` is used when a package is not QAed (quality assessed)
- `stable:` is used when a package is QAed
- … (tags should be self-explanatory soooo)

# Policies and Guildelines

## Outdated packages

- for reference: https://www.youtube.com/watch?v=zsyX04mn2_Q
- mark package as outdated
  - can add check by comparing with repology or other trusted sources ([HeadPAT]?)
- create GitHub Issue (GHI)
- add tag to pkg sth like `archaic:` or `old:` or `outdated:`

## Broken packages

- mark package as broken
- create GHI
- add tag `broken:`
- warn users not to install package in `yal`

## Categories and groups

- add package to a package group only if it’s 80% of the time needed by users
  - e.g. add `linux` (kernel) to `@base`
  - e.g. add `rust-analyzer` to `@rustdev`
  - e.g. ❌ `firefox` in `@desktop` (there are other browsers and sometimes you don’t need a browser)
- add package to a category if… it’s part of it!
  - can add multiple categories
  - ⇒ easy software browsing
  - e.g. discord in `#Internet` and `#Messaging` etc.

[Terra]: https://github.com/terrapkg/packages
[Cap'n Proto]: https://capnproto.org
[Flatbuffers]: https://flatbuffers.dev
[alkahest]: https://crates.io/crates/alkahest
[rkyv]: https://crates.io/crates/rkyv
[HeadPAT]: https://github.com/terrapkg/HeadPAT
