# Design

## Context

See proposal.md for motivation. Constraints that shape the approach:

- The repository is empty apart from research notes and OpenSpec. There is no existing architecture to fit into.
- The CarPlay screen can only show system-rendered templates. All visual design happens through template data, never through views the app draws.
- Video reaches the car over AirPlay video. The app's job is to make its AVFoundation playback AirPlay-capable and to describe content well; the system does the rest.
- Whether video is allowed is the car's decision, read from the session and liable to change at any time.
- The planning and much of the authoring happens in a Linux cloud session with no Xcode or Swift toolchain. Build and run verification needs a Mac with Xcode 27 and the CarPlay Simulator.
- Reports from the iOS 27 beta and RC describe the video entitlement producing a blank CarPlay screen. Diagnosability of the entitlement set matters.

## Goals / Non-Goals

**Goals:**

- A single app target whose project file is generated from a text definition so it can be authored and reviewed without Xcode.
- Clear layer boundaries so the CarPlay layer, the player, and the catalog can each be replaced or tested in isolation.
- Everything that does not touch the CarPlay or AVFoundation frameworks is unit-testable on the iOS Simulator with no car attached.
- One switch to run the app with audio-only entitlements for diagnosing CarPlay connection problems.

**Non-Goals:**

- Cross-platform or Linux-buildable code. The app is iOS only.
- A reusable framework or Swift package for third parties.
- Visual design of the iPhone UI beyond what is needed to meet the AirPlay requirement.

## Decisions

### D1: Generate the Xcode project with XcodeGen

**Choice.** Describe the target, build settings, entitlements, and Info.plist in a `project.yml` and generate `CarPlayVideoPlayer.xcodeproj` with XcodeGen. The generated project is not committed.

**Why.** A `project.pbxproj` cannot be sensibly hand-authored or reviewed in a diff, and this change is written largely outside Xcode. XcodeGen is a single Homebrew install, its file is plain YAML, and it can be validated as YAML here even though generation itself runs on the Mac.

**Alternatives.** Tuist offers the same benefit with more machinery and a Swift DSL; too heavy for one target. A Swift package alone cannot carry entitlements, an Info.plist scene manifest, or an app target with the right capabilities. Committing a hand-made pbxproj was rejected for reviewability.

### D2: Four layers in one target

**Choice.** One app target with four source folders, each owning one of the four capabilities:

| Folder | Owns | Depends on |
|---|---|---|
| `Catalog/` | `CatalogItem`, `CatalogSection`, `Catalog`, `CatalogSource` protocol, `BundledCatalogSource` | Foundation only |
| `Playback/` | `PlaybackController` wrapping one `AVPlayer`, now playing info, remote commands, published `PlaybackState` | Catalog |
| `CarPlay/` | `CarPlaySceneDelegate`, `SessionCapability` observer, `TemplateBuilder`, `PlaybackConfigurationMapper` | Catalog, Playback |
| `Phone/` | SwiftUI `App`, catalog list, player screen | Catalog, Playback |

**Why.** The spec capabilities map one-to-one onto folders, so a reviewer can find the code for a requirement by name. Keeping it one target avoids module boundaries and access-control churn in a scaffold.

**Alternatives.** Splitting `Catalog` and `Playback` into a local Swift package would allow `swift test` on Linux for the catalog, but `Playback` needs AVFoundation and MediaPlayer, so only a sliver would benefit. Not worth the second build graph yet. Revisit when a real content source arrives.

### D3: PlaybackController is the single source of truth

**Choice.** A `@MainActor @Observable final class PlaybackController` owns the only `AVPlayer`. It exposes `state` (`idle`, `playing`, `paused`, `ended`, `failed(reason)`), `currentItem`, `elapsed`, `duration`, and `videoPermitted`. It installs a periodic time observer at one-second cadence, mirrors state into `MPNowPlayingInfoCenter`, and registers `MPRemoteCommandCenter` handlers for play, pause, toggle, and next or previous track. The player is created with `allowsExternalPlayback = true` and `usesExternalPlaybackWhileExternalScreenIsActive = true`.

**Why.** Both the phone UI and the CarPlay templates observe one object, which is the only way to meet the shared-state requirement without a sync layer. External playback flags are what make the AirPlay video path available.

**Alternatives.** Separate players per surface were rejected because the specs require identical state on both. `AVQueuePlayer` is unnecessary without playlists.

### D4: Capability detection via CPSessionConfiguration, pushed to observers

**Choice.** The scene delegate creates a `CPSessionConfiguration` on connect, reads `supportsVideoPlayback`, and holds it for the connection's lifetime. A small `SessionCapability` observable publishes `videoPermitted`. The template builder and `PlaybackController` observe it. On every delegate callback from the session configuration, and on each scene foreground, the value is re-read and published if changed.

**Why.** The specs require reacting to mid-session changes. Re-reading on known delegate events plus scene lifecycle is cheap and covers the cases the SDK exposes. See Open Questions for the exact change callback.

