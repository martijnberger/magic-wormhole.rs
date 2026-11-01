# Python implementation parity review

Reviewed on 2026-10-31. This is a capability and architecture comparison, not a security audit. Recheck the pinned baselines before using it to plan an upstream update.

## Baselines

| Repository | Branch / revision | Result |
| --- | --- | --- |
| Canonical Rust upstream | `main`, `41d78aed662d29729c98ac5715d7e413c139e18e` | No missing upstream commits at review time |
| This fork before receiver hardening | `926c4171a2564ddfff825492028b24473b37b6bc` | Four commits ahead, zero behind canonical Rust upstream |
| Canonical Python implementation | `master`, `621ac5310006ee4a4c6d749ba19c258ba5325de2` | Comparison baseline; NEWS lists release 0.24.0 and subsequent unreleased changes |

Rust upstream uses `main`, not `master`. The fork-only commits modernize dependencies, TLS defaults, CI, malformed-message handling, and CLI diagnostics; they do not add protocol features. The reviewed Rust package is version 0.8.1, with Rust 1.92 minimum.

- [Rust upstream-to-fork comparison](https://github.com/martijnberger/magic-wormhole.rs/compare/41d78aed662d29729c98ac5715d7e413c139e18e...926c4171a2564ddfff825492028b24473b37b6bc)
- [Pinned Python release notes](https://github.com/magic-wormhole/magic-wormhole/blob/621ac5310006ee4a4c6d749ba19c258ba5325de2/NEWS.md)

## Capability comparison

| Area | Rust checkout | Python baseline |
| --- | --- | --- |
| Classic protocol | SPAKE2, HKDF, Secretbox, classic transit and file-transfer acknowledgements | Compatible classic protocol |
| Single files | Supported; live interoperability verified in both directions | Supported |
| Text | Unsupported at review; implemented in the follow-up below | Supports text and reading text from stdin |
| Sending directories | Ordinary file offer containing a `.tar` archive | Compressed directory offer |
| Receiving directories | Saves Python directory offers as `.zip`; no automatic extraction | Extracts received directories |
| Direct connectivity | STUN-assisted TCP NAT traversal, direct/relay selection | Direct connections and relay fallback; no equivalent STUN path identified |
| Tor | No native integration | Integrated Tor support |
| Mailbox reconnection | No equivalent automatic recovery loop | Automatic reconnection through Twisted ClientService |
| Dilation | No usable implementation in this checkout | Optional experimental reconnecting, multiplexed subchannels |
| Port forwarding | Built-in CLI, own application ID and MessagePack protocol over classic transit | Separate Fowl application uses Dilation; not a base CLI mode or a compatible forwarding protocol |
| Other CLI features | Clipboard integration, fuzzy code completion, send-many | Verification prompt, receiver-allocated codes, SSH-key exchange |
| QR codes | Supported | Supported |
| Runtime / embedding | Async Rust, smol-oriented native implementation | Python / Twisted |
| License | EUPL | MIT |

STUN improves connection opportunities; it does not guarantee traversal through every NAT or firewall. Rust's relay-only option is not a substitute for Tor anonymity.

Rust's experimental `transfer-v2` is a separate protocol, not Python Dilation. It is disabled by default and the stock version configuration does not advertise it even when compiled. Do not count it as production cross-language resumable transfer support. Python's own ecosystem documentation rates Dilation as experimental and Dilated File Transfer as proof-of-concept; the ordinary Python CLI is not evidence of production resumable transfers either.

Enabling Rust TLS support does not change the default mailbox URL from `ws://` to `wss://`. Mailbox TLS and Wormhole end-to-end encryption are separate layers.

CLI configuration is not interchangeable: `WORMHOLE_RELAY_URL` selects the transit relay in Rust but the mailbox/rendezvous server in Python. Rust uses `WORMHOLE_MAILBOX_URL` for the latter.

### Source map

- Rust classic transfer and archive handling: `src/transfer/v1.rs`.
- Rust capability advertisement and v1/v2 selection: `src/transfer.rs`.
- Rust direct connections and STUN: `src/transit.rs`, `src/transit/transport.rs`.
- Rust mailbox lifecycle: `src/core/rendezvous.rs`.
- Rust forwarding application/protocol: `src/forwarding.rs`.
- Rust CLI options: `cli/src/main.rs`.
- [Pinned Python CLI](https://github.com/magic-wormhole/magic-wormhole/blob/621ac5310006ee4a4c6d749ba19c258ba5325de2/src/wormhole/cli/cli.py).
- [Pinned Python directory receive/extraction](https://github.com/magic-wormhole/magic-wormhole/blob/621ac5310006ee4a4c6d749ba19c258ba5325de2/src/wormhole/cli/cmd_receive.py).
- [Pinned Python mailbox reconnection](https://github.com/magic-wormhole/magic-wormhole/blob/621ac5310006ee4a4c6d749ba19c258ba5325de2/src/wormhole/_rendezvous.py).
- [Pinned ecosystem comparison and forwarding applications](https://github.com/magic-wormhole/magic-wormhole/blob/621ac5310006ee4a4c6d749ba19c258ba5325de2/docs/ecosystem.rst).

## Exercised behavior

Built and ran the Rust CLI, installed the pinned Python source in an isolated environment, and used an isolated local mailbox server with direct transit connections.

- Rust to Python and Python to Rust single-file transfers completed with identical bytes.
- Rust directory to Python produced a `.tar`; archived contents matched.
- Python directory to Rust produced a `.zip`; archived contents matched.
- Python text to Rust failed with `Expected 'transit', but got: 'offer'`, confirming unsupported text reception.
- Tor, WAN NAT traversal, reconnection, and Dilation were compared from source/documentation, not exercised.

The temporary mailbox, transfer outputs, Python installation, and comparison clone were removed after the review.

## Suggested parity work

1. **Text transfer — implemented:** see the follow-up below for APIs, CLI behavior, and verification.
2. **Native directories:** exchange Python-compatible directory offers and ZIP content, and extract safely. Prove both directions, empty/nested directories, filenames, and permissions where supported.
3. **CLI workflows:** decide which Python workflows matter: verification prompt, receiver allocation, SSH-key exchange, and migration guidance for option/environment differences. These are product choices, not prerequisites for ordinary file interoperability.
4. **Reconnection and Dilation:** treat as a separate architecture project. Define lifecycle, subchannel negotiation, ordering, recovery, and interoperability requirements before implementing; do not relabel Rust transfer-v2 as Dilation.
5. **Tor:** add only with explicit privacy requirements and end-to-end routing checks; relay-only operation does not provide equivalent anonymity.

These are planning notes, not commitments to implement every Python option or unsafe convenience mode. Preserve Rust-specific advantages where useful.

## Receiver traversal finding

A separate concrete issue surfaced during the comparison: a peer-supplied filename such as `../escaped.bin` could escape the default CLI receiver's `--out-dir`. This was reproduced with a live Rust-to-Rust transfer. The CLI was then changed to reject path-bearing names before creating a file and to notify the sender of rejection. A repeat live transfer rejected the malicious name, and ordinary Python-to-Rust transfer still produced identical bytes. CLI regression tests passed.

The follow-up hardening moves classic-offer filename validation into the library, protecting CLI and library consumers before a receive request is returned. Filesystem acceptance helpers validate every name in the complete offer tree before preparing writers or creating directories, including names of empty directories. `Offer::accept_all` now returns `std::io::Result<OfferAccept>`; callers must handle this result.

Verification includes a failing-before/passing-after nested-directory traversal regression, transfer tests with default and experimental features, CLI tests with experimental features, and live rejection of relative, absolute, and Windows-style paths. A normal Unicode/spaced filename still transfers with identical bytes. Windows-specific drive/stream/trailing-dot validation has conditional regression coverage but was not executed on Windows during this Linux review.

Name validation prevents peer-supplied path traversal; it does not protect library helpers against pre-existing or concurrently replaced symlinks in a locally untrusted destination. These helpers require a trusted destination directory. See the Unreleased Security section in `CHANGELOG.md` for shipped behavior. This finding does not change the feature-parity conclusions above and is not a claim that the implementation has undergone a full security audit.

## Text-transfer implementation follow-up

Text transfers now use the Python-compatible `offer.message` / `answer.message_ack`
exchange over the encrypted mailbox, without initializing transit. The sender requires
the exact acknowledgement `"ok"` and tolerates unsolicited classic transit hints.
`transfer::send_text` sends text; `transfer::request_offer` distinguishes text and file
offers. `ReceiveTextRequest` exposes the message and explicit accept/reject operations.
Existing file-only receive APIs retain their behavior and reject text.

The CLI supports `send --text MESSAGE`, including an empty string, and `send --text -`
for complete UTF-8 stdin without trimming. `receive` detects text automatically and
creates no output files or directories. It escapes terminal controls, preserves text
when piped, appends a newline, and acknowledges only after writing and flushing stdout.
Text cannot be combined with file paths or `--rename`; `send-many` remains file-only.

Live checks against the same pinned Python revision covered both directions,
Unicode/multiline text, empty messages and stdin, unsolicited transit hints, incorrect
acknowledgements, peer rejection, public-API cancellation, output failure, and a real
terminal receiving control characters. Ordinary bidirectional file transfers retained
identical bytes and Unicode/spaced filenames. Python's CLI prompts for input when
given an empty `--text ""`, so Python-to-Rust empty-message coverage used Python's API.

The next parity target remains **native directory transfers**: Python-compatible
directory offers and ZIP content in both directions, with safe extraction rather
than automatic extraction of untrusted paths without validation.
