<!--
Sync Impact Report
- Version change: 2.2.0 -> 2.3.0 (MINOR).
- Added principle: VI. Mandatory Logging. Logging is no longer conditional
  ("if the library emits logs"); every significant operation and every failure
  path MUST be logged. The principle does not name a logging library.
- Removed from Additional Constraints (stack, not constitution):
  - the logging library and its link (`log` crate) and the requirement to pin
    it in `Cargo.lock`;
  - the "Rust toolchain: 1.77.2" bullet, already covered by Principle IV.
- Kept on purpose: Windows 8 Embedded target, `x86_64-pc-windows-msvc`,
  `rust-version = "1.77.2"`, edition 2021 and the pinned toolchain in
  Principle IV, because they are compatibility requirements of the main
  target, not stack choices. GitHub Spec Kit (process) and the clippy lints
  that enforce Principle V are also kept.
- Cleanup: removed lines and bullets duplicated by an earlier merge, and fixed
  the footer, which still said 2.1.0 although the 2.2.0 amendment was applied.
- Rationale: logging is a core requirement of a debugging proxy; the library
  chosen to implement it is an implementation decision that belongs to the
  plan, not to the constitution.
- Backwards compatibility: no code exists yet that emits logs, so nothing
  breaks; future features MUST include logging in their specs and plans.
- Follow-up files: none required. Specs and plans written from now on MUST
  describe the logging of each feature.
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

### VI. Mandatory Logging
Logging is REQUIRED, not optional. The library MUST emit log records for every
significant operation (for example: accepting and closing connections,
receiving and forwarding requests and responses, TLS handshakes, configuration
changes) and for every failure path, before the error is returned to the
caller. Each record MUST carry a severity level (error, warning, info, debug,
trace) and enough context to identify the operation it describes. The library
emits records but MUST NOT decide where they are written; the output
destination is configured by the consuming application. The choice of logging
library is an implementation decision recorded in the plan, not in this
constitution. Every spec and plan MUST state what its feature logs, and code
review rejects changes that add operations or failure paths without logging.

## Additional Constraints

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
- Code review verifies compliance with all Core Principles, including the
  presence of logging required by Principle VI.
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

**Version**: 2.3.0 | **Ratified**: 2026-11-06 | **Last Amended**: 2026-11-24
