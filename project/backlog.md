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

This is a product backlog just for pre-v1.0.0 release. After that, bugs, features, and enhancements will be managed in Github Issues.

## Issues

- Build and test in a pinned container.
	- ID: 2026101303344551
	- Type: Enhancement
	- Status: Queued
	- Priority [Feature|Enhancement] | Severity [Bug]: High
	- Opened: 20261013-033445
	- Opened by: JC
	- Target OS: Linux
	- Requirements [Feature]:
		- `cicd.bash --container` runs the build and test stages in an image built from a pinned recipe in `cicd/container/`. Toolchains come from pinned downloads with checksums.
		- Toolchains on the build box clash between projects, such as 2 Zig versions on one link. A container keeps this project's pins apart.
		- Other test boxes stay native, since their OS is what's being tested.
		- zuid's `--container` stage is the model. gitsby and convert-base-v2 have one too.
		- 2026-10-24: this is the CICD reset's containers item now. Build and test run in the container by default, and `--no-container` runs them on the host, in place of an opt-in `--container`. A missing docker or an image that won't build falls back to the host with one line saying so, and never fails the run. Priority raised from Low.
		- Sweeping override, 2026-10-24: this supersedes any earlier decision on this project that conflicts with it.
		- It overrides the opt-in `--container` flag above.

- CICD reset: other hosts and VMs
	- ID: 2026102416310500
	- Type: Enhancement
	- Status: Queued
	- Needs external testing: Yes, on the Windows, macOS, and arm64 Linux test boxes.
	- Priority [Feature|Enhancement] | Severity [Bug]: High
	- Opened: 20261024-163105
	- Opened by: JC
	- Requirements [Feature]:
		- Sweeping override, 2026-10-24: this supersedes any earlier decision on this project that conflicts with it.
		- It covers the old item to verify the Windows paths on real Windows hardware.
		- The default run tests the Windows, Intel macOS, and arm64 Linux builds on the test boxes, and FreeBSD once it's built. `--no-remote` skips them.
		- Windows tests go to b29w first, and to vm925w only for GPU or timed work. Symlink and junction checks there need a user that can make symlinks.
		- Each box is held through the shared host lock while it's in use. A VM is put back the way it was found, through the VM state helper.
		- A box that isn't there shows as SKIP with a warning, and the run goes on.
		- Each box gets a work folder for this project under its temp root.

- CICD reset: container cleanup
	- ID: 2026102416310501
	- Type: Enhancement
	- Status: Queued
	- Priority [Feature|Enhancement] | Severity [Bug]: High
	- Opened: 20261024-163105
	- Opened by: JC
	- Prereq IDs: 2026101303344551
	- Requirements [Feature]:
		- Sweeping override, 2026-10-24: this supersedes any earlier decision on this project that conflicts with it.
		- Every container, image, and volume the pipeline makes gets the `dev.project` and `dev.run` labels. Containers are named by project, run, and step.
		- Containers run with `--rm` unless `--keep-temp` is passed.
		- At the end of a run, on failure and Ctrl+C too, the run's containers are removed by label. Then the project's dangling images, and its older tags once a newer one passed, keeping the current tag and the one before.
		- Then `docker builder prune -f --filter until=168h`. Cache volumes are kept under 2 GB.
		- Nothing without this project's label is touched. No `docker rmi -f` and no `docker system prune`.

- CICD reset: CPU and priority
	- ID: 2026102416310502
	- Type: Enhancement
	- Status: Queued
	- Priority [Feature|Enhancement] | Severity [Bug]: High
	- Opened: 20261024-163105
	- Opened by: JC
	- Requirements [Feature]:
		- Sweeping override, 2026-10-24: this supersedes any earlier decision on this project that conflicts with it.
		- Build and test use at most half the CPUs, or a quarter on a busy box. Busy means the 1-minute load is over half the CPU count, checked at the start and before each step. Nothing caps them now.
		- Fuzzing and leak checks use a quarter, busy or not.
		- The counts are worked out in one place and passed to every tool that runs in parallel, such as `GOMAXPROCS`, `go test -p`, and `-parallel`.
		- Every step runs under `nice -n 19` and `ionice -c 3`. Containers get `--cpus` and `--cpu-shares 256`.
		- Other hosts follow the same rules with their own CPU count and load.

