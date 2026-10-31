# Rusty Wormhole

Get things from one computer to another, safely.

<http://magic-wormhole.io/>

This is a Rust port of the Python version at <https://github.com/magic-wormhole/magic-wormhole>.

## Comparison with the Python implementation

Features that are missing:

- Text message sending and receiving
- Native folder transfer (sent folders become `.tar` files; received Python folders become `.zip` files, both requiring manual extraction)
- Tor support
- Automatic mailbox reconnection and Python's experimental Dilation API

Additional features compared with Python's `wormhole` CLI:

- STUN-assisted direct connections across some NATs and firewalls (not guaranteed for every network)
- Automatically copies your code to the clipboard
- Port forwarding in addition to file transfer (experimental)
- Send a file to multiple people (experimental)
- Fuzzy wormhole code completion

See [PYTHON_PARITY.md](PYTHON_PARITY.md) for the pinned comparison, exercised interoperability scenarios, and suggested parity work.

## Getting started

If you want to toy with the CLI, `cargo run -- --help` will get you started. The code sits in `./cli/src`. For more instructions see [cli/README.md](cli/README.md).

If you'd like to use Wormhole in your application, see the [library documentation](https://docs.rs/magic-wormhole/) or run `cargo doc -p magic-wormhole --open`.

If you don't fear touching code and want to contribute, `./src/lib.rs`, `./src/transfer.rs` and `./src/transit.rs` are rather easy to get into. The [protocol specification](https://github.com/magic-wormhole/magic-wormhole-protocols) will probably be useful to you.

### WebSocket TLS

Native builds of the library and CLI enable `tls` by default, using `futures-rustls`
with bundled WebPKI root certificates for `wss://` mailbox connections. The CLI's
`all` feature uses the same backend. Set `default-features = false` for the library
or pass `--no-default-features` when building the CLI to opt out.

For system trust stores, select `futures-rustls-native-certs` or
`futures-rustls-platform-verifier` instead. `native-tls` remains available as an
explicit alternative. These features work for both the library and CLI.

## Applications using Wormhole Rust as library

- [Warp](https://gitlab.gnome.org/World/warp), a GUI client using Gtk
- [Wormhole File Transfer](https://github.com/wormhole-app/wormhole), a Android client using Flutter
- [Wyrmhole](https://github.com/ClaytonWas/wyrmhole), a cross-platform GUI using Tauri
- [rvIRC](https://github.com/KaraZajac/rvIRC), a vim-based IRC client with filesharing

(feel free to add yours)

## License

This work is licensed under the EUPL v1.2 or later. Contact the owner(s) for use in proprietary software.

----------

[![Matrix][matrix-room-image]][matrix-room-url]
[![Irc][irc-room-image]][irc-room-url]
![Build Status][build-status-image]
[![Deps][deps-status-image]][deps-status-url]
[![Codecov][codecov-image]][codecov-url]
[![Is-It-Maintained-Resolution-Time][iim-resolution-image]][iim-resolution-url]
[![Crates.io][crates-io-image]][crates-io-url]
[![Docs.rs][docs-image]][docs-url]

[matrix-room-image]: https://img.shields.io/badge/matrix.org-%23magic--wormhole-brightgreen
[matrix-room-url]: https://matrix.to/#/#magic-wormhole:matrix.org
[irc-room-image]: https://img.shields.io/badge/irc.libera.chat-%23magic--wormhole-brightgreen
[irc-room-url]: https://web.libera.chat/
[build-status-image]: https://github.com/magic-wormhole/magic-wormhole.rs/workflows/Rust/badge.svg
[deps-status-image]: https://deps.rs/repo/github/magic-wormhole/magic-wormhole.rs/status.svg
[deps-status-url]: https://deps.rs/repo/github/magic-wormhole/magic-wormhole.rs
[codecov-image]: https://codecov.io/gh/magic-wormhole/magic-wormhole.rs/branch/main/graph/badge.svg
[codecov-url]: https://codecov.io/gh/magic-wormhole/magic-wormhole.rs
[crates-io-image]: https://img.shields.io/crates/v/magic-wormhole.svg
[crates-io-url]: https://crates.io/crates/magic-wormhole
[docs-image]: https://docs.rs/magic-wormhole/badge.svg
[docs-url]: https://docs.rs/magic-wormhole
[iim-resolution-image]: http://isitmaintained.com/badge/resolution/magic-wormhole/magic-wormhole.rs.svg
[iim-resolution-url]: http://isitmaintained.com/project/magic-wormhole/magic-wormhole.rs
