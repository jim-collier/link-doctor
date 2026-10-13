<!-- markdownlint-disable MD007 -- Unordered list indentation -->
<!-- markdownlint-disable MD010 -- No hard tabs -->
<!-- markdownlint-disable MD041 -- First line in a file should be a top-level heading -->

<!-- TOC ignore:true -->
# Project backlog

<!-- TOC ignore:true -->
## Table of contents

<!-- TOC -->

- [Introduction](#introduction)
- [Issues](#issues)
- [Old format](#old-format)
	- [Bugs](#bugs)
	- [Features and enhancements](#features-and-enhancements)
	- [Done](#done)
		- [Done - Bugs](#done---bugs)
		- [Done - Features and enhancements](#done---features-and-enhancements)
	- [Deferred](#deferred)
	- [Canceled](#canceled)
- [Template](#template)

<!-- /TOC -->

## Introduction

Going forward, new issues in the new template at the bottom of this file, will go in the '## New format' section only. No more status emojis. Refer to '## Reference' for sort order. Issues in the old format (with status emojis) won't be refactored, but will continue to be worked until moved to closed, canceled, or deferred sections, and emojis updated. (Eventually this will all be moved to nano-git-db anyway. This new template is an intermediate effort to make issues going forward more structured and importable.)

This is a product backlog just for pre-v1.0.0 release. After that, bugs, features, and enhancements will be managed in Github Issues, and/or [todo.md](../todo.md)

## Issues

- Run on the last Core 2 Duo, and AMD CPUs from that era.
	- ID: 2026101301161865
	- Type: Enhancement
	- Status: Queued
	- Needs external testing: Yes, on an old CPU or an emulated one.
	- Priority [Feature|Enhancement] | Severity [Bug]: Low
	- Opened: 20261013-011618
	- Opened by: JC
	- Target OS: Linux, Windows and FreeBSD on x86_64, where this project builds them. Macs that old can't run a current macOS anyway.
	- Requirements [Feature]:
		- Release builds run on the last Core 2 Duo (Penryn, 2008), and on AMD Phenom II and Athlon II, if that's reasonably doable.
			- Penryn has SSSE3 and SSE4.1 but no POPCNT. The AMD ones have POPCNT but no SSSE3. So the shared floor is plain x86-64, also called x86-64-v1.
		- If that isn't doable, the floor is the first Core i3/i5/i7 (Nehalem, 2008), which is x86-64-v2. AMD's first chip at that level is Bulldozer, 2011.
		- Faster code for newer CPUs is still fine, but it's picked at run time by a CPU feature test, never by the build target.
		- Tests run the release build on an emulated Penryn and Phenom II, through Intel SDE or QEMU's `-cpu Penryn` and `-cpu phenom`.
	- Progress log:
		- 20261013: Go builds for plain x86-64 by default (`GOAMD64=v1`), and nothing here sets it, so this may already hold. Pin `GOAMD64=v1` in the release build so a stray environment can't raise it. The standard library already picks faster code at run time.

## Old format

### Bugs

### Features and enhancements

- Verify the Windows paths on real Windows hardware: symlink recreate (file vs dir flag, privilege), junction reparse-buffer write, and `.lnk` `TargetPath` set/save. (Because...built and cross-compiled from Linux.)

- Use the improved file selection filtering engine defined (and created?) in the section '##### Input selection' of '../rapid-photo-downloader-cli/github/project/design.md'.

### Done

#### Done - Bugs

#### Done - Features and enhancements

- ✅ Scaffold Go project, build/cross-compile matrix, CICD engine + config + test harness.

- ✅ Core CLI: custom arg parser (prefix abbreviation, `=`/space values, positional START/FROM/TO).

- ✅ Filters: include/exclude regex, name and wholename globs, and re-include, as one ordered pipeline.

- ✅ Recursive walk collecting symlinks, loop-safe (no dir-symlink follow), `--max-depth`.

- ✅ Repoint engine: regex `--from` + template `--to` (`$1`), literal `--literal`, apply-by-default, or show first with `--dry-run`.

- ✅ POSIX (Linux/BSD/macOS) symlink read/atomic rewrite.

- ✅ Windows symlink + NTFS junction + `.lnk` read/rewrite (compile-verified; see verify item below).

- ✅ Go unit tests (args/filters/transform/walk) + integration harness over scratch symlink trees.

- ✅ When run with no args, behave like `--help` (print help, exit 0) instead of silently scanning the current directory. Pass `.` explicitly to list it.

- ✅ OS-aware dogfood destinations in `cicd/config.bash`: `DOGFOOD_FIXED_DESTS` is now selected by `uname -s` (Linux / macOS / Windows-under-Git-Bash), with sensible per-OS defaults. Engine unchanged (still installs into the first existing + writable dir).

- ✅ `--xdev`: exact alias for `--no-cross-device` (find's spelling), via the `flagAliases` table.

- ✅ Rigorous feature/combination testing in `cicd/test.bash`, all on scratch symlink trees (never live files): device no-op, target filters + `--inc` alias, follow-links reach-behind + cycle termination, print0 record count + no-summary, renormal relative/absolute round-trip, confirm yes/no, a combined multi-flag run, and invalid-combination exit codes. 56 checks total.

- ✅ `--confirm`: preview the whole plan, then one `y/N` prompt before writing anything (full-plan gate, not per-link). `process` builds the plan first, so one code path drives dry-run, confirm, and apply. Mutually exclusive with `--print0`.

- ✅ `--renormal-relative` / `--renormal-absolute`: re-normalize each target's spelling (relative to the link, or cleaned absolute). Runs after `--from`/`--to`, or standalone to tidy existing targets. Mutually exclusive; logical-path only (no symlink resolution), so the link still points at the same place.

- ✅ `-0` / `--print0`: machine output, one link path per NUL record, no human decorations or summary (list mode = matches, edit mode = links that change). For `xargs -0` style scripting.

- ✅ `-L` / `--follow-links`: descend into directory symlinks during the walk, loop-safe via a visited-canonical-path set (each directory walked once). Default walk still never follows.

- ✅ `--inc-target` / `--exc-target`: select links by their current target (where they point), not just their own path. Reuses the filter engine as a second ordered pipeline over the target. `--inc`/`--exc` short spellings preserved via an exact-alias table.

- ✅ `--no-cross-device`: don't descend into directories on another filesystem (`find -xdev` style). Per-OS device probe (POSIX `st_dev`, Windows volume serial); prunes at directory boundaries during the walk.

- ✅ Selection engine split into its own reusable package (`source/filter/`), independent of the CLI, so a later file-lister project can use it as-is. The CLI keeps a thin flag-to-rule adapter.

- ✅ Fuzz and security test suites: arg parser, glob translator, regex engine, and the selection pipeline under random input; link-follow safety, symlink-cycle safety, and timeout-bounded matching.

### Deferred

### Canceled

## Template

### Old format

- 🔘 Not started

- 🛠️ Started, and/or partially complete

- 🔬 Testing not started or finished

- ✋ Defer

- ✅ Complete

- 🚫 Canceled

### New format

- Notes:

	- Only use rows that you actually need or expect will be filled in. Always fill in the title, ID, Type, Status, Opened and Created by.

	- The ID is the local time to the hundredth of a second. Opened is when it was written down, which may differ. (Use a keyboard macro and possibly something like project 'zuid' to generate.)

	- Status values meaning: Testing means the fix is in and checks are running or still to run. Waiting on signoff means automated testing passed. Moot means something else changed that made it irrelevant. Canceled means it still applies but was decided against. Waiting for testing means the fix is in and waits on a long CI run or an outside test host. Can't reproduce means a real attempt to reproduce it failed.

	- As issues are worked, and statuses change, place them in correct sorting order within the list:
		- First by status: Waiting for answers, Waiting on signoff, Testing, Waiting for testing, Can't reproduce, Stalled, Started, Queued, Done, Deferred, Canceled, Moot
		- Then by severity|priority: Critical, High, Avg, Low
		- Then by type: Bugs, [not bugs together]

	- Rows marked [Bug] are for bugs only, and rows marked [Feature] for features and enhancements. Priority and Severity share one row and one scale. Priority is for a Feature or Enhancement, and Severity for a Bug. Children are not nested. They sit at the top level and point back with Parent ID.

Template:

- Title
	- ID: YYYYmmDDHHMMSSNN
	- Type: [Bug|Feature|Enhancement|Task]
	- Status: [Queued|Waiting for answers|Waiting on signoff|Waiting for testing|Started|Testing|Stalled|Can't reproduce|Moot|Canceled|Deferred|Done]
	- Needs local test suite run?:
	- Needs external testing:
	- Priority [Feature|Enhancement] | Severity [Bug]: [Critical|High|Avg|Low]
	- Opened:
	- Opened by:
	- Assigned to:
	- Parent ID:
	- Prereq IDs:
	- Related IDs:
	- Target OS:
	- Test environment:
	- Version and build:
	- Requirements  [Feature]:
		- Hierarchical bulleted list.
	- Steps to reproduce [Bug]:
		- …
	- Incorrect behavior [Bug]:
	- Expected behavior [Bug]:
	- Reproduced [Bug]: [No, or when, where and how]
	- Possible cause [Bug]:
	- Actual cause [Bug]:
		- …
	- Estimated effort: [High|Avg|Low]
	- Actual effort: [High|Avg|Low]
	- Progress log:
		- …
	- Decisions:
		- …
	- Actual fix [Bug]:
	- Branch:
	- Commit:
	- Test case: [Reason not applicable, or CI test case #]
	- Acceptance signoff:
	- Superseded by ID:
	- Closed:
