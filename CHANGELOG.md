# Changelog

All notable changes to **ediabasx-embedded** are documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)
and the project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

Tags are bare semver (no `v` prefix), e.g. `0.1.0`.

## [Unreleased]

### Added
- `LICENSE` (PolyForm Noncommercial 1.0.0) — the project was always
  intended as noncommercial; this commits the explicit terms in-tree.
- README trailer: *Right to Repair*, *Support*, *License*, *Disclaimer*
  sections to align with the rest of the bimmerz tooling family.
- `.github/FUNDING.yml` — GitHub Sponsors + Buy Me a Coffee links.

## [0.1.0] — 2026-06-13

Initial release. C11 port of the BMW BEST/2 interpreter with feature-parity
to the TypeScript reference for the opcode-dispatch surface and the
`Ediabas` wrapper-class result shape. Consumable both as a standalone POSIX
library and as an ESP-IDF managed component.

### Added

- **BEST/2 opcode dispatch** across the seven handler files in `src/opcodes_*.c`:
  arithmetic, flow, string/binary, result-set, table, comm, and misc. All
  semantics mirror the TS reference at `packages/interpreter/` exactly.
- **`edxn_ediabas_t` wrapper** (`src/ediabas.c`, `include/ediabasx/ediabas.h`)
  — recommended entry point. Owns the persistent system-results accumulator,
  drives INITIALISIERUNG + IDENTIFIKATION bootstrap on GRP load, performs
  GRP→PRG variant swap, materialises the C-API-compatible result shape
  (`[system_set, …data_sets]`) into `built_sets`.
- **GRP→PRG variant resolution + cache.** A `.grp` header (`version == 0`)
  is detected on load; the wrapper runs IDENT, looks up `VARIANTE`, calls the
  registered `sgbd_loader` to fetch the matching `.prg`, and caches the
  group→variant mapping for subsequent loads of the same group.
- **`tabsetex` external table loading** (`opcode 0xAA`) — when arg1 names an
  external file and a `table_loader` is registered, that file's table
  registry replaces the active source until `tabset` resets it.
- **Binary-payload jobs (`apiJobData` channel).** `edxn_ediabas_exec_data()`
  and `edxn_vm_exec_data()` accept a `(bin, bin_len)` buffer fed to `pary`
  (`0x7F`) and slot-indexed `parb`/`parw`/`parl`/`parr` reads. Payload size
  capped at `EDXN_PARAM_BINARY_MAX` (1 KiB); excess is truncated to mirror
  the TS runtime, where the SGBD's own input-length check is authoritative.
  Legacy non-`_data` entry points are thin wrappers passing `NULL, 0`.
- **Trap machinery** matching TS `isTrapErrorDetected` semantics:
  `error_trap_mask` (`settmr`/`gettmr`), `error_trap_bit_nr` (`sett`/`clrt`/`eerr`),
  `jt`/`jnt` jump-on-trap, and the `0x40000000` "generic error" sentinel
  used by `comm_error()` so SGBDs can handle transport failures via their
  own `jt` checks.
- **Polymorphic operand readers** in `vm_internal.h` (`edxn_resolve_int`,
  `edxn_resolve_binary`, `edxn_resolve_string`) — the single point of truth
  for "this opcode accepts any operand type and reads it as the natural shape".
- **POSIX platform backend** (`platform/posix/`) — termios-based K-line
  serial transport, file-backed SGBD + table loaders with case-insensitive
  directory scan for BMW disk layout.
- **ESP-IDF component support.** `CMakeLists.txt` is dual-mode: when
  consumed under `ESP_PLATFORM` it short-circuits to `idf_component_register`;
  otherwise it falls through to the standalone POSIX project build. An
  `idf_component.yml` manifest is shipped so the IDF Component Manager can
  pull it directly.
- **Test harness** (`test/`) — `test_prg`, `test_vm`, `test_ediabas`
  ctest cases plus the `edxn_run` CLI driver (`./build/edxn_run --help`).
- **CI** (`.github/workflows/build.yml`) — standalone POSIX build on
  ubuntu-latest + macos-latest. ctest deferred to a follow-up release
  (a `test_results` case in `test_vm.c` aborts on macOS).

### Known gaps vs the TS reference

Carried forward into 0.2 planning, not blockers for 0.1:

- **Comm transport surface is a subset** of TS `CommunicationInterface`:
  frequent-mode, port write, programming voltage, SI relay, loopback test,
  and multi-byte `xstate` only work when the backend implements the
  corresponding callback. Missing callbacks degrade to a silent no-op.
- **No simulation backend** — the TS port has `SimulationInterface` for
  record/replay of job traffic; the native port doesn't ship one yet.
- **No cooperative break / cancel signal.** TS exposes
  `Interpreter.requestBreak()` + `Ediabas.break()` for aborting an
  in-flight job with `EDIABAS_BIP_0008`; the C port has no equivalent.

[Unreleased]: https://github.com/emdzej/ediabasx-embedded/compare/0.1.0...HEAD
[0.1.0]: https://github.com/emdzej/ediabasx-embedded/releases/tag/0.1.0
