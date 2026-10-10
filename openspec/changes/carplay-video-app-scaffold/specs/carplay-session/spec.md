# Spec Delta

## Purpose

Defines how the app joins, maintains, and leaves a CarPlay session, and how it learns whether the connected car permits video playback.

## ADDED Requirements

### Requirement: App is listed in CarPlay on every head unit
The app SHALL declare both the CarPlay video and the CarPlay audio app categories so that it appears on the CarPlay home screen whether or not the head unit supports video playback.

#### Scenario: Head unit supports video
- **WHEN** the iPhone connects to a head unit that advertises video playback support
- **THEN** the app icon appears on the CarPlay home screen

#### Scenario: Head unit does not support video
- **WHEN** the iPhone connects to a head unit that does not advertise video playback support
- **THEN** the app icon still appears on the CarPlay home screen

### Requirement: CarPlay scene connects and presents a root
The app SHALL accept a CarPlay scene connection and present a root browsing screen within the connection callback, without requiring the iPhone app to be in the foreground.

#### Scenario: Connect while iPhone app is backgrounded
- **WHEN** CarPlay connects and the iPhone app is suspended or not running
- **THEN** the app launches into the CarPlay scene and shows the root browsing screen

#### Scenario: Connect while iPhone app is in the foreground
- **WHEN** CarPlay connects and the iPhone app is already on screen
- **THEN** the CarPlay root browsing screen appears and the iPhone UI keeps working independently

### Requirement: Scene disconnection releases CarPlay state
The app SHALL release all CarPlay interface references when the scene disconnects and SHALL be able to accept a new connection afterwards.

#### Scenario: Unplug and replug
- **WHEN** the CarPlay connection drops and is re-established during one app lifetime
- **THEN** the root browsing screen is presented again and no stale templates or controllers from the previous connection are reused

#### Scenario: Playback continues through disconnect
- **WHEN** media is playing and the CarPlay connection drops
- **THEN** playback on the iPhone is unaffected by the disconnect

### Requirement: Video capability is determined at connect time
The app SHALL read the session's video playback capability when the scene connects and SHALL expose it to the browsing layer before the root screen is built.

#### Scenario: Video-capable session
- **WHEN** the connected session reports that video playback is supported
- **THEN** the browsing layer is told to use the video-first presentation

#### Scenario: Audio-only session
- **WHEN** the connected session reports that video playback is not supported
- **THEN** the browsing layer is told to use the audio-first presentation

### Requirement: Capability changes are propagated during a session
The app SHALL observe changes to the session's video playback capability for the life of the connection and SHALL notify the browsing and playback layers on each change.

#### Scenario: Car revokes video mid-session
- **WHEN** the car stops permitting video while the app is connected
- **THEN** the browsing layer switches to the audio-first presentation and the playback layer is told video is unavailable

#### Scenario: Car grants video mid-session
- **WHEN** the car starts permitting video while the app is connected
- **THEN** the browsing layer switches to the video-first presentation

### Requirement: No app-level driving lock
The app SHALL NOT implement its own parked, gear, or motion gating for video. It SHALL rely solely on the capability reported by the session.

#### Scenario: Vehicle starts moving
- **WHEN** the vehicle begins to move while video is playing
- **THEN** the app takes no action of its own and only responds if the session capability changes
