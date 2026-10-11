# Proposal

## Why

We want a Carlinkit Mini Ultra 3 to run firmware we control so it can accept AirPlay mirroring and forward it to the car's wired CarPlay port. Carlinkit ships at least three unrelated boards under that name, only two have any open foothold, and every write to the flash risks a brick. Before touching firmware we need to know exactly which board we hold, hold a verified backup, and understand what is on it. This change delivers the read-only reconnaissance tooling and the documented hardware procedure that every later phase depends on.

## What Changes

- Add a Python command-line tool, `dongle-id`, that queries the dongle's vendor web interface over its hotspot, parses the version endpoints, and classifies the unit as variant A (Allwinner V821B + AIC8800D80), B (Axera AX520 + AIC8800D80), C (Anyka or V851SE "yunlian"), or unknown, and reports which open projects support it. It never calls any endpoint that changes state.
- Add a Python command-line tool, `fw-inspect`, that takes a raw flash backup image and reports its layout: filesystem and image magics with offsets, embedded version strings, kernel and device tree hints, and an inventory of files in any squashfs or EROFS region it can extract with host tools. It never modifies the input.
- Add a hardware runbook, `docs/re/runbook.md`, that walks the owner from an unopened dongle to a verified backup and a shell, using LIVI Link's provisioning tool for variants A and B as the reversible path, with catplay's wizard as the alternative, and records what to capture at each step.
- Add a `re/` directory for captured artifacts (version dumps, backups are listed by hash only, never committed) with a README describing the layout and the no-binaries rule.
- Add a Python project definition and tests so both tools run with `python3 -m` and are covered by unit tests against fixtures.

Not in scope: writing anything to the dongle, building or flashing firmware, the AirPlay receiver, the CarPlay transmitter integration, and support for variant C beyond identifying it.

## Capabilities

### New Capabilities

- `dongle-identification`: how a connected Mini Ultra class dongle is identified and classified from its stock web interface without changing its state, and how support status is reported.
- `firmware-image-inspection`: how a raw flash backup is analysed read-only into a layout report and file inventory, and how unrecognised content is reported rather than guessed.

### Modified Capabilities

None. The project has no existing specs.

## Impact

- New files under `tools/` (Python package), `tests/`, `docs/re/`, `re/`.
- New development dependencies: Python 3.10+, `requests` optional (standard library `urllib` preferred), `squashfs-tools` and `erofs-utils` on the host for extraction, both optional with graceful degradation.
- No change to the dongle. The tools are read-only by design, and the runbook's write steps are performed by the owner with third-party tools that take their own backups.
- Owner dependencies: a Mini Ultra 3 in hand, a Linux or macOS computer with USB, and willingness to accept brick risk at the runbook's clearly marked first write step.
- Known risk: variant C is a dead end for now. The identification tool exists precisely so we learn that before buying tools or opening the case.
