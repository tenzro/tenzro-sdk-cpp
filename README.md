# tenzro-sdk-cpp

The official Tenzro SDK for C and C++: a stable C ABI client for [Tenzro](https://tenzro.com) nodes, plus a header-only C++ wrapper (chain id 13380, `0x3444`).

> **Known limitation: no TLS yet.** This client speaks plain `http://` only and cannot reach `https://rpc.tenzro.com` or any production endpoint directly. Use a local or dev node, or put a TLS-terminating proxy in front, until TLS support ships.

## Install

```sh
cmake -S . -B build && cmake --build build
cmake --install build --prefix /usr/local
```

## Quickstart (C++)

```cpp
#include <tenzro/client.hpp>
tenzro::Client c("http://127.0.0.1:28545", 13380);
if (auto id = c.chain_id()) printf("%llu\n", (unsigned long long)id.value());
```

Run the example: `./build/tenzro_basic [endpoint] [address]` (or set `TENZRO_RPC_URL`).

## Methods

| C function | JSON-RPC method(s) |
|---|---|
| `tenzro_client_new` / `tenzro_client_free` | none (no network I/O) |
| `tenzro_get_chain_id` | `eth_chainId` |
| `tenzro_get_block_number` | `eth_blockNumber` |
| `tenzro_get_status` | `eth_blockNumber`, `net_peerCount`, `eth_syncing`, `getHealth` |
| `tenzro_get_balance` | `tenzro_getBalance` (hex string result) |
| `tenzro_last_error`, `tenzro_status_str`, `tenzro_abi_version` | none |

Writes, signing, wallets, inference, identity and payments are not yet implemented.

### Ownership rules (C ABI)

- `tenzro_client_new` returns a handle you own; free it once with `tenzro_client_free` (NULL is a no-op).
- All results are written into caller-supplied memory — the library returns no heap memory for you to free.
- `tenzro_last_error` returns a handle-owned string, valid until the next call on that handle; do not free it.

### C++ wrapper

`include/tenzro/client.hpp` (C++17) provides RAII `tenzro::Client` and a `Result<T>` (value or `Error`); `std::expected` is not used, so it builds under plain C++17.

## Design: hand-written C core, no Rust, no cbindgen

The core is about 300 lines of C over POSIX sockets with a hand-written header (`include/tenzro/tenzro.h`): the call surface is small enough that a Rust toolchain plus cbindgen would be heavier than the code it wraps, and the C++ side gets a plain `cmake` build. The cost is today's TLS gap above. A Rust core with cbindgen behind the same ABI is the planned path once TLS or a larger surface is needed.

## GPU Provider Kit

See [docs/gpu-provider.md](docs/gpu-provider.md) and `examples/cuda/` — docs plus one sample for providers serving models on NVIDIA GPUs. CUDA is a compute API, not a protocol client, so it lives here rather than as its own SDK.

## Compatibility

Built and tested on Linux x86_64 (cmake + ctest, both clean; the example runs against a live Tenzro node). macOS is in CI but not yet run; Windows is not yet supported. `http://` only (see above) — chunked HTTP responses are also rejected, and the JSON parser handles the documented methods' responses, not arbitrary JSON. A client handle is not thread-safe. vcpkg/Conan packaging isn't wired up yet.

## Links

[tenzro-network](https://github.com/tenzro/tenzro-network) (the protocol and node) and the other official SDKs in the [tenzro](https://github.com/tenzro) organization (TypeScript, Rust, Python, Go, Swift, Kotlin, .NET).

## License

Apache-2.0. See [LICENSE](LICENSE).
