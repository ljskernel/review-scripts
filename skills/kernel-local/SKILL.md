---
name: kernel-local
description: Lorenzo's local kernel build/boot/test workflow wrapping ~/code/review-scripts. Load whenever the kernel skill is loaded, and ALWAYS load before configuring or building a kernel, running mm/VMA selftests, booting a test kernel, or doing per-commit build checks in any kernel tree.
---

# Local kernel build & test workflow

All build/boot/test operations go through the wrappers in `~/code/review-scripts`
(on `$PATH`). Never hand-roll `make`/`vng` invocations when a wrapper exists.
For the full catalogue read `~/code/review-scripts/README.md`. Essentials:

## Toolchain

Everything builds with clang: every build sets `LLVM=1`. If you run a bare
config target yourself, keep the toolchain consistent — use
`make LLVM=1 olddefconfig` or `make LLVM=1 oldconfig`, never the plain
gcc-defaulting forms.

## Configure

- `review-config` / `review-config-debug` — configure via `hooks/kernel-config[-debug]`
- `review-reconfig[-debug]` — same, but `make mrproper` first
- Config changes belong in `~/code/review-scripts/hooks/kernel-config*`, not
  ad-hoc edits in the kernel tree

## Build

- `review-mk` — iterate: all-core `LLVM=1` build, aborts on first stderr output,
  emits compile_commands.json (`review-mk-nosym` skips that)
- `review-build` — configure + build via the hooks; `review-rebuild[-debug]`
  for a from-scratch reconfig + build
- `review-build-commits [from] [to]` — build every commit in a range
  (both default to `HEAD`)
- `review-build-commits-pedantic` — per-commit allnoconfig/debug/rust/nommu/arm64
- arm64 variants exist: `review-mk-arm64`, `review-vng-arm64`,
  `review-mm-tests-arm64`

## Boot / test

- `review-vng [args]` — boot the current tree in virtme-ng (panic_on_warn,
  mm-selftest-friendly config); `review-vng-debug` is the same but outputs
  kernel log output to stdout.

- `review-mm-tests [--vma-tests-only|--mm-tests-only]` — build/run VMA tests
  and mm selftests against the current tree; no active review session needed
- `review-mm` — review-mm-tests + pedantic build of the current commit

## Rules

- Prefer `review-mk` for build iteration; `review-build` after config changes.
- `review-check-mm-tests` runs with sudo and R/W host filesystem access — ask
  before running it.
- Active-review scripts (`review-start`, `review-diff`, `review-rebase`, ...)
  manage `review/<name>-vN` branches and `review-*` tags; don't delete those by
  hand — use `review-stop` / `review-clean`.
