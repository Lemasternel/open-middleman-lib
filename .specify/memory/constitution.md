# open-middleman-lib Constitution

## Core Principles

### I. Library-Only Scope
The project delivers a Rust library crate and nothing else. It MUST NOT ship
a binary. Applications that expose the library are built by consumers.

### II. Specifications Are the Source of Truth
Specifications define expected behavior; the code MUST conform to them.
When code and spec diverge, a report referencing the code location and the
spec section MUST be produced. The divergence is resolved by fixing the code
or by amending the spec through the amendment process (see Governance).

### III. Test-First (NON-NEGOTIABLE)
Tests for a behavior are written before its implementation and MUST be
observed failing before implementation code is written. Each test function
covers exactly one scenario. Every public behavior is covered at least for
its success path, its error path, and its boundary conditions.

### IV. Public API Stability and Consumer Control
The public API is explicit and documented. The library MUST NOT:
- install a global logger or other global state;
- read configuration from fixed filesystem paths;
- terminate the process or register signal handlers.
Logging is emitted through the `log` facade only; the consumer decides how
it is recorded. Breaking changes to the public API require a MAJOR version
bump and a documented migration note.

### V. Target Compatibility
The library MUST build and run on Windows 8 Embedded. It declares
`rust-version = "1.77.2"` in `Cargo.toml`, and the development toolchain is
pinned to 1.77.2 via `rust-toolchain.toml`. Every dependency MUST build on
1.77.2; a dependency that raises the MSRV is not allowed. Only APIs available
on the Windows 8 target may be used.

### VI. Production Code Safety
`unwrap()` and `expect()` are prohibited in production code. Errors are
returned with explicit error types. The library MUST NOT panic on any input
originating from network traffic. Tests may use `unwrap()` and `expect()`.
Enforced in CI via `clippy::unwrap_used` and `clippy::expect_used`.

### VII. Traffic and Credential Confidentiality
Captured traffic, certificates, and private keys MUST NOT be written to logs
or persisted to disk.

## Additional Constraints

- Rust toolchain: 1.77.2 (pinned in `rust-toolchain.toml`, declared in `Cargo.toml`).
- Logging: `log` crate (https://github.com/rust-lang/log), version pinned in `Cargo.lock`.
- Spec process: GitHub Spec Kit (https://github.com/github/spec-kit).

## Development Workflow

- Any behavior change starts with a spec update.
- Commit messages follow https://github.com/iuricode/padroes-de-commits.
- Code review verifies compliance with all Core Principles.
- CI builds and tests the library on the 1.77.2 toolchain.
- CI runs clippy with the unwrap and expect lints denied.

## Governance

- This constitution supersedes other project practices.
- Amendments require a documented rationale, maintainer approval, and a
  backwards-compatibility assessment.
- Violations must be justified in the plan's Complexity Tracking section.

**Version**: 1.0.0 | **Ratified**: 2026-11-06 | **Last Amended**: 2026-11-06