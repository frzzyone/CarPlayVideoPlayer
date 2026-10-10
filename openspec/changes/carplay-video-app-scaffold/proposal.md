# Proposal

## Why

The repository has research on how video reaches a CarPlay screen but no code. iOS 27 opened an official CarPlay video app category, and the only App Store legal route is a CarPlay-framework browsing UI paired with AirPlay video playback. This change lays down the runnable app shell so every later feature (real content sources, accounts, playlists) builds on a correctly entitled, correctly structured foundation instead of retrofitting one.

## What Changes

- Add an Xcode-buildable iOS app target, `CarPlayVideoPlayer`, with a reproducible project definition, Swift as the only language, and an iOS 27.0 minimum deployment target.
- Declare the CarPlay video and CarPlay audio entitlements together so the app is listed in CarPlay on every head unit, video capable or not.
- Register a CarPlay template application scene and keep a connection lifecycle that survives connect, disconnect, and reconnect.
- Detect whether the connected session allows video and switch the CarPlay UI between a video-first and an audio-first presentation, at connect time and whenever the car changes its answer.
- Provide a CarPlay browsing UI built only from CarPlay templates: a tab bar root, list templates with thumbnails, and a details header with a Play action, each carrying playback configuration (progress, duration, play or pause state, video or audio presentation).
- Provide an AVFoundation-based player that supports AirPlay video streaming, publishes playback state, and feeds the CarPlay now playing template.
- Ship a bundled sample catalog of publicly hosted HLS streams so the scaffold is demonstrable in the CarPlay Simulator without any backend. Real content sources are out of scope.
- Provide a minimal iPhone UI that lists the catalog and plays an item with the system player, which exposes the AirPlay route picker. The phone UI exists only so the AirPlay requirement is met; it is not a designed product surface.
- Add a README section covering entitlement request, CarPlay Simulator setup with the video-capable vehicle configuration, and how to build.

Not in scope: remote content APIs, authentication, playlists, search, download or offline, analytics, App Store submission assets, and any non-template drawing on the CarPlay screen.

## Capabilities

### New Capabilities

- `carplay-session`: how the app joins a CarPlay session. Entitlement pairing, scene connection and disconnection, detection of video playback support, and reaction to the car changing that support mid-session.
- `carplay-browse`: what the person sees and can do on the CarPlay screen. Template hierarchy, list and thumbnail content, details header, video-first versus audio-first presentation, and playback configuration on browsable items.
- `video-playback`: how media plays. Player lifecycle, AirPlay video routing, playback state publication, now playing integration, and audio-only continuation when video becomes unavailable.
- `content-catalog`: the media model the other capabilities consume. Item shape, catalog loading from the bundled sample source, and the seam for future sources.

### Modified Capabilities

None. The project has no existing specs.

## Impact

- New files: project definition, `CarPlayVideoPlayer/` source tree, entitlements file, Info.plist scene configuration, bundled catalog JSON, unit test target, README.
- New external dependency: the project definition tool used to generate the Xcode project. It is a development-time tool only; the shipped app has no third-party runtime dependencies.
- Apple account dependency: the CarPlay video and audio entitlements must be granted by Apple before the app runs on a device or in the CarPlay Simulator with the real provisioning profile. Until then the app still builds and the phone UI runs.
- Hardware dependency: the CarPlay video UI only appears on a head unit that advertises video support, or in the CarPlay Simulator's video-capable configuration. Nothing in this change can make an existing non-video head unit show video.
- Known risk: iOS 27 beta and RC forum reports describe the video entitlement producing a blank CarPlay screen with no scene callbacks. The design must make the entitlement easy to toggle so this can be diagnosed.
