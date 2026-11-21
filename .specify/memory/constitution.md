<!--
Sync Impact Report
- Version change: 1.0.0 -> 2.2.0. This PR brings in the 2.1.0 revision
  (amended 2026-11-12, never merged before) and adds the 2.2.0 amendment.
- 2.1.0 (MAJOR then MINOR): removed principles "Public API Stability and
  Consumer Control" and "Traffic and Credential Confidentiality"; Target
  Compatibility and Production Code Safety renumbered to IV and V and
  expanded (x86_64-pc-windows-msvc, edition 2021, `panic!()` prohibited,
  definition of production code); clarified Library-Only Scope, divergence
  reports and Test-First commit order; logging made conditional; Governance
  gained the versioning policy and the PR requirement.
- 2.2.0 (MINOR): Development Workflow now requires Conventional Commits 1.0.0
  instead of iuricode/padroes-de-commits, and adds a Conventional Branch
  naming rule. Only abbreviated types are allowed in commits and branches
  (`feat`, `fix`, ...); `feature/` and `bugfix/` are not accepted. Same
  wording as open-middleman-cli.
- Rationale (2.2.0): widely used, tool-friendly conventions, shared by lib and
  CLI.
- Backwards compatibility (2.2.0): no effect on code or public API; existing
  commits and branches are not rewritten.
- Follow-up files: Cargo.toml (edition 2021, rust-version 1.77.2) and
  rust-toolchain.toml (1.77.2) added to match Principle IV; Cargo.lock
  regenerated as lockfile v3, which Cargo 1.77.2 can read.
-->
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
The library MUST build and run on Windows 8 Embedded, x86_64
(`x86_64-pc-windows-msvc`). It declares `rust-version = "1.77.2"` in
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
- Commit messages follow Conventional Commits 1.0.0
  (https://www.conventionalcommits.org, source:
  https://github.com/conventional-commits/conventionalcommits.org):
  `<type>[optional scope][!]: <description>`, with optional body and footers.
  Only the abbreviated type names are allowed: `feat`, `fix`, `docs`,
  `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore` and `revert`
  (never long forms such as `feature` or `bugfix`). Breaking changes are
  marked with `!` or a `BREAKING CHANGE:` footer.
- Branch names follow Conventional Branch
  (https://conventional-branch.github.io, source:
  https://github.com/conventional-branch/conventional-branch):
  `<type>/<description>`, using the same abbreviated naming. Allowed types
  are `feat/`, `fix/`, `hotfix/`, `release/` and `chore/`; the long forms
  `feature/` and `bugfix/` are not accepted. Names use lowercase letters,
  digits and hyphens (dots only in release versions), with no consecutive,
  leading or trailing hyphens or dots. The default branch (`master`/`main`)
  and `develop`, if used, are exempt.
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

**Version**: 2.2.0 | **Ratified**: 2026-11-06 | **Last Amended**: 2026-11-21