- CICD reset: standard flags
	- ID: 2026102416310503
	- Type: Enhancement
	- Status: Queued
	- Priority [Feature|Enhancement] | Severity [Bug]: High
	- Opened: 20261024-163105
	- Opened by: JC
	- Requirements [Feature]:
		- Sweeping override, 2026-10-24: this supersedes any earlier decision on this project that conflicts with it.
		- `cicd.bash` takes the shared flags: `--quiet` and `-q`, `--message` and `-m`, `--slow` as another name for `--long`, `--leakcheck`, `--no-leakcheck`, `--backlog-run-quick` with optional areas, `--backlog-run-full`, `--merge-gate`, `--no-container`, `--no-remote`, `--no-sync`, `--keep-temp`, and `--list`. `--gate` and `--ci` are other names for the quick backlog run and the merge gate. The existing flags stay.
		- `--long` runs everything, and a plain run takes no more than about an hour. `--quick` runs what's fast in about 10 minutes, and skips only the cross-builds now.
		- The backlog runs and the merge gate never dogfood, publish, commit, push, install, sync, or rewrite a tracked file, and never prompt.
		- A pre-push hook runs `--merge-gate` on a push to main. It skips the gate when the commits touch only docs and the other paths listed in one place.
		- No gate on other pushes. There's no dev branch yet.

- CICD reset: step output
	- ID: 2026102416310504
	- Type: Enhancement
	- Status: Queued
	- Priority [Feature|Enhancement] | Severity [Bug]: High
	- Opened: 20261024-163105
	- Opened by: JC
	- Requirements [Feature]:
		- Sweeping override, 2026-10-24: this supersedes any earlier decision on this project that conflicts with it.
		- Each step prints a 79-character rule, a `Starting` line, its output, and a `PASS` or `FAIL` line, in the shared format with a full local timestamp and "Step N of M".
		- One blank line between steps, and one at the end.
		- Steps turned off aren't counted. The first section lists them with the flag or reason for each.
		- The stage headers read "1/6 Format" with a time of day now, under a shorter rule.

- CICD reset: a test for every Done item
	- ID: 2026102416310505
	- Type: Enhancement
	- Status: Queued
	- Priority [Feature|Enhancement] | Severity [Bug]: High
	- Opened: 20261024-163105
	- Opened by: JC
	- Requirements [Feature]:
		- Sweeping override, 2026-10-24: this supersedes any earlier decision on this project that conflicts with it.
		- Every Done feature and bug gets a regression test in CI, where one makes sense, and names its test ID. Otherwise it says in a few words why it has none.
		- None of the Done items name a test yet.

- CICD reset: test IDs
	- ID: 2026102416310506
	- Type: Enhancement
	- Status: Queued
	- Priority [Feature|Enhancement] | Severity [Bug]: High
	- Opened: 20261024-163105
	- Opened by: JC
	- Requirements [Feature]:
		- Sweeping override, 2026-10-24: this supersedes any earlier decision on this project that conflicts with it.
		- Every CI test gets a unique ID: its creation time in whole minutes since 2000-01-01 UTC, in lowercase Crockford base 32. Take the time from the test's first commit, or the nearest evidence. An ID never changes.
		- The run prints one `PASS`, `FAIL`, or `SKIP` line per regression and fuzz test, with its ID and name. The harness prints `ok` with a label now, and `go test ./...` counts as one check.

- CICD reset: temp files and safe removal
	- ID: 2026102416310507
	- Type: Enhancement
	- Status: Queued
	- Priority [Feature|Enhancement] | Severity [Bug]: Avg
	- Opened: 20261024-163105
	- Opened by: JC
	- Requirements [Feature]:
		- Sweeping override, 2026-10-24: this supersedes any earlier decision on this project that conflicts with it.
		- Each run makes its own folder under `${HOST_TEMP}/<project>/`, named by time and a counter, and points `TMPDIR`, `TEMP`, and `TMP` at it. The harness's scratch trees go inside it.
		- A trap removes it at the end, on failure and Ctrl+C too. `--keep-temp` keeps it and prints the path.
		- Each run also removes this project's stale run folders, older than 2 days with no live process from their `run.pid`.
		- Recursive removes go through the repo's own copy of the safe removal guard, or one that names the one parent it may remove under, such as the Makefile's `dist`. File list in the private notes.

- CICD reset: version line
	- ID: 2026102416310508
	- Type: Enhancement
	- Status: Queued
	- Priority [Feature|Enhancement] | Severity [Bug]: Avg
	- Opened: 20261024-163105
	- Opened by: JC
	- Requirements [Feature]:
		- Sweeping override, 2026-10-24: this supersedes any earlier decision on this project that conflicts with it.
		- `--version` prints `<program name> v<version> build <build>`, then the copyright line. It prints the bare `git describe` output now.
		- The build number is the commit time in minutes since 2000, in lowercase Crockford base 32, so builds stay reproducible.
		- A non-release build adds its build date after the build number. Release builds leave it off.
		- The version and the tag come from one place.

