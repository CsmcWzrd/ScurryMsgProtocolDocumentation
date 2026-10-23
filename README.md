# ScurryMsg 1.10.0 — SCURRY 1.2

SCURRY **1.2 is implemented** for peer and central clients, ScurryServer and all four language codecs. Protocol 1.0 is deprecated and rejected. The owner's 2026-10-16 implementation instruction supersedes the earlier proposed-only 1.2 status.

- Peers return verified SHA-512 receipts on the completed-transfer connection, including resume and durable replay.
- Central clients submit receiver receipts; senders fetch `/delivery-success`; the server deletes fetched acknowledgements.
- Cross-server delivery uses four raw 256-byte fields, scoped credentials, header-only notices, chunked recovery and receiver-authenticated storage.
- Central composers offer optional recipient-server and full-name fields. Fetched receipts appear in Acknowledgements.
- Every first header is `Protocol: SCURRY 1.2 UTF-8 UTF-8 /endpoint`. Credentials use `Sender's-Login`/`Sender's-Password`, except final import uses receiver-named credentials.

[Design](docs/SCURRY-1.2-DESIGN.md) · [RFC](docs/SCURRY-1.2-RFC.txt) · [readable message examples](docs/SCURRY-1.2-MESSAGE-EXAMPLES.md) · [validation](docs/VALIDATION-1.10.md).

## Packaged protocol documents

The source archive includes the complete documentation under `docs/`:

- [Illustrated design (PDF)](docs/Scurry-Protocol-Specification-1.2-Corrected.pdf) and [editable design](docs/SCURRY-1.2-DESIGN.md).
- [Message exchange reference (PDF)](docs/SCURRY-1.2-EXCHANGE-REFERENCE.pdf) and [editable reference](docs/SCURRY-1.2-EXCHANGE-REFERENCE.md): header types/values first, then peer, single-server, multiple-server and contact-book exchanges.
- [Endpoint examples](docs/SCURRY-1.2-MESSAGE-EXAMPLES.md), [current RFC](docs/SCURRY-1.2-RFC.txt) and [validation](docs/VALIDATION-1.10.md).

The exchange reference shows ASCII projections of SCURRY fixed-slot frames; directory requests retain their actual UDP JSON format. See [documentation index](docs/README.md).

## Run or build

Python peer GUI: `python applications/ScurryMsg-Python/ScurryMsg.py`.
Python central GUI: `python applications/ScurryMsgCentral-Python/ScurryMsgCentral.py`.
Install requirements from `requirements.txt`; Tcl/Tk is required. Python 3.15 is a target acceptance check; the included validation report states the runtime actually tested.

Windows C# clients/key tools: open `ScurryMsg.sln` in VS2022 17.8+ with .NET desktop development and .NET 8. Select the desired startup project. `build-windows.cmd` builds and runs the standard .NET tests. Windows server: use the separate `ScurryServer/ScurryServer.sln` and its documented vcpkg/CMake dependencies.

Linux native server:

```sh
cmake -S ScurryServer -B build/server -DSCURRY_BUILD_TESTS=ON
cmake --build build/server --parallel
ctest --test-dir build/server --output-on-failure
```

[Server setup](docs/SCURRYSERVER-SETUP.md), [database adapters](docs/SCURRYSERVER-DATABASES.md), [build guide](BUILDING.md). SQLite catalog metadata is server-owned; payload storage can use SQLite, ODBC SQL, CouchDB or configured Cassandra. Central clients use encrypted flat files, not databases. Empty server certificate/key paths create an ephemeral RSA-3072 test identity; deploy managed TLS identities and trust before remote use.

Peer defaults remain TCP 33337 for messaging and UDP 33336 for ScurryLink discovery. Exchange certificate pins and pair credentials through a trusted channel. ScurryLink discovery does not provide NAT traversal or configure firewalls. Central requests require canonical login/password headers even when a client certificate is also configured.

For cross-server mail, enter the destination server's hostname/port and recipient's exact full name in the central composer. The sender's configured server hostname must also be reachable by the receiver. Both servers must be trusted by the client CA policy. Clear the optional server field for same-server delivery.

## Upgrade

Upgrade all endpoints together; there is no downgrade to 1.0. Back up the server catalog/payload store and existing client profiles. Keep vault keys and account-password formats. Previously queued or stored 1.0 frames are not silently rewritten: review and recompose queued messages, and perform a deliberate offline migration for old stored frames before enabling 1.2 access. The design documents the destructive-fetch crash window for acknowledgements.

The separate 1.1 shared blogs/chat/calendar-resource proposal remains pending. Existing local workspace features, key generation, cards, encrypted storage, calendar views and consent settings remain included.

## Test

```sh
./test.sh
SCURRY_SERVER_BINARY="$PWD/build/server/ScurryServer" python tests/test_protocol12.py
SCURRY_SERVER_BINARY="$PWD/build/server/ScurryServer" python ScurryServer/tests/test_server.py
dotnet run --project tests/Scurry.Protocol.Tests -c Release
dotnet run --project tests/Scurry.Central.Tests -c Release
```

`tests/Scurry.Protocol12.Tests` is the GUI-independent C# central end-to-end fixture. Supply temporary native sender/receiver server ports, their CA PEM and a temporary journal directory as its four arguments. Both test server instances must have Alice/Bob accounts as created by `tests/test_protocol12.py`. Native Windows composer tests remain in `tests/Scurry.WindowsComposer.Tests`.

| Source | Purpose |
|---|---|
| `src/Scurry.Protocol` | .NET 1.2 framing, peer engine, streaming, receipts and encrypted file helpers |
| `src/ScurryMsg.Client` | Windows peer GUI |
| `applications/ScurryMsg-Python` | Python/Tk peer GUI |
| `applications/ScurryMsgCentral-*` | Python/Tk and C# central GUIs |
| `ScurryServer` | Native Linux/Windows server and database adapters |
| `libraries/c` | C codec/callback dispatcher, C++ wrapper, OpenSSL checksum helper |
| `libraries/java` | Java codec/callback dispatcher and standard-library checksum helper |
| `libraries/python` | Python framing, peer/central engines, federation and receipts |
| `experiments/scurry-1.2` | Retained original checksum vectors/reference experiment |
| `docs/ARCHIVE-README-1.9.md` | Historical setup and previous release details |

C/C++ and Java codecs accept application-owned socket/TLS callbacks; they are not additional desktop mailbox applications. Link the optional C checksum target `scurry_receipt` with OpenSSL Crypto. The existing project license and human-use terms are unchanged.
