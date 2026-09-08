# Maintenance documentation

## Dependencies

Prefer standard-library APIs available in the workspace MSRV (currently Rust 1.92).
Evaluate replacements against the resolved dependency graph, supported platforms,
and protocol compatibility, as well as the direct dependency count.

- Library errors remain typed `std::error::Error` implementations, using
  `thiserror` for derives. Do not add `anyhow` or `eyre` to the library, its tests,
  or its examples. Tests and examples can use `Box<dyn std::error::Error>` when
  they need to propagate several error types.
- `anyhow` is confined to the CLI for application-level context and reporting.
- The default TLS backend is rustls with bundled WebPKI roots. Platform trust
  stores and `native-tls` remain explicit feature choices. This configures TLS
  for `wss` connections; it does not change the rendezvous server URL.

### Review: September 2026

This review and its implementation were prepared by an AI agent.

Applied reductions:

| Previous dependency or helper | Replacement |
| --- | --- |
| `eyre` in library tests and examples | Standard boxed errors; public library error enums stay typed |
| CLI `color-eyre` and its color helpers | CLI-only `anyhow` and the existing `console` dependency |
| Unused CLI `env_logger` | Removed; CLI already uses `tracing-subscriber` |
| `test-log` default logging features | Tracing and color features only |
| `futures-concurrency`, used by two library tests | `futures::future::join` from the existing dependency |
| Separate `serde_derive` declarations | Serde's `derive` feature and reexports |
| `futures::pin_mut!` | [`std::pin::pin!`](https://doc.rust-lang.org/std/pin/macro.pin.html) |
| Reexported `ready`, `pending`, and `Future` | [`std::future`](https://doc.rust-lang.org/std/future/index.html), including the Rust 2024 `Future` prelude import |
| Nested results and error logging through `map_err` | [`Result::flatten` and `Result::inspect_err`](https://doc.rust-lang.org/std/result/enum.Result.html) |
| Cloning keys from an owned map | [`HashMap::into_keys`](https://doc.rust-lang.org/std/collections/struct.HashMap.html#method.into_keys) |
| Handwritten byte-array counter for hashcash | `u64::wrapping_add` and `u64::to_be_bytes`; preserve the wire encoding |
| Random salt collected through a temporary vector | `rand::random::<[u8; 16]>()` |

This simplification reduces `Cargo.lock` from 462 to 441 package entries.
Removing a direct dependency does not necessarily remove its transitive uses:
Serde still needs `serde_derive`, and optional `test-log` dependencies can remain
in the lockfile even when they are absent from the active build graph.

Retained dependencies:

| Dependency | Reason |
| --- | --- |
| `time` | Hashcash needs a UTC calendar date, which `std::time` does not format. `zxcvbn` already depends on `time`, so switching this one use to Jiff would add a second calendar library. There is no Chrono dependency. Revisit Jiff if timezone or calendar arithmetic requirements grow. |
| `async-trait` | Transit calls asynchronous methods through trait objects. Native async trait methods are not [dyn-compatible](https://doc.rust-lang.org/reference/items/traits.html#dyn-compatibility); removing the macro would require handwritten boxed futures. |
| `futures` and `futures-lite` | Async I/O, streams, selection, and combinators still require these crates. The standard pinning and basic future helpers cover only part of their functionality. |
| `libc` and `socket2` | Nonblocking connection handling and socket options need platform APIs. [`ErrorKind::InProgress`](https://doc.rust-lang.org/std/io/enum.ErrorKind.html#variant.InProgress) is unstable on Rust 1.92. |
| `thiserror` and `derive_more` | Generate trait implementations; removing them would add handwritten boilerplate rather than use a standard derive. |
| `rand` 0.8 | `crypto_secretbox` 0.1 and `spake2` 0.4 require its `rand_core` 0.6 generation. Coordinate a future upgrade with those crypto crates. |
| Cryptography, encodings, archives, URL parsing, and async runtime crates | No equivalent standard-library facilities cover these uses. |

The code already uses `OnceLock`, `LazyLock`, `IsTerminal`, and `io::Error::other`.
Transitive copies of older helper crates must be addressed by their upstream
dependants rather than replaced in this repository.

Run `cargo deny --workspace --locked check` to audit both the library and CLI.
CI includes the whole workspace. `deny.toml` records version-scoped exceptions
for existing clipboard dependencies: `clipboard-win` and `error-code` use
[BSL-1.0](https://spdx.org/licenses/BSL-1.0.html), and `foldhash` 0.1 uses
[Zlib](https://spdx.org/licenses/Zlib.html). Revisit these entries when updating
the corresponding packages.

## Release

To create a new release, follow these steps:

- Update version number in Cargo.toml for library and CLI
- Update CHANGELOG.md with release date
- Update Cargo.lock
- Commit & push the changes
- Tag the commit: `git tag -as a.b.c`
- Push the tag: `git push origin a.b.c`
- Verify GitHub release was created by CI
- Push a new crate version to crates.io with `cargo publish --workspace`
