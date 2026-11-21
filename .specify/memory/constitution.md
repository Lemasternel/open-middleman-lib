# open-middleman-lib Constitution

## Core Principles

### I. Library-Only Scope
The project delivers a Rust library crate and nothing else. It MUST NOT
declare a `[[bin]]` target or ship an executable. Applications that expose
the library are built by consumers.

### II. Specifications Are the Source of Truth
Specifications define expected behavior; the code MUST conform to them.
When code and spec diverge, a report referencing the code location and the
spec section MUST be produced by the author of the change (or the reviewer who
detects it) and attached to the pull request. The divergence is resolved by
fixing the code or by amending the spec through the amendment process (see
Governance).

### III. Test-First (NON-NEGOTIABLE)
Tests for a behavior are written before its implementation and MUST be
observed failing before implementation code is written; the failing test
MUST be committed before the implementation commit. Each test function
covers exactly one scenario. Every public behavior is covered at least for
its success path, its error path, and its boundary conditions.

### IV. Target Compatibility
The library MUST build and run on Windows 8 Embedded (`x86_64-pc-windows-msvc`). 
It declares `rust-version = "1.77.2"` in
`Cargo.toml`, uses `edition = "2021"` (the 2024 edition requires a newer
compiler), and the development toolchain is pinned to 1.77.2 via
`rust-toolchain.toml`. Every dependency MUST build on 1.77.2 and MUST NOT
require Windows APIs unavailable on Windows 8.

### V. Production Code Safety
`unwrap()`, `expect()` and `panic!()` are prohibited in production code.
"Production code" is everything outside `#[cfg(test)]` modules and the
`tests/` directory. Failures are reported through `Result` with an explicit
error type; `Option` is used only for legitimate absence of a value. Tests
may use `unwrap()` and `expect()`.
Enforced in CI via `clippy::unwrap_used`, `clippy::expect_used` and
`clippy::panic`.

## Additional Constraints

- Rust toolchain: 1.77.2 (pinned in `rust-toolchain.toml`, declared in `Cargo.toml`).
- Logging: if the library emits logs, it uses the `log` crate
  (https://github.com/rust-lang/log) facade only, with the version pinned in
  `Cargo.lock`.
- Spec process: GitHub Spec Kit (https://github.com/github/spec-kit).

## Development Workflow

- Any behavior change starts with a spec update.
- Commit messages follow https://github.com/iuricode/padroes-de-commits.
- Code review verifies compliance with all Core Principles.
- CI builds and tests the library on the 1.77.2 toolchain.
- CI runs clippy with `unwrap_used`, `expect_used` and `panic` denied.

## Governance

- This constitution supersedes other project practices.
- Amendments require a documented rationale, maintainer approval (the
  repository owner), and a backwards-compatibility assessment, and are made
  through a pull request that updates this file.
- Constitution versioning: MAJOR for removing or redefining a principle in an
  incompatible way, MINOR for adding or materially expanding a principle or
  rule, PATCH for wording and clarifications.
- Violations must be justified in the plan's Complexity Tracking section.

**Version**: 2.1.0 | **Ratified**: 2026-11-06 | **Last Amended**: 2026-11-12
