# Magic Wormhole CLI

A Rust implementation of the classic `magic-wormhole` command line client.

This package provides a library and a command-line tool named `wormhole-rs`, which transfers text messages, arbitrary-sized files, and directories between computers. Directories are transferred as archives requiring manual extraction. The two endpoints use identical "wormhole codes": normally the sender generates and displays a code, which the receiver then enters.

## Installation

### GitHub Releases

We publish source code and binaries to GitHub releases. Visit the [releases page](https://github.com/magic-wormhole/magic-wormhole.rs/releases) for the latest release.

Or use [cargo binstall](https://github.com/cargo-bins/cargo-binstall):

```bash
cargo binstall magic-wormhole-cli
```

### crates.io

You can use cargo to install the CLI from crates.io:

```bash
cargo install --locked magic-wormhole-cli
```

## Usage

```bash
# Send files or a folder.
wormhole-rs send <PATH>

# Send a text message, including an empty message with --text "".
wormhole-rs send --text "Hello from Rust"

# Read the complete UTF-8 message from stdin without trimming whitespace.
printf '%s' 'Hello from stdin' | wormhole-rs send --text -

# Receive either text or a file using the sender's code.
wormhole-rs receive <CODE>
```

`--text` cannot be combined with file paths or `--rename`. `send-many` remains file-only.
Text travels over the encrypted mailbox without a transit connection. Receiving text
prints it to stdout with a trailing newline and creates no files. Terminal output
escapes control characters other than newline and tab; piped output preserves the
original text. Diagnostics go to stderr. The sender receives a success acknowledgement
only after the receiver writes and flushes the text.

Run `wormhole-rs --help` or a subcommand with `--help` for additional options.
