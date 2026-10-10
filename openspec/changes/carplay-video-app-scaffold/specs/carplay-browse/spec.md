# Spec Delta

## Purpose

Defines what a person sees and can do on the CarPlay screen: the browsing hierarchy, how items are presented, and how presentation adapts to whether video is allowed.

## ADDED Requirements

### Requirement: Root is a tab bar of catalog sections
The CarPlay root screen SHALL be a tab bar with one tab per catalog section, built only from system-provided CarPlay templates.

#### Scenario: Catalog with sections
- **WHEN** the catalog contains two or more sections
- **THEN** the root shows one tab per section, in catalog order, each opening a list of that section's items

#### Scenario: Catalog is empty
- **WHEN** the catalog loads with no items
- **THEN** the root shows a single tab with an informational list row stating that no content is available

### Requirement: Section lists show item thumbnails and metadata
Each section list SHALL present one row per item with the item's title, a thumbnail, and a one-line subtitle.

#### Scenario: Item has artwork
- **WHEN** an item provides a thumbnail image
- **THEN** the row shows that image at the aspect ratio declared by the item

#### Scenario: Item has no artwork
- **WHEN** an item provides no thumbnail image
- **THEN** the row shows a placeholder image and the row remains selectable

### Requirement: Rows carry playback configuration
Every item row SHALL carry playback configuration describing the item's preferred presentation (video or audio), its elapsed time and duration when known, and whether the next action is play, pause, or replay.

#### Scenario: Never played item
- **WHEN** an item has not been played in this session
- **THEN** its row reports zero elapsed time and a play action

#### Scenario: Partially played item
- **WHEN** an item was played to a point before its end
- **THEN** its row reports that elapsed time, the item's duration, and a play action

#### Scenario: Currently playing item
- **WHEN** an item is the one currently playing
- **THEN** its row reports the live elapsed time and a pause action

### Requirement: Selecting an item opens a details screen
Selecting a row SHALL push a details screen with a prominent header showing the item's thumbnail, title, description, playback configuration, and a primary Play action.

#### Scenario: Play from details
- **WHEN** the person activates the Play action on the details screen
- **THEN** playback of that item starts and the now playing screen is presented

#### Scenario: Return from details
- **WHEN** the person navigates back from the details screen
- **THEN** the section list is shown with its scroll position preserved

### Requirement: Presentation follows session video capability
The browsing layer SHALL present items as video content when the session permits video and as audio content when it does not, updating in place when the capability changes.

#### Scenario: Video-first presentation
- **WHEN** the session permits video
- **THEN** rows and headers declare a video presentation and the Play action starts video playback

#### Scenario: Audio-first presentation
- **WHEN** the session does not permit video
- **THEN** rows and headers declare an audio presentation and the Play action starts audio-only playback

#### Scenario: Capability changes while browsing
- **WHEN** the session's video capability changes while a list or details screen is shown
- **THEN** the visible rows and header update their presentation without the person navigating away

### Requirement: Playback configuration stays current
The browsing layer SHALL refresh the playback configuration of visible rows and headers on every playback state change so that the shown progress and action never lag the player.

#### Scenario: Pause from now playing
- **WHEN** playback is paused from the now playing screen and the person returns to the details screen
- **THEN** the header shows the paused elapsed time and a play action

#### Scenario: Item finishes
- **WHEN** the current item plays to its end
- **THEN** its row and header show full progress and a replay action

### Requirement: Template depth stays within the platform limit
The browsing hierarchy SHALL never exceed the template depth permitted for audio and video apps, counting the root.

#### Scenario: Deepest path
- **WHEN** the person navigates root, section list, details, now playing
- **THEN** no further template is pushed by the app and the hierarchy is at most five templates deep
