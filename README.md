# tenzro-sdk-cpp

A **minimal** C / C++ client for [Tenzro](https://tenzro.com) nodes (chain id 13380, `0x3444`). It is not a port of the Rust SDK: it covers a handful of read-only calls, and those are tested.

> **WARNING: no TLS.** This client speaks plain `http://` only and cannot reach `https://rpc.tenzro.com` or any production endpoint. It is usable today only against local or dev nodes (or behind a TLS-terminating proxy you run). Adding TLS is a known, launch-relevant gap.

Status: **0.1.0, experimental.** Linux and macOS (POSIX sockets). Not tested on Windows.

## What is implemented

| C function | JSON-RPC method(s) |
|---|---|
| `tenzro_client_new` / `tenzro_client_free` | none (no network I/O) |
| `tenzro_get_chain_id` | `eth_chainId` |
| `tenzro_get_block_number` | `eth_blockNumber` |
| `tenzro_get_status` | `eth_blockNumber`, `net_peerCount`, `eth_syncing`, `getHealth` |
| `tenzro_get_balance` | `tenzro_getBalance` (hex string result) |
| `tenzro_last_error`, `tenzro_status_str`, `tenzro_abi_version` | none |

All of these methods are in the node's `OPEN_METHODS` list, so no auth is needed. Writes, signing, wallets, inference, identity, payments and the rest of the Rust SDK surface are **not** implemented.

**Limitations (honest list):**
- `http://` only. `https://` endpoints (including `https://rpc.tenzro.com`) are rejected with `TENZRO_ERR_UNSUPPORTED_SCHEME`. Use a local node, or put a TLS-terminating proxy in front, until TLS support is added.
- Chunked HTTP responses are rejected; the JSON parser is a minimal top-level scanner, not a general one.
- A client handle is not thread-safe.

## Design choice: hand-written C core, no Rust, no cbindgen

The core is about 300 lines of C over POSIX sockets, with a hand-written header (`include/tenzro/tenzro.h`). Reasons: the call surface is tiny, so a Rust toolchain plus cbindgen would be a heavier dependency than the code it wraps, and the C++ consumer gets a plain `cmake` build. The cost is no TLS. If TLS or a larger surface is needed, the right move is a Rust core with cbindgen behind the same ABI.

## Ownership rules (C ABI)

- `tenzro_client_new` returns a handle you own; free it once with `tenzro_client_free` (NULL is a no-op).
- All results are written into caller-supplied memory. The library returns no heap memory to be freed by you.
- `tenzro_last_error` returns a handle-owned string, valid until the next call on that handle. Do not free it.

## Build

```sh
cmake -S . -B build && cmake --build build
ctest --test-dir build        # offline tests with a fake local server
cmake --install build --prefix /usr/local
```

## C++ usage

The header-only wrapper `include/tenzro/client.hpp` (C++17) provides RAII `tenzro::Client` and a simple `Result<T>` (value or `Error`). `std::expected` is not used so the header builds under C++17.

```cpp
#include <tenzro/client.hpp>
tenzro::Client c("http://127.0.0.1:28545", 13380);
if (auto id = c.chain_id()) printf("%llu\n", (unsigned long long)id.value());
```

Run the example: `./build/tenzro_basic [endpoint] [address]` (or set `TENZRO_RPC_URL`). Output against a live dev node:

```
endpoint:     http://127.0.0.1:28545
chain id:     13380
block number: 330
peers:        0
syncing:      0
healthy:      1
balance(0x0000000000000000000000000000000000000001) = 0x0
```

## GPU Provider Kit

See [docs/gpu-provider.md](docs/gpu-provider.md) and `examples/cuda/`. Documentation plus one sample; CUDA is a compute API, not a Tenzro protocol client.

## Not done / stretch goals

- vcpkg and Conan packaging (not wired up; no port files are shipped because none have been tested).
- TLS, write calls, signing, Windows support.

## License

Apache-2.0. See [LICENSE](LICENSE).