- CICD reset: macOS runner
	- ID: 2026102416310509
	- Type: Enhancement
	- Status: Queued
	- Priority [Feature|Enhancement] | Severity [Bug]: Avg
	- Opened: 20261024-163105
	- Opened by: JC
	- Target OS: macOS
	- Requirements [Feature]:
		- Sweeping override, 2026-10-24: this supersedes any earlier decision on this project that conflicts with it.
		- A hosted workflow on GitHub's free arm64 macOS runner builds, runs the tests, and smoke-tests the CLI on the built binary, without a GUI.
		- Pick which flags start it, by how long it takes.

- CICD reset: CICD process gaps
	- ID: 2026102416310510
	- Type: Enhancement
	- Status: Queued
	- Priority [Feature|Enhancement] | Severity [Bug]: Avg
	- Opened: 20261024-163105
	- Opened by: JC
	- Requirements [Feature]:
		- Sweeping override, 2026-10-24: this supersedes any earlier decision on this project that conflicts with it.
		- What's left of the shared CICD rules after the other reset items:
			- No fetch and fast-forward before the build.
			- No lint stage, no lint output under `cicd/artifacts/lint/`, and no lint `--check` gate.
			- No race detector, goleak, staticcheck, govulncheck, soak test, or CodeQL.
			- No profiler stage or flame report.
			- No FreeBSD build, no `.deb`, `.rpm`, or FreeBSD `.pkg`, no Windows installer, and no checksums file.
			- No pinned tool versions.
			- A "Proceed?" prompt in place of the commit message prompt.
			- Its own output helpers in place of the `fEcho` family.
			- No demo gif.
			- No end-of-run line saying what cleanup removed and freed.

- Rework the command line: fewer, clearer flags.
	- ID: 2026102819042523
	- Type: Enhancement
	- Status: Queued
	- Needs local test suite run?: Yes
	- Priority [Feature|Enhancement] | Severity [Bug]: High
	- Opened: 20261028-190425
	- Opened by: JC
	- Related IDs: 2026102819042524, 2026102819042525
	- Requirements [Feature]:
		- Only START is positional. The FROM and TO positionals go.
		- `--match-regex REGEX` and `--match-wild PATTERN` replace `--from` and `-F`. One or the other, not both. Wildcards are `*` and `?` and can't capture, so with `--match-wild` the replacement stands in for the whole match.
		- `--replace TEMPLATE` replaces `--to`. `$1` and `${name}` work with `--match-regex` only.
		- `--renorm rel[ative]|abs[olute]` replaces `--renormal-relative` and `--renormal-absolute`. Repeatable, so `canon` (2026102819042524) can go with either. `rel` and `abs` together is an error.
		- `--find-in[clude] REGEX`, `--find-ex[clude] REGEX`, `--find-target-in[clude] REGEX`, and `--find-target-ex[clude] REGEX` replace `--include`, `--exclude`, `--inc-target`, and `--exc-target`. Each abbreviation is a prefix of exactly one name, so the alias table goes.
		- `--re-include` and the four glob flags (`--name`, `--iname`, `--wholename`, `--iwholename`) go. Regex lookaround and `(?i)` cover them. With no widener left, rule order no longer matters. The filter package may keep globs and re-add for the file-lister project; the CLI stops exposing them.
		- `--no-nested[-filesystem]` replaces `--no-cross-device`, same st_dev check. `--xdev` stays as the one alias.
		- A `--match-wild` value that holds no wildcard and names an existing path under the current directory is an error, since the shell most likely expanded an unquoted glob. Extra positionals after it get the same message, which shows the quoted form.
		- README, design.md, `test.bash`, the fuzz targets, and `changelog.md` follow. Pre-release, so no aliases for the old spellings.

- Read and write `.lnk` shortcuts on every OS, without COM.
	- ID: 2026102819042524
	- Type: Feature
	- Status: Queued
	- Needs local test suite run?: Yes
	- Needs external testing: Yes, a smoke run on vm925w that Explorer opens a rewritten shortcut.
	- Priority [Feature|Enhancement] | Severity [Bug]: High
	- Opened: 20261028-190425
	- Opened by: JC
	- Prereq IDs: 2026102819042523
	- Target OS: All
	- Requirements [Feature]:
		- A pure Go parser and writer for the Shell Link format replaces the `go-ole` path on Windows too, so one code path runs everywhere and is testable on Linux. `go-ole` leaves `go.mod`.
		- A shortcut stores its target up to four ways, and each is handled: the LinkInfo local or network path, the relative path, the `%VAR%` environment path, and the item ID list.
		- `--match-*` rewrites the local, network, and environment paths as text, so `%USERPROFILE%\Old` to `%USERPROFILE%\New` keeps the variable. The relative path is recomputed from the absolute one after every edit, so the two never disagree.
		- Writing drops the item ID list and clears its flag, since Windows reads it before any path. Strings are written as UTF-16. Extra data blocks unrelated to the target, such as console settings and the icon path, are kept as they were. The volume serial is kept when the drive is the same.
		- Resolving a shortcut off Windows follows Windows' own order: the environment path, with `USERPROFILE` falling back to `$HOME`; then the local path, where a rooted path with no drive is tried as is, a drive path is placed by volume serial through `/dev/disk/by-uuid` and the mount table, and a UNC path is looked for among mounted SMB shares; then the relative path from the shortcut's folder. Walks are case-insensitive.
		- nemo-anywhere's `libnemo-private/nemo-lnk.c` is the model, and its test fixtures and `.lnk` fuzz corpus seed this project's, copied by reflink.
		- A new parse path gets a fuzz target, and the write path a security scenario, per the testing rule in memory.

