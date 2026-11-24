<!--
Sync Impact Report
- Version change: 2.3.0 -> 3.0.0 (MAJOR).
- Rationale: the constitution states principles and rules only; it MUST NOT
  name the programming language, its version, toolchain files, build manifests
  or lock files, linters or any other stack detail. Those are implementation
  decisions recorded in the plan.
- Redefined principles (stack-specific rules removed or rewritten in a
  language-agnostic way):
  - I. Library-Only Scope: no longer names a language crate or a build-manifest
    binary target; the project still MUST NOT ship an executable.
  - IV. Target Compatibility: removed the target triple, the minimum language
    version, the language edition, and the pinned toolchain and build-manifest
    files. Windows 8 Embedded (x86_64) remains the main target, and every
    dependency and the toolchain MUST still support it.
  - V. Production Code Safety: rewritten without language constructs. Aborting
    the process on unexpected states is still prohibited in production code;
    failures are still reported to the caller as explicit, typed errors; the
    rule is still enforced automatically in CI, with the tool chosen in the
    plan.
- Development Workflow: the CI rule pinned to a language version became
  "CI builds and tests for the main target"; the named linter rules became an
  automated check for Principle V.
- Unchanged: II, III, VI (Mandatory Logging), Additional Constraints (GitHub
  Spec Kit), commit and branch rules, Governance.
- Backwards compatibility: no rule is made stricter; existing code and build
  configuration still comply. Removing rules from the constitution does not
  remove them from the repository: the current toolchain and lint settings stay
  as implementation decisions.
- Follow-up files: none required. The plan template already keeps language,
  version and tools in its Technical Context section.
-->
# open-middleman-lib Constitution

## Core Principles

### I. Library-Only Scope
The project delivers a library and nothing else. It MUST NOT build or ship
an executable. Applications that expose the library are built by consumers.

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
Windows 8 Embedded (x86_64) is the main target. The library MUST build and
run on it, and support for it MUST NOT be dropped or degraded. The toolchain
and every dependency MUST support Windows 8 Embedded, and nothing used by the
library MAY require operating system APIs or features that are unavailable on
Windows 8. A change that breaks Windows 8 Embedded support is rejected, even
if it works on newer systems.

### V. Production Code Safety
Production code MUST NOT abort or crash the process on an unexpected state,
and MUST NOT assume that an operation which can fail has succeeded. "Production
code" is everything except test code. Failures are reported to the caller as
explicit, typed errors; an empty value is used only for the legitimate absence
of a value, never to signal an error. Tests may assert and abort freely.
This rule MUST be enforced by an automated check in CI.

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
- CI builds and tests the library for the main target (Principle IV).
- CI runs the automated check that enforces Principle V and fails on any
  violation.

## Governance

- This constitution supersedes other project practices.
- Amendments require a documented rationale, maintainer approval (the
  repository owner), and a backwards-compatibility assessment, and are made
  through a pull request that updates this file.
- Constitution versioning: MAJOR for removing or redefining a principle in an
  incompatible way, MINOR for adding or materially expanding a principle or
  rule, PATCH for wording and clarifications.
- Violations must be justified in the plan's Complexity Tracking section.

**Version**: 3.0.0 | **Ratified**: 2026-11-06 | **Last Amended**: 2026-11-24
