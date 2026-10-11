# Tasks

## 1. Project skeleton

- [ ] 1.1 Create `tools/dongle_re/` package with `__init__.py`, `report.py` (JSON and Markdown writers, SHA-256 helper), and `pyproject.toml` declaring the `dongle-id` and `fw-inspect` console scripts and a `pytest` dev dependency. Verify with `python3 -m pytest` discovering zero tests and `python3 -c "import dongle_re"` succeeding.
- [ ] 1.2 Create `re/captures/.gitkeep`, `re/backups/README.md` describing the hash-only rule, and extend `.gitignore` with `*.bin`, `*.img`, `*.lfwb`, `*.c2aflash`, `tools/**/__pycache__`. Verify `git status` ignores a dummy `re/backups/test.bin`.

## 2. Dongle identification

- [ ] 2.1 Implement `signatures.py` with the variant table (A: V821B + AIC8800D80; B: AX520 + AIC8800D80; C: yunlian family or host endpoint 404; unknown) and per-variant support rows for LIVI Link, catplay, ocbm. Verify with a unit test asserting each documented field set from the public issues classifies to the expected variant with the expected evidence list.
- [ ] 2.2 Implement the read-only HTTP helper with a two-path GET allowlist and a timeout. Verify with a unit test that any other path or method raises before any socket is opened.
- [ ] 2.3 Implement the version parser: colon-separated lines to fields, failure placeholders to unavailable, raw text retained. Verify with unit tests on the three documented response shapes including the one with failed reads.
- [ ] 2.4 Implement `identify.py` main: probe, parse, classify, support rows, dated JSON capture under `re/captures/`, terminal report, non-zero exit with hotspot and address guidance when unreachable. Verify with a test using a local HTTP server fixture for each variant and one for the unreachable case, asserting capture contents and exit code.

## 3. Firmware image inspection

- [ ] 3.1 Implement the magic table and pass-one scanner with header size reads for squashfs, EROFS, JFFS2, UBI, gzip, XZ, LZMA, zstd, FDT, RISC-V and ARM Linux image headers, U-Boot legacy header, OpenSBI banner. Verify with a unit test on a synthetic image assembled from minimal valid headers that every region is found at the right offset with the right size where applicable.
- [ ] 3.2 Implement pass two: gap detection into unknown spans with Shannon entropy. Verify with a unit test that a random span reports high entropy and a zero-filled span reports low.
- [ ] 3.3 Implement string extraction for version banners, product model and custom tags, kernel version strings, and known vendor markers, with offsets. Verify with a unit test on a synthetic image containing planted strings.
- [ ] 3.4 Implement optional extraction via `unsquashfs` and `fsck.erofs` when present, producing a file inventory with sizes and flagged categories (executables, shared libraries, init scripts, CGI scripts). Verify with a unit test that builds a tiny squashfs via `mksquashfs` when available and skips with a clear message when not, and that a missing tool yields a named recommendation without failing.
- [ ] 3.5 Implement `inspect_image.py` main: read-only open, output directory named by input hash, JSON report, Markdown summary, input hash file. Verify with an end-to-end test on the synthetic image asserting the input file's bytes and mtime are unchanged and all three outputs exist.

## 4. Runbook and documentation

- [ ] 4.1 Write `docs/re/runbook.md`: identify, photograph and record markings, run `dongle-id`, decision table by variant, LIVI Link provisioning with backup location and rescue console, shell access and what to capture (version files, partition table, mounted filesystems, process list, listening ports), copying the stock backup off the device, running `fw-inspect`, catplay wizard as the alternative route, restore-to-stock steps, and the brick warning before every write. Verify by a second read-through checking every command references a tool documented in its own project's README.
- [ ] 4.2 Add a top-level `README.md` section pointing to the dossier, runbook, and both tools with their one-line usage. Verify the documented commands run as written against the synthetic fixture.

## 5. First real capture (owner, hardware)

- [ ] 5.1 Owner runs `dongle-id` against the real unit and commits the capture JSON. Verify the classification matches the chip markings read through the case or after opening.
- [ ] 5.2 If variant A or B: owner follows the runbook to a LIVI Link backup and shell, records the shell captures under `re/captures/`, and runs `fw-inspect` on the stock backup, committing the JSON and Markdown report but not the image. Verify the inspect report lists the stock CarPlay stack binaries and the CGI scripts named in the dossier.
- [ ] 5.3 Promote the real capture and the real inspect report into test fixtures so the signature table and magic table are checked against ground truth. Verify the test suite passes with them included.

## Workflow follow-up

- Archive once 5.2 is complete or, on variant C, once 5.1 has recorded the dead end. The next change plans firmware work from the inspect report.
