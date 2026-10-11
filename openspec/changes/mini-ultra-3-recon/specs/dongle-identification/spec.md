# Spec Delta

## Purpose

Identifies which hardware variant of a Carlinkit Mini Ultra class dongle is connected, from its stock web interface, without changing the device, and reports which open firmware projects support it.

## ADDED Requirements

### Requirement: Identification is read-only
The identifier SHALL only issue HTTP GET requests to version and information endpoints and SHALL NOT call any endpoint that reboots, resets, updates, or changes settings on the dongle.

#### Scenario: Normal run
- **WHEN** the identifier runs against a reachable dongle
- **THEN** the only requests sent are GETs to the version endpoint and the host information endpoint

#### Scenario: Unknown endpoint names are never probed
- **WHEN** a response suggests other CGI endpoints exist
- **THEN** the identifier does not request them

### Requirement: Version fields are parsed into a structured record
The identifier SHALL parse the plain-text version endpoint into named fields including product model, custom tag, platform, system version, protocol version, and activation state, preserving unknown fields verbatim.

#### Scenario: Well-formed response
- **WHEN** the version endpoint returns colon-separated key and value lines
- **THEN** every line becomes a field in the record and the raw text is kept alongside

#### Scenario: Fields that failed to read
- **WHEN** a value reads as a failure placeholder from the firmware
- **THEN** the field is recorded as unavailable rather than as that placeholder text

### Requirement: Variant is classified with an explicit confidence
The identifier SHALL classify the dongle as variant A, B, C, or unknown using the parsed fields and the host information endpoint's status, and SHALL state which evidence drove the classification.

#### Scenario: Variant C signature
- **WHEN** the product model contains the yunlian family marker or the host information endpoint returns not found
- **THEN** the classification is variant C with the matching evidence listed

#### Scenario: Insufficient evidence
- **WHEN** the fields do not match any known signature
- **THEN** the classification is unknown and the report says the case must be opened to read chip markings

### Requirement: Support status is reported per project
For each classification the identifier SHALL report the support status of LIVI Link, catplay, and ocbm as supported, in progress, or unsupported, with the firmware or port name where one exists.

#### Scenario: Variant A
- **WHEN** the classification is variant A
- **THEN** the report lists LIVI Link as supported with its firmware name, catplay as in progress, and ocbm as a C2Air port

#### Scenario: Variant C
- **WHEN** the classification is variant C
- **THEN** the report lists every project as unsupported and advises against any write

### Requirement: Results are saved as a dated capture
The identifier SHALL write the raw responses and the structured report to a capture file under the project's captures directory, named with the date and the system version, and SHALL print the same report to the terminal.

#### Scenario: Successful capture
- **WHEN** identification completes
- **THEN** a JSON capture exists containing raw responses, parsed fields, classification, evidence, and support status

#### Scenario: Dongle unreachable
- **WHEN** the version endpoint cannot be reached within the timeout
- **THEN** the tool exits non-zero with a message naming the expected hotspot and address and writes no capture
