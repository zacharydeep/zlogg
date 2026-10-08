# Locked toolchain

This file records the toolchain versions Klogg is built and verified with.
Per `AGENTS.md`, only versions actually exercised on Windows 10/11 x64 may be
reported as Windows-verified.

## Target toolchain

| Component | Version | Notes |
|---|---|---|
| CMake | >= 3.20, CMake 4 supported | 3.20 needed to map `cxx_std_20` to MSVC `/std:c++20`; `cmake/ucm.cmake` and pinned deps are made CMake 4 tolerant (see below) |
| C++ standard | C++20 | `CMAKE_CXX_STANDARD 20`, `CMAKE_CXX_EXTENSIONS OFF` |
| Qt | **6.8 LTS** (6.8.3 on Windows/macOS CI) | minimum enforced by `KLOGG_QT6_MIN_VERSION` (default `6.8`) |
| Compiler (Windows) | MSVC VS2022 (19.4x) | `ilammy/msvc-dev-cmd`, runner `windows-2022` |
| Compiler (Linux) | GCC 14.2 | Debian 13 (trixie) |
| Compiler (macOS) | AppleClang 15.0 | Xcode 15.2, runner `macos-14` (arm64) |
| Generator | Ninja | |
| Build type | RelWithDebInfo | |
| Test framework | Catch2 2.13.8 | |
| Regex backend (Windows x64) | Hyperscan | `KLOGG_USE_HYPERSCAN=ON` |
| Regex backend (Linux/macOS) | Vectorscan | `KLOGG_USE_VECTORSCAN=ON` |
| Memory allocator | TBB (Windows), mimalloc (Linux) | `KLOGG_OVERRIDE_MALLOC=OFF` on CI |
| OpenSSL (Windows packaging) | OpenSSL 3.0.16 (x64) | from the Qt tools repository; x86 ships without SSL |

### Third-party code is built at its own standard

The project is C++20, but the pinned regex engines and the `exprtk` fork are
C++17-era code and are built as C++17 (see `3rdparty/CMakeLists.txt`). Building
the regex engines as C++20 breaks them with newer libc++, and hyperscan is
additionally missing `<stdexcept>`/`<new>` includes that the newer MSVC STL no
longer provides transitively. `3rdparty/patches/` holds the one source patch
(exprtk), applied by `PATCH_COMMAND`.

### Configure options

Authoritative secret-free gate (`.github/workflows/baseline.yml`):

```
-G Ninja -DCMAKE_BUILD_TYPE=RelWithDebInfo -DKLOGG_GENERIC_CPU=ON \
-DKLOGG_USE_SENTRY=OFF -DKLOGG_OVERRIDE_MALLOC=OFF -DKLOGG_QT6_MIN_VERSION=6.8
```

`ci-build.yml` (release pipeline, needs packaging/signing secrets) additionally
uses `-DKLOGG_USE_SENTRY=ON`.

Warnings are errors (`WARNINGS_AS_ERRORS=ON` by default, `/WX` / `-Werror`).

## Verification status

| Platform | Toolchain | Status |
|---|---|---|
| Linux x64 (Debian 13, GCC 14.2, Qt 6.8.2) | C++20, `WARNINGS_AS_ERRORS=ON` | **Verified** — `ci_build` builds clean; `ctest` 3/3 pass. Also verified with **CMake 4.0.2** (configure + build + ctest) |
| macOS 14 arm64 (AppleClang 15.0, Qt 6.8.3) | C++20, Vectorscan, Sentry off | **Verified** — `ci_build` builds clean; `ctest` 3/3 pass (938 assertions in `klogg_itests`) |
| Windows x64 (MSVC VS2022, Qt 6.8.3, Hyperscan) | C++20, Sentry off | **Partial** — `ci_build` builds clean (645/645); `klogg_smoke` and `klogg_tests` (4054 assertions) pass; **`klogg_itests` segfaults** while tearing down `LogData`/`LogDataWorker` in the "search for regex" scenario. Reproducible; under investigation |

> The Linux and macOS results validate C++20 semantics, Qt 6.8 API usage and the
> CMake logic. Windows is the authoritative target and is **not** green until the
> `klogg_itests` crash is understood.
