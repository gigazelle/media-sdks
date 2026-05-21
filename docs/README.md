# Adobe Media SDK Documentation

This folder contains implementation guides, migration references, and validation docs for the Adobe Media SDK (formerly Video Heartbeat Library).

> **Looking for the current implementation?** Adobe recommends the [Edge Network implementation](https://experienceleague.adobe.com/docs/media-analytics/using/implementation/edge/implementation-edge.html) for all new projects.

## SDK versions

| Version | Status | Platforms |
|---|---|---|
| 3.x | Supported (Analytics-only) — see [Experience League](https://experienceleague.adobe.com/docs/media-analytics/using/implementation/overview.html) | JavaScript, Chromecast, Roku |
| [2.x](./2.x/) | End of support Aug 31, 2021 | Android, iOS, JavaScript |
| [1.x](https://github.com/Adobe-Marketing-Cloud/video-heartbeat/tree/master/docs) | Deprecated | Android, iOS, JavaScript, Apple TV, Chromecast, Roku, TVML |

## Contents

### Setup guides
- [2.x overview](./2.x/overview.md)
- [2.x Android setup](./2.x/android-setup.md)
- [2.x iOS / tvOS setup](./2.x/ios-setup.md)
- [2.x JavaScript setup](./2.x/javascript-setup.md)

### Migration guides
- [1.x to 2.x migration](./migration/1x-to-2x.md)
- [Standalone SDK to Tags (Launch) migration](./migration/sdk-to-tags.md)

### Validation
- [Validation overview](./validation/overview.md)
- [Heartbeat parameters reference](./validation/heartbeat-params.md)
- [Test 1: Standard playback](./validation/test1-standard-playback.md)
- [Test 2: Media interrupt](./validation/test2-media-interrupt.md)
- [Test call details](./validation/test-call-details.md)

## Samples

See the [samples/](../samples/) folder for runnable code examples organized by SDK version.