**Alternatives.** Polling on a timer was rejected as wasteful and imprecise. Reading once at connect fails the mid-session scenarios.

### D5: Templates built by a pure builder, configuration mapped by a pure mapper

**Choice.** `TemplateBuilder` turns a `Catalog` plus `SessionCapability` into a `CPTabBarTemplate` of `CPListTemplate`s and builds the details template for an item. `PlaybackConfigurationMapper` turns `(CatalogItem, PlaybackState, videoPermitted)` into a `CPPlaybackConfiguration` with `preferredPresentation` of `.video` or `.audio`, elapsed, duration, and action play, pause, or replay. Both are stateless and take their inputs as values.

**Why.** CarPlay template classes can be instantiated in a Simulator unit test without a car, so pure builders make the browsing capability testable: row counts, titles, placeholder images, depth, and configuration values are all assertable. The mapper is the single place the progress and action rules live, which the specs require to stay current everywhere.

**Alternatives.** Building templates inside the scene delegate was rejected as untestable. Caching templates across capability changes was rejected; rebuilding the row data in place is simpler and the lists are small.

### D6: Details screen uses the iOS 27 details header

**Choice.** The details template is a list template carrying the iOS 27 details header element with thumbnail, title, body, playback configuration, and a Play action button as its first action, so the system combines Play with the progress display.

**Why.** This is the presentation Apple designed for video apps and the one the Simulator's video-capable configuration renders.

**Alternatives.** A plain list row with a Play item was rejected; it does not show progress or presentation.

### D7: Entitlement set chosen by build configuration

**Choice.** Two entitlement files: `CarPlayVideoPlayer.entitlements` with both `com.apple.developer.carplay-video` and `com.apple.developer.carplay-audio`, and `CarPlayVideoPlayer-AudioOnly.entitlements` with only the audio key. `project.yml` defines a `Debug-AudioOnly` configuration and matching scheme that points `CODE_SIGN_ENTITLEMENTS` at the second file.

**Why.** The blank-screen reports mean the first thing to try when CarPlay shows nothing is removing the video key. A scheme switch does that without editing files.

**Alternatives.** A single file edited by hand was rejected as error-prone and easy to commit by accident.

### D8: Catalog is a bundled JSON file behind a protocol

**Choice.** `CatalogSource` is `protocol CatalogSource { func load() async throws -> Catalog }`. `BundledCatalogSource` decodes `SampleCatalog.json` from the main bundle with a tolerant decoder that collects per-item diagnostics and skips invalid items. Tests use an `InMemoryCatalogSource`.

**Why.** The protocol is the seam the specs call for. JSON in the bundle keeps the scaffold free of networking while still exercising real decoding and error paths.

**Alternatives.** Hardcoded Swift literals were rejected because they bypass the decoding and error handling the catalog spec requires.

### D9: iPhone UI uses AVPlayerViewController

**Choice.** The phone player screen wraps `AVPlayerViewController` bound to the shared player. It supplies the AirPlay route picker and external playback UI for free.

**Why.** Meets the AirPlay requirement with no custom controls. The phone UI is explicitly not a product surface in this change.

**Alternatives.** A SwiftUI `VideoPlayer` view lacks the route picker; custom controls are out of scope.

## Risks / Trade-offs

- [No Xcode in the authoring environment, so nothing is compiled here] → Keep Swift straightforward and framework-conventional, validate `project.yml` and `SampleCatalog.json` syntactically here, and make the first apply task on the Mac a clean `xcodegen generate` and build. Tasks state which verifications run on the Mac.
- [Video entitlement blank-screen bug in iOS 27 builds] → D7 scheme switch for diagnosis, plus the README documents filing Feedback with the reproduction.
- [Exact iOS 27 API names for the details header and playback configuration attachment points may differ from the WWDC description] → The builder and mapper isolate those calls to two files; correcting a name touches nothing else.
- [Sample stream hosts may go away] → URLs live only in `SampleCatalog.json`; the loader tolerates failures per item and the player reports a failed state instead of crashing.
- [AirPlay handoff from a CarPlay Play action is system-driven and under-documented] → Set the external playback flags per D3 and verify in the Simulator's video configuration; if manual route selection is needed, the phone route picker is already present.
- [Template depth limit of five] → The deepest path is root, list, details, now playing, which is four. The builder test asserts this.

## Migration Plan

Greenfield. Nothing to migrate or roll back. Removing the change means deleting the source tree and project definition.

## Open Questions

- Which `CPSessionConfigurationDelegate` callback, if any, fires when `supportsVideoPlayback` changes in iOS 27. D4 re-reads on every delegate callback and on scene foreground regardless, so the answer refines but does not change the approach.
- Whether the Simulator's video-capable configuration routes video automatically when Play is chosen on the CarPlay details screen, or requires picking the route on the phone. Affects README wording only.
