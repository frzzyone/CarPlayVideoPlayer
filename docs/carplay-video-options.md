# Streaming video to a CarPlay screen: options (October 2026)

## The short version

Until 2026 Apple blocked all video on CarPlay. That changed in two steps:

| Release | What it added |
|---|---|
| iOS 26.4 (spring 2026) | AirPlay video to a CarPlay head unit while parked. Any iPhone app that already supports AirPlay video can send video to a car that supports "video in car". |
| iOS 27 (September 2026) | A new CarPlay app category, **CarPlay video app**, with its own entitlement and a browsing UI on the car screen. |

The catch is the vehicle. The head unit must advertise video support to the iPhone, and as of the
latest reporting no automaker has publicly shipped a production car that does. An iOS update alone
does not make an existing head unit video capable. So before writing any code, confirm what the
target screen is, because that decides which option below is realistic.

## Option 1: Official CarPlay video app (iOS 27)

This is the only App Store legal route and the one this repository should target if the goal is a
real app.

**How it works**

- The app declares the `com.apple.developer.carplay-video` entitlement (iOS 27+). Apple grants it
  on request at <https://developer.apple.com/carplay/> after you agree to the CarPlay Entitlement
  Addendum. Audio and video entitlements may be combined in one app so it still appears in cars
  without video support.
- The app uses the normal CarPlay template framework (`CPTemplateApplicationScene`, list, grid,
  tab bar, now playing templates) to provide a browsing UI on the car screen.
- Actual video playback is not drawn by the app. It goes over the AirPlay video path to the head
  unit. Apple's guide says video apps "must support AirPlay video streaming"; if the app already
  does, no extra playback work is needed.
- `CPSessionConfiguration.supportsVideoPlayback` (iOS 26.4+) tells the app whether the current
  session allows video. Show video content when true and audio-first content otherwise.
- `CPPlaybackConfiguration` (iOS 26.4+) carries elapsed time, duration, play/pause/replay action,
  and a `preferredPresentation` of `.video` or `.audio`. Attach it to list thumbnails and the
  details header, and update it on every playback state change.
- The system enforces the parked rule. Apple DTS confirmed apps do not implement their own
  gear lock. When the car reports video unavailable, playback continues as audio only.
- The in-car player supplies subtitle and audio-language menus, so expose those tracks.

**Testing without a supporting car**

Apple's CarPlay Simulator (in "Additional Tools for Xcode", also reachable via Device Hub) has a
vehicle configuration that advertises video support. Note the Xcode 27 beta CarPlay display was
reported black on iOS 27 simulator configurations, and one developer forum thread reports the
video entitlement causing a blank screen with no scene callbacks through the iOS 27 RC. Expect
rough edges.

**Guidelines to satisfy**

1. The app must be designed primarily to provide video playback services.
2. It must support AirPlay video streaming.

**Blocker:** you need a head unit that supports video in car. If the target screen is an existing
vehicle, this option will not light up on it today.

## Option 2: AirPlay video only, no CarPlay app (iOS 26.4+)

If the car supports video in car, any AirPlay-capable video app on the iPhone can already pick the
car display from the AirPlay menu while parked. This needs no entitlement and no CarPlay code.
Same vehicle blocker as Option 1.

## Option 3: Android "AI box" adapter (works today on any CarPlay screen)

Devices such as the Ottocast P3 Pro, Ottocast Play2Video, and Carlinkit TBox Ultra plug into the
car's wired CarPlay USB port, present themselves to the head unit as a CarPlay phone, and run a
full Android OS with YouTube, Netflix, and so on. This is the only route that puts video on an
existing CarPlay screen with no jailbreak and no automaker support.

- No code to write. Not an iPhone app at all.
- Quality varies by chipset. Older boxes on Android 8 or 9 are losing streaming app support.
- Reviewers describe the result as near native but a compromise.

## Option 4: Unofficial routes (development only, not shippable)

- **Navigation entitlement plus custom drawing.** Navigation apps get a `CPWindow` they draw
  into, and an `AVPlayerLayer` can be placed there. Apple's guide says navigation apps must be
  primarily navigation and cannot be combined with other categories, so this will not pass
  review, and Apple reviews every entitlement request against the stated purpose. Works only for
  a personal dev-signed build if you already hold the maps entitlement.
- **Jailbreak tweaks** (CarBridge, NGXPlay) mirror arbitrary iPhone apps to the CarPlay screen.
  Requires a jailbroken device on a supported iOS version.
- **Reimplementing the CarPlay protocol** on your own hardware so a Raspberry Pi acts as the
  phone. The protocol uses Apple MFi authentication and is not publicly documented. Open source
  projects (node-carplay, pi-carplay, pycarplay) go the other direction: they turn a Pi into a
  head unit by driving a Carlinkit dongle. Not useful for sending video to an existing screen.

## Recommendation

1. First find out whether the target head unit supports video in car. Quickest check: on an
   iPhone running iOS 26.4 or later, connect to the car, park, open an AirPlay-capable video app,
   and see whether the car display appears in the AirPlay picker.
2. If it does, or if the goal is to ship an app for cars that will, build Option 1 here: a Swift
   CarPlay video app with combined audio and video entitlements, AirPlay video playback, and the
   CarPlay Simulator video configuration for testing. Request the entitlement early since review
   can take weeks.
3. If the screen is an existing car with no video support and the goal is just to watch video,
   Option 3 is the practical answer and needs no code.

## Sources

- Apple, CarPlay Developer Guide, 2026-06-08 edition: <https://developer.apple.com/download/files/CarPlay-Developer-Guide.pdf>
- Apple, WWDC26 session 212 "Rev up your CarPlay app": <https://developer.apple.com/videos/play/wwdc2026/212/>
- Apple docs, `CPSessionConfiguration.supportsVideoPlayback`: <https://developer.apple.com/documentation/carplay/cpsessionconfiguration/supportsvideoplayback>
- Apple docs, `CPPlaybackConfiguration`: <https://developer.apple.com/documentation/carplay/cpplaybackconfiguration>
- Apple Developer Forums, video entitlement blank screen: <https://developer.apple.com/forums/thread/843374>
- Apple Developer Forums, iOS 27 video playback questions: <https://developer.apple.com/forums/thread/838529>
- MacRumors, iOS 27 CarPlay video apps: <https://www.macrumors.com/2026/06/08/new-apple-carplay-features-ios-27/>
- 9to5Mac, CarPlay video in iOS 26: <https://9to5mac.com/2026/05/06/carplay-video-ios-26/>
- CarPlayLife, 2026 adapter roundup: <https://www.carplaylife.com/feature/best-wireless-adapters-to-convert-wired-carplay-into-wireless-android-auto-in-2026/>
- react-carplay / node-carplay: <https://github.com/rhysmorgan134/react-carplay>