- `--renorm canon[ical]`: resolve a target like realpath.
	- ID: 2026102819042525
	- Type: Feature
	- Status: Queued
	- Needs local test suite run?: Yes
	- Needs external testing: Yes, junctions on vm925w.
	- Priority [Feature|Enhancement] | Severity [Bug]: Avg
	- Opened: 20261028-190425
	- Opened by: JC
	- Prereq IDs: 2026102819042523
	- Related IDs: 2026102819042524
	- Target OS: All
	- Requirements [Feature]:
		- `--renorm canon` resolves every symlink in the target and writes the result absolute. With `--renorm rel` as well, it resolves first and then spells the result relative to the link's folder. `rel` and `abs` stay logical-only, as today.
		- A target that can't be resolved, such as a dangling one, fails that link with a warning, like `realpath -e`.
		- The help says plainly that this changes where a link to a link points once the middle link moves, which the logical modes never do.
		- On a `.lnk`, off Windows, the target is resolved per 2026102819042524 and written back as a rooted backslash path with no drive. On Windows it is written as a drive path.
		- On Windows, `canon` works on junctions, following further junctions and symlinks and rewriting the reparse buffer in place, which needs no elevation. `rel` on a junction warns and skips, since a junction only holds an absolute target. Symlinks still need the privilege or developer mode, and the help says so.

- Make release archives rebuild to their published checksums.
	- ID: 2026102817592630
	- Type: Enhancement
	- Status: Queued
	- Priority [Feature|Enhancement] | Severity [Bug]: Avg
	- Opened: 20261028-175926
	- Opened by: JC
	- Related IDs: 2026102416310508, 2026102416310510, 2026101301161865
	- Requirements [Feature]:
		- The cross stage builds whatever is checked out. On `dev` the release tag isn't reachable, so `--version` prints a commit hash. Release archives build from the tag.
		- Go builds pass `-buildvcs=false` and an empty `-buildid=`, and pin `GOAMD64=v1`.
		- Archive entries get a fixed mtime, owner, and order, taken from the commit time. Today each build stamps the binary with the build time.
		- The cross stage writes `SHA256SUMS` beside the archives. v1.0.0-beta.1 has one made by hand.

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

- Rename to Link Doctor and publish v1.0.0-beta.1.
	- ID: 2026102817592625
	- Type: Task
	- Status: Done
	- Priority [Feature|Enhancement] | Severity [Bug]: High
	- Opened: 20261028-175926
	- Opened by: JC
	- Requirements [Feature]:
		- Rename `repoint-symlink` to `link-doctor` everywhere: the GitHub repo, the Go module, the binary, and the docs. The official title is "Link Doctor".
		- Rename the working folder `./github` to `./repo`.
		- Merge the old feature branches and start a `dev` branch.
		- Tag v1.0.0-beta.1 and publish it as a GitHub prerelease, with notes and binaries. The download table has the OS in rows, and x86_64 and ARM64 in columns.
	- Progress log:
		- 20261028: The first archives came from `dev`, before the tag was reachable there, and reported a commit hash as their version. They were rebuilt from the tag. Follow-up is 2026102817592630.
	- Verified: All 6 archives hold the right OS and architecture, and every public download matches `SHA256SUMS`. The Linux and Windows x86_64 binaries print `v1.0.0-beta.1`.
	- Branch: rename
	- Commit: d710581
	- Test case: `cicd/test.bash` passes on the tagged tree, 58 checks.
	- Acceptance signoff: Self-closed: mechanical.
	- Closed: 20261028-175926

## Old format

### Bugs

### Features and enhancements

- Verify the Windows paths on real Windows hardware: symlink recreate (file vs dir flag, privilege), junction reparse-buffer write, and `.lnk` `TargetPath` set/save. (Because...built and cross-compiled from Linux.)
	- 20261028: The `.lnk` part is superseded by 2026102819042524, which replaces the COM path with a pure Go writer. Symlinks and junctions still need the smoke run.

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
