# Tasks

## 1. Project definition

- [ ] 1.1 Write `project.yml` for XcodeGen with one iOS app target `CarPlayVideoPlayer` (Swift, iOS 27.0 deployment target, bundle id placeholder), a `CarPlayVideoPlayerTests` unit test target, and `Debug`, `Debug-AudioOnly`, `Release` configurations. Verify here by parsing it as YAML; verify on the Mac with `xcodegen generate` succeeding.
- [ ] 1.2 Add `CarPlayVideoPlayer.entitlements` (video and audio keys) and `CarPlayVideoPlayer-AudioOnly.entitlements` (audio key only), wired to the configurations via `CODE_SIGN_ENTITLEMENTS`. Verify by inspecting the generated target's build settings per configuration on the Mac.
- [ ] 1.3 Add `Info.plist` with the CarPlay template application scene manifest and background audio mode. Verify by `plutil -lint` on the Mac and by the CarPlay scene connecting in the Simulator in task 6.1.
- [ ] 1.4 Add `.gitignore` covering the generated `.xcodeproj`, `DerivedData`, and `xcuserdata`. Verify `git status` shows no generated files after `xcodegen generate` on the Mac.

## 2. Catalog layer

- [ ] 2.1 Implement `CatalogItem`, `CatalogSection`, `Catalog` value types with `Codable`, including `MediaKind` and `ThumbnailAspect` enums and optional thumbnail and duration. Verify with a unit test that round-trips a full item and an item without thumbnail.
- [ ] 2.2 Implement `CatalogSource` protocol and `InMemoryCatalogSource`. Verify with a unit test that a consumer receives sections in supplied order.
- [ ] 2.3 Implement `BundledCatalogSource` with a tolerant decoder that skips items missing id, title, or media URL, records diagnostics, and reports an error for unparseable input. Verify with unit tests for one malformed item among valid ones and for garbage input.
- [ ] 2.4 Add `SampleCatalog.json` with at least two sections, at least one video and one audio item, all public unauthenticated HLS URLs. Verify here that the JSON parses and every URL responds to a HEAD or GET request; verify on the Mac that the bundled loader returns every item with no diagnostics.

## 3. Playback layer

- [ ] 3.1 Implement `PlaybackState` and `PlaybackController` owning one `AVPlayer` with external playback enabled, `play(item:)`, `pause()`, `resume()`, `seek(to:)`, and a one-second periodic time observer publishing elapsed and duration. Verify with a unit test using a local bundled audio fixture that state moves idle to playing to paused and elapsed advances.
- [ ] 3.2 Mirror state into `MPNowPlayingInfoCenter` on every state change and register `MPRemoteCommandCenter` handlers for play, pause, toggle, next, and previous. Verify with a unit test that pausing sets playback rate zero in the now playing dictionary and that invoking the registered pause handler pauses the player.
- [ ] 3.3 Implement `videoPermitted` handling: when false, playback continues as audio and published presentation becomes audio; starting an item while false starts audio only. Verify with a unit test that toggling `videoPermitted` during playback does not change `state` from playing.
- [ ] 3.4 Publish `failed(reason)` when an item cannot load, leaving the previous item's elapsed time untouched. Verify with a unit test using an unreachable URL.

## 4. CarPlay layer

- [ ] 4.1 Implement `SessionCapability` observable and `CarPlaySceneDelegate` that creates `CPSessionConfiguration` on connect, publishes `supportsVideoPlayback`, re-reads it on every configuration delegate callback and scene foreground, presents the root template, and releases interface controller and configuration on disconnect. Verify on the Mac by connecting, disconnecting, and reconnecting in the CarPlay Simulator with the root appearing each time and no retained controller (assert nil after disconnect in a debug log).
- [ ] 4.2 Implement `PlaybackConfigurationMapper` producing `CPPlaybackConfiguration` from item, playback state, and `videoPermitted`: never played gives zero elapsed and play; partially played gives stored elapsed and play; playing gives live elapsed and pause; ended gives full progress and replay; presentation follows `videoPermitted`. Verify with a unit test per rule.
- [ ] 4.3 Implement `TemplateBuilder.rootTemplate(catalog:capability:)` producing a tab bar with one list template per section, placeholder image for items without thumbnail, and an informational single-tab root for an empty catalog. Verify with unit tests asserting tab count, row titles, placeholder presence, and empty-catalog row text.
- [ ] 4.4 Implement `TemplateBuilder.detailsTemplate(item:...)` using the iOS 27 details header with thumbnail, title, body, playback configuration, and Play as the first action, whose handler starts playback and pushes `CPNowPlayingTemplate.shared`. Verify with a unit test that the header's first action title is Play and that the pushed hierarchy depth is at most four including now playing.
- [ ] 4.5 Wire observation so visible rows and the details header refresh their playback configuration on every `PlaybackController` change and on every `SessionCapability` change without navigation. Verify on the Mac in the Simulator by pausing from now playing, going back, and seeing the header show play with the paused time; and by toggling the Simulator's video capability while a list is shown.

## 5. Phone layer

- [ ] 5.1 Implement the SwiftUI `App` with the scene configuration selector returning the CarPlay scene for CarPlay sessions and the window scene otherwise, and inject one shared `PlaybackController` and catalog. Verify on the Mac that launching with and without the Simulator connected lands in the right scene.
- [ ] 5.2 Implement a catalog list view and a player view wrapping `AVPlayerViewController` bound to the shared player. Verify on the Mac that selecting an item plays it and the AirPlay route picker is visible in the player controls.

## 6. Integration and documentation

- [ ] 6.1 End to end in the CarPlay Simulator video-capable configuration on the Mac: root tabs appear, a video item's details screen shows the header with Play, Play starts playback and shows now playing, and now playing shows title and artwork. Record the observed AirPlay handoff behavior for the README.
- [ ] 6.2 Run the app under the `Debug-AudioOnly` scheme and verify the root still appears in a non-video Simulator configuration with audio presentation on all rows.
- [ ] 6.3 Write `README.md` covering prerequisites (Xcode 27, XcodeGen, CarPlay Simulator from Additional Tools for Xcode), entitlement request at the Apple CarPlay portal, generating and building the project, choosing the video-capable Simulator configuration, the `Debug-AudioOnly` diagnostic scheme, and the observed handoff behavior from 6.1. Verify by following the README from a clean clone on the Mac.

## Workflow follow-up

- Archive the change once the Mac verifications in groups 4 to 6 have passed and the README matches observed behavior.
