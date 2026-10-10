# Spec Delta

## Purpose

Defines the media model the browsing and playback layers consume, and how the catalog is loaded from the bundled sample source with a seam for future sources.

## ADDED Requirements

### Requirement: Catalog is organised into sections of items
A catalog SHALL consist of an ordered list of sections, each with a title and an ordered list of items. Section and item order SHALL be preserved as loaded.

#### Scenario: Load sample catalog
- **WHEN** the bundled sample catalog is loaded
- **THEN** sections and items are returned in the order they appear in the source

### Requirement: Item carries what browsing and playback need
Each item SHALL have a stable identifier, a title, a subtitle, a description, a media URL, a media kind of video or audio, and an optional thumbnail URL with an aspect ratio of portrait or landscape. Duration MAY be absent until playback discovers it.

#### Scenario: Complete item
- **WHEN** an item in the source provides every field
- **THEN** all fields are available to consumers unchanged

#### Scenario: Item without thumbnail
- **WHEN** an item in the source omits the thumbnail
- **THEN** the item loads with no thumbnail and no aspect ratio, and consumers treat it as having a placeholder

### Requirement: Invalid items are skipped, not fatal
The catalog loader SHALL skip an item that lacks an identifier, title, or media URL, SHALL record a diagnostic for it, and SHALL still return the remaining valid items.

#### Scenario: One malformed item
- **WHEN** the source contains one item with no media URL among valid items
- **THEN** the returned catalog omits that item, keeps the others, and the diagnostic names the skipped identifier or index

#### Scenario: Unparseable source
- **WHEN** the source cannot be parsed at all
- **THEN** the loader reports a load error and returns an empty catalog rather than crashing

### Requirement: Catalog source is replaceable
The catalog SHALL be obtained through a source abstraction so that the bundled sample source can be replaced by another source without changing the browsing or playback layers.

#### Scenario: Swap source in tests
- **WHEN** a test supplies an in-memory source
- **THEN** the browsing layer builds its hierarchy from that source with no other change

### Requirement: Sample catalog is demonstrable offline from a backend
The bundled sample catalog SHALL reference only publicly hosted streams that require no authentication, SHALL include at least one video item and one audio item, and SHALL include at least two sections.

#### Scenario: First run in the CarPlay Simulator
- **WHEN** the app launches in the CarPlay Simulator with no network configuration beyond internet access
- **THEN** the root shows at least two tabs and a video item can be started
