---
title: Repositories
---

# List of SecureCells repositories

The SecureCells projects includes full-stack development efforts, with modifications to both
hardware and software. The relevant repositories used during development are listed below.

Software developed for SecureCells is typically tested with QEMU and OpenSBI firmware.
QEMU emulates the architecture, optionally including SCxxx instructions.
When QEMU does not emulate SCxxx instructions, OpenSBI emulates them during trap handling.

While `seccell-archtests` runs on baremetal, the remaining microbenchmarks currently
run using `seccell-seL4`.

| **Emulation**                                                                                                                                        
| [QEMU                  ](https://github.com/HexHive/seccell-qemu)                 | QEMU implementation based on rv64gc            |
| [OpenSBI               ](https://github.com/HexHive/seccell-opensbi)              | Firmware implementing SCxxx instructions       |
| **FPGA**                                                                                                                               
| [RocketChip            ](https://github.com/HexHive/seccell-rocket)               | RocketChip RTL implementation                  |
| **Compilation**                                                                                                                                 
| [GNU toolchain         ](https://github.com/HexHive/seccell-riscv-gnu-toolchain)  | Toolchain for compiling SC-riscv               |
| [Binutils/GDB          ](https://github.com/HexHive/seccell-riscv-binutils-gdb)   | Binutils adding instruction opcodes            |
| **Operating Systems**                                                                                                                                 
| [seL4                  ](https://github.com/HexHive/seccell-seL4)                 | Microkernel OS                                 |
| [Linux                 ](https://github.com/HexHive/seccell-linux)                | Coming up...                                   |
| **Testing**                                                                                                                                 
| [Architectural Tests   ](https://github.com/HexHive/seccell-archtests)            | Baremetal tests for instructions               | 
| **Applications**                                                                                                                                 
| [ubenchmark - browser  ](https://github.com/HexHive/seccell-browser)              | Browser example from the paper                 |
| [ubenchmark - memcache ](https://github.com/HexHive/seccell-memcache)             | `memcached`-like example from the paper        |
| **seL4 support**                                                                                                                               
| [libseccells           ](https://github.com/HexHive/seccell-seL4_libseccells)     | Support library (incl. threading, macros)      |
| [seL4 playground       ](https://github.com/HexHive/seccell-sel4-playground)      | Project for building seL4 applications         |
