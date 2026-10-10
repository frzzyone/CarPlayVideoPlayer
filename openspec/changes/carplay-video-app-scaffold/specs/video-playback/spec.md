# Spec Delta

## Purpose

Defines how media plays: starting and controlling playback, routing video to the car over AirPlay, publishing state to the rest of the app, and continuing as audio when video is unavailable.

## ADDED Requirements

### Requirement: Single shared player
The app SHALL use one player instance shared by the iPhone UI and the CarPlay UI so that both surfaces reflect the same item, position, and state.

#### Scenario: Start on CarPlay, observe on iPhone
- **WHEN** playback is started from the CarPlay details screen
- **THEN** the iPhone UI shows the same item as playing with the same position

#### Scenario: Start on iPhone, observe on CarPlay
- **WHEN** playback is started from the iPhone UI while CarPlay is connected
- **THEN** the CarPlay now playing screen shows the same item

### Requirement: Playback supports AirPlay video streaming
Playback of a video item SHALL be routable to an AirPlay video receiver, including a CarPlay head unit that supports video, and the iPhone UI SHALL offer the system route picker.

#### Scenario: Car display available as route
- **WHEN** a video item is playing and the iPhone is connected to a head unit that supports video
- **THEN** the head unit is offered as a destination in the system route picker

#### Scenario: Route selected
- **WHEN** the person selects the head unit as the route
- **THEN** video and audio play on the head unit and the iPhone shows the item as playing externally

### Requirement: Playback state is published
The player SHALL publish its current item, elapsed time, duration, and a state of idle, playing, paused, or ended, and SHALL emit an update on every change of those values.

#### Scenario: Time progresses
- **WHEN** playback is running
- **THEN** elapsed time updates are emitted at least once per second

#### Scenario: Item ends
- **WHEN** the current item reaches its end
- **THEN** the state becomes ended and elapsed time equals duration

### Requirement: Now playing information is kept current
The player SHALL keep the system now playing information (title, artwork, elapsed time, duration, playback rate) current so that the CarPlay now playing screen and lock screen show accurate data.

#### Scenario: Pause
- **WHEN** playback is paused
- **THEN** the now playing information reports a zero playback rate and the paused elapsed time

#### Scenario: Seek
- **WHEN** the elapsed time is changed by a seek
- **THEN** the now playing information reports the new elapsed time within one second

### Requirement: Remote commands are honoured
The player SHALL respond to system play, pause, toggle, and skip commands issued from CarPlay or other remote controls.

#### Scenario: Pause from steering wheel control
- **WHEN** a pause command arrives from the vehicle
- **THEN** playback pauses and the published state becomes paused

### Requirement: Video unavailable continues as audio
When the session reports that video is not permitted, the player SHALL continue or start playback of the current item with audio only and SHALL NOT stop playback.

#### Scenario: Video revoked during playback
- **WHEN** a video item is playing and the session stops permitting video
- **THEN** the audio keeps playing without interruption and the published presentation becomes audio

#### Scenario: Start while video not permitted
- **WHEN** an item is started while the session does not permit video
- **THEN** playback starts as audio only

### Requirement: Playback errors are reported, not swallowed
The player SHALL publish a failed state with a human-readable reason when an item cannot be loaded or played, and SHALL leave the previous item's position untouched.

#### Scenario: Unreachable stream
- **WHEN** an item's media URL cannot be loaded
- **THEN** the state becomes failed with a reason and the browsing layer can show it to the person
