# Design

## Context

See proposal.md for motivation and docs/re/mini-ultra-3.md for the research. Constraints that shape this phase:

- The dongle is a black box until identified. Three unrelated boards share the name, and only the stock web interface at the hotspot address is a known, safe, read-only probe on all of them.
- Nothing in this cloud session can reach USB or Wi-Fi hardware. Tools must be testable against fixtures, and the hardware steps are a runbook the owner executes.
- Third-party tools (LIVI Link provisioning, catplay wizard) already implement the risky part: entering the bootloader, taking a backup, flashing. We should not duplicate them in phase one, only sequence and document them.
- A flash backup is 16 MiB of mixed content. Generic carving tools exist, but we want a report tuned to this device family and reproducible in CI.

## Goals / Non-Goals

**Goals:**

- Zero-risk first contact: one command tells the owner what they have and whether to continue.
- A backup analysis that turns a 16 MiB blob into a layout and inventory we can reason about and commit as text.
- A runbook that gets from box to backup and shell with the reversible path first.
- Everything testable offline with fixtures.

**Non-Goals:**

- Writing to the dongle from our tools.
- Reimplementing FEL, recovery, or flashing. Those belong to LIVI Link and catplay.
- Any firmware build, and the AirPlay or transmitter work. Those are later changes.

## Decisions

### D1: One Python package, two console entry points

**Choice.** `tools/dongle_re/` as a package with `identify.py` and `inspect_image.py` modules exposed as `python3 -m dongle_re.identify` and `python3 -m dongle_re.inspect_image`, plus a `pyproject.toml` with console scripts `dongle-id` and `fw-inspect`. Standard library only for the core; `pytest` for tests.

**Why.** Python is what catplay's and LIVI's host tooling already use, so the owner's machine will have it. Standard library keeps install friction at zero on macOS and Linux. Two modules share a small `report.py` for JSON and Markdown emission.

**Alternatives.** Rust would match catplay but is heavier for a scanning tool and slower to iterate. Shell scripts were rejected as untestable.

### D2: Identification uses a signature table, not heuristics in code

**Choice.** A `signatures.py` data table maps observable evidence (product model substrings, custom tag, platform string, host endpoint status) to variant letters and support rows. The classifier walks the table and records which rows matched. Adding a newly observed board is a table edit.

**Why.** Observed field values are already inconsistent, with Allwinner-style model names on Anyka boards and `AX1800` platform strings on non-Ingenic units. Encoding the evidence as data keeps the classification auditable and lets the report explain itself.

**Alternatives.** Hard-coded if-chains were rejected as opaque. Probing USB IDs was deferred: it needs a host USB stack and the Wi-Fi probe already suffices for A versus C.

### D3: Read-only is enforced structurally

**Choice.** The identifier has an allowlist of exactly two paths, both GET. The HTTP helper refuses any other path or method. The inspector opens its input with read-only flags and writes only inside a freshly created output directory named by the input hash.

**Why.** The project context forbids device writes in this phase. An allowlist makes that a property of the code, not of the author's care, and makes the "never probes unknown endpoints" scenario trivially true.

### D4: Image inspection is a two-pass carve

**Choice.** Pass one scans the whole image for magics with a fixed table (squashfs `hsqs`, EROFS superblock at 1024 with magic `0xE0F5E1E2`, JFFS2 `0x1985`, UBI `UBI#`, gzip, XZ, LZMA, zstd, FDT `0xD00DFEED`, RISC-V and ARM Linux image headers, U-Boot legacy header, OpenSBI banner) and reads header-carried sizes where defined. Pass two fills gaps as unknown spans and computes Shannon entropy per span. Extraction uses `unsquashfs` and `fsck.erofs --extract` if present, discovered via `shutil.which`.

**Why.** This is what the device family actually contains per catplay's recovery flow (EROFS initrd, squashfs root, OpenSBI, FDT) and ocbm's notes (jffs2 on the i.MX family). A tuned table beats a generic carver for signal and runs in CI with no dependencies. Entropy on the leftovers flags encrypted or compressed regions honestly.

**Alternatives.** Depending on binwalk was rejected: heavy install, slow, and we want a report shape we control. Writing our own squashfs or EROFS reader was rejected as scope creep.

### D5: Runbook sequences third-party tools, reversible path first

**Choice.** `docs/re/runbook.md` is ordered: identify, photograph and hash everything, LIVI Link provisioning for variants A and B (because it backs up, has a rescue system, and restores stock from its own web UI), then shell access through LIVI's rescue console, then capture of the stock backup for `fw-inspect`. catplay's wizard is documented as the alternative route with its own backup step. Every write step is marked with the brick warning and the restore path.

**Why.** LIVI Link is the only path with a documented, tested return to stock for the Ultra 3 variants. We want the first write to be the one most likely to be undoable.

### D6: Captures are text, backups are hashes

**Choice.** `re/captures/` holds JSON and Markdown from both tools. `re/backups/README.md` lists backup files by SHA-256, size, date, and where they are stored off-repo. A `.gitignore` blocks `*.bin`, `*.img`, `*.lfwb`, `*.c2aflash`.

**Why.** Backups contain Carlinkit's proprietary firmware and the unit's MFi certificate. They must not land in a public repository, but the hash lets every report be tied to a specific image.

## Risks / Trade-offs

- [Signature table is wrong for a board we have not seen] → Classification carries evidence and an explicit unknown state; the runbook says to open the case on unknown.
- [Owner holds variant C] → Identification says so before any purchase of tools or any write. Phase ends with a documented dead end and a watch on catplay's Anyka port.
- [Stock firmware changes the web interface] → The identifier records raw responses, so a changed layout is still captured for a table update.
- [Host lacks extraction tools] → Inspector degrades to layout and strings and names the missing tool.
- [Nothing here can be exercised on real hardware from this session] → Fixtures are built from the field values documented in public issues, and the owner's first real capture becomes a committed fixture.

## Migration Plan

Greenfield. Removing the change deletes `tools/`, `tests/`, `re/`, and the runbook.

## Open Questions

- Which variant the owner's unit is. Answered by running `dongle-id`. It changes the runbook branch taken, not the tooling.
- Whether the owner's host is Linux or macOS. Affects only which extraction tool package names the runbook lists.
