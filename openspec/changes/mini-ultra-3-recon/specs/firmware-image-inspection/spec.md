# Spec Delta

## Purpose

Analyses a raw flash backup image of a dongle read-only, reporting its layout, embedded versions, and file inventory so that later firmware work starts from facts rather than guesses.

## ADDED Requirements

### Requirement: Input is never modified
The inspector SHALL open the image read-only and SHALL write only to a separate output directory it creates.

#### Scenario: Run against a backup
- **WHEN** the inspector runs on an image file
- **THEN** the image's bytes and modification time are unchanged afterwards

### Requirement: Known regions are located by magic
The inspector SHALL scan the image for the signatures of squashfs, EROFS, JFFS2, UBI, gzip, LZMA, XZ, zstd, flattened device tree, Linux kernel images for RISC-V and ARM, and U-Boot or OpenSBI headers, and SHALL report each hit with its offset and, where the header carries it, size.

#### Scenario: Image with a squashfs root
- **WHEN** the image contains a squashfs superblock
- **THEN** the report lists it with offset, byte size from the superblock, and compression type

#### Scenario: Device tree present
- **WHEN** the image contains a flattened device tree
- **THEN** the report lists its offset and, if extractable, the model and compatible strings

### Requirement: Version and vendor strings are extracted
The inspector SHALL collect printable strings that look like firmware versions, product models, kernel version banners, and known vendor markers, and SHALL list them with offsets.

#### Scenario: Stock firmware image
- **WHEN** the image carries the stock web interface and its version data
- **THEN** the report includes the system version and product model strings with offsets

### Requirement: Filesystems are inventoried when host tools allow
When the host has the matching extraction tool, the inspector SHALL extract each squashfs or EROFS region into the output directory and produce a file inventory with sizes and a shortlist of executables, shared libraries, init scripts, and web interface scripts. When the tool is missing it SHALL say so and skip extraction without failing the run.

#### Scenario: squashfs with host tool present
- **WHEN** a squashfs region is found and the extraction tool is installed
- **THEN** the inventory lists the files and flags the CarPlay stack binaries and the CGI scripts

#### Scenario: Host tool missing
- **WHEN** a region is found and the extraction tool is not installed
- **THEN** the report names the tool to install and the run still completes

### Requirement: Unknown regions are reported, not interpreted
Bytes not covered by a recognised region SHALL be reported as unknown spans with offset, length, and an entropy estimate, with no further claims.

#### Scenario: Encrypted or compressed blob
- **WHEN** a span has high entropy and no known magic
- **THEN** it is listed as unknown high-entropy with its offset and length

### Requirement: Report is machine-readable and human-readable
The inspector SHALL write a JSON report and a Markdown summary into the output directory and SHALL record the input's SHA-256 so captures can be matched to a specific backup without committing the backup.

#### Scenario: Completed run
- **WHEN** inspection finishes
- **THEN** the output directory contains the JSON report, the Markdown summary, and the input hash
