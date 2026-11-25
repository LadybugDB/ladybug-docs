---
title: System requirements
description: Hardware and software requirements for running Ladybug graph database including memory, CPU, and OS specifications.
---

## CLI, C, and C++

The Ladybug CLI and the C and C++ APIs are pre-compiled for:
- **macOS >= 13.3**, as separate x86-64 (Intel) and arm64 (Apple Silicon) binaries.
- **Linux**, with separate x86-64 and aarch64 binaries. It requires **glibc 2.28 or later** (the `manylinux_2_28` baseline), which covers most modern Linux distributions such as **RHEL/AlmaLinux/CentOS/Rocky Linux/Oracle Linux 8.0 or later, Ubuntu 20.04 or later, and Debian 11 or later**.
- **Windows**, with separate x86-64 and aarch64 binaries, supporting most modern Windows versions such as **Windows 10 and 11**.

### Runtime requirements

The native engine links OpenSSL 3 dynamically and the native packages do not bundle it, so it must be installed on the machine that runs your app. This is intentional: OpenSSL is security-critical software with frequent vulnerability fixes, so you should rely on your OS or runtime vendor — your Linux distribution's package manager, Homebrew's `openssl@3` on macOS, or a maintained OpenSSL 3 installer on Windows — to receive timely security updates, rather than on a copy bundled with Ladybug:

- **Windows**: `libssl-3-x64.dll` and `libcrypto-3-x64.dll` (`-arm64` on ARM64) on the DLL search path, for example from an OpenSSL 3 installer or copied next to your executable, plus the Microsoft Visual C++ Redistributable (the system-wide runtime libraries, such as `MSVCP140.dll` and `VCRUNTIME140.dll`, required by apps built with Visual Studio — the VS 2015–2022 package covers the VS 2022 builds, but the architecture must match: x64 or ARM64).
- **Linux**: the distribution's OpenSSL 3 package (`libssl.so.3`, `libcrypto.so.3`).
- **macOS**: Homebrew's `openssl@3`.

When one of these is missing, the first database call throws `DllNotFoundException` for `lbug_shared` "or one of its dependencies".

## Python

Ladybug's Python API wheel has been pre-compiled for **CPython 3.7 to 3.11**. Compatibility for macOS and Windows is
the same as the pre-compiled C++ API (i.e. macOS >= 11.0, Windows 10 and 11). For Linux, the pre-compiled wheels follow
the [`manylinux_2_28`](https://github.com/pypa/manylinux) standard for both x86-64 and aarch64 architectures.

## Node.js

Ladybug's Node.js API is based on **Node-API version 5**. For a list of compatible Node.js versions, please refer to [the official documentation](https://nodejs.org/api/n-api.html#node-api-version-matrix). The native module is pre-compiled for all supported platforms and will be automatically installed when you run `npm install @ladybugdb/core`. The OS compatibility is the same as the pre-compiled CLI, C and C++ APIs.

## Java and Android

Ladybug's Java API is compatible with **Java 11+**. The OS compatibility is the same as the C/C++ APIs.
The Java API is distributed as a `JAR` file with pre-compiled native libraries for all supported platforms bundled inside. It does not require any additional dependencies.

The Java API also works on the Android ARMv8-A platform. The precompiled binaries are compiled targeting [API level 21](https://developer.android.com/tools/releases/platforms). 


## Rust

The minimum supported Rust version is 1.81.0. The Rust API is closely linked with the C++ API. By default, it compiles the C++ library from source, and this build process has been tested using MSVC on Windows, GCC on Linux, and Clang on macOS and Linux. If you're linking to the pre-built libraries instead (see [here](https://docs.rs/lbug/latest/lbug/#building)), the restrictions mentioned for the C++ API above apply.

## Go

Ladybug's Go API is a wrapper around the C API. The minimum supported Go version is 1.20.

## Swift

Ladybug's Swift API requires Swift 5.9 or later. It supports the following platforms:
- macOS v11 or later
- iOS v14 or later
- Linux (see the [Swift documentation](https://www.swift.org/platform-support/) for officially supported list of distributions)

Windows is not supported and there is no future plan to support it. 
