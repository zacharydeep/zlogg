# Locked toolchain

This file records the toolchain versions Klogg is built and verified with.
Per `AGENTS.md`, only versions actually exercised on Windows 10/11 x64 may be
reported as Windows-verified.

## Target toolchain

| Component | Version | Notes |
|---|---|---|
| CMake | >= 3.20 | required to map `cxx_std_20` to MSVC `/std:c++20` |
| C++ standard | C++20 | `CMAKE_CXX_STANDARD 20`, `CMAKE_CXX_EXTENSIONS OFF` |
| Qt | **6.8 LTS** (6.8.3 on Windows/macOS CI) | minimum enforced by `KLOGG_QT6_MIN_VERSION` (default `6.8`) |
| Compiler (Windows) | MSVC VS2022 (19.4x) | `ilammy/msvc-dev-cmd`, runner `windows-2022` |
| Compiler (Linux) | GCC 14.2 | Debian 13 (trixie) |
| Generator | Ninja | |
| Build type | RelWithDebInfo | |
| Test framework | Catch2 2.13.8 | |
| Regex backend (Win x64) | Hyperscan | `KLOGG_USE_HYPERSCAN=ON`, `KLOGG_USE_VECTORSCAN=OFF` |
| Regex backend (Linux) | Vectorscan | `KLOGG_USE_VECTORSCAN=ON` |
| Memory allocator | TBB (Windows), mimalloc (Linux) | `KLOGG_OVERRIDE_MALLOC=OFF` on CI |

### Windows CI configure options

```
-G Ninja -DCMAKE_BUILD_TYPE=RelWithDebInfo -DKLOGG_GENERIC_CPU=ON \
-DKLOGG_USE_SENTRY=ON -DKLOGG_OVERRIDE_MALLOC=OFF -DKLOGG_QT6_MIN_VERSION=6.8
```

Warnings are errors (`WARNINGS_AS_ERRORS=ON` by default, `/WX` / `-Werror`).

## Verification status

| Platform | Toolchain | Status |
|---|---|---|
| Linux x64 (Debian 13, GCC 14.2, Qt 6.8.2) | C++20, `WARNINGS_AS_ERRORS=ON` | **Verified** — `ci_build` builds clean; `ctest` 3/3 pass |
| Windows x64 (MSVC VS2022, Qt 6.8.3) | Hyperscan, Sentry | **Pending** — CI job configured in `.github/workflows/ci-build.yml`, not yet run/confirmed |
| macOS (Qt 6.8.3) | | **Pending** |

> The Linux results validate C++20 semantics, Qt 6.8 API usage and the CMake
> logic. They do **not** validate MSVC warnings under `/WX`, MSVC STL behaviour,
> Windows packaging, Hyperscan-on-Windows, or Qt 6.8 MSVC binaries.
