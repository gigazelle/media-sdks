# Migrating from Media SDK 1.x to 2.x

## Overview

The migration from VHL 1.x to VHL 2.x is straightforward, with the new version featuring simplified APIs for initialization, configuration, and player delegates.

Here are the primary differences between 1.x and 2.x:

* **Plugins and delegates**: You no longer need to implement plugins and delegates for Analytics, VideoPlayer, and Heartbeat.
* **Configuration**: You no longer need to instantiate configurations for the 1.x plugins.

### Benefits of 2.x

* All of the public methods are consolidated into the `MediaHeartbeat` class to make implementation easier on developers.
* All configs are now consolidated into the `MediaHeartbeatConfig` class.
* You no longer need to instantiate configs for the Analytics, VideoPlayer, and Heartbeat plugins. You only need to instantiate the `MediaHeartbeat` class with `MediaHeartbeatDelegate` and `MediaHeartbeatConfig` instances. This is the only implementation that is required to initialize Media Analytics.

   With the initialization of `MediaHeartbeat`, you can safely delete all of the implementation for Analytics Plugin, VideoPlayer Plugin, and Heartbeat Plugin. Also, remove all the existing implementation for initialization that takes in an array of plugins as an input.

---

## Code Comparison

All of the configuration parameters and tracking APIs are now consolidated into the `MediaHeartbeats` and `MediaHeartbeatConfig` classes.

**Configuration API changes:**

* `AdobeHeartbeatPluginConfig.sdk`: Renamed to `MediaConfig.appVersion`
* `MediaHeartbeatConfig.playerName`: Now set through `MediaHeartbeatConfig` instead of `VideoPlayerPluginDelegate`
* The `AppMeasurement` instance (JavaScript only): Now sent through the `MediaHeartbeat` constructor.

**Configuration properties changes:**

* `sdk`: Renamed to `appVersion`
* `publisher`: Removed; IMS Org ID is used instead as a publisher
* `quietMode`: Removed

**Links to 1.x and 2.x sample players:**

* [1.x Sample Player](https://github.com/Adobe-Marketing-Cloud/video-heartbeat/blob/master/sdks/js/samples/BasicPlayerSample/script/app/analytics/video.analytics.provider.js#L58)
* [2.x Sample Player](https://github.com/Adobe-Marketing-Cloud/media-sdks/blob/master/sdks/js/2.x/samples/BasicPlayerSample/script/app/analytics/video.analytics.provider.js#L47)

The following sections provide code comparisons between 1.x and 2.x, covering Initialization, Core Playback, Ad Playback, Chapter Playback, and some additional events.

### VHL Code Comparison: INITIALIZATION

#### Object Initialization

| 1.x API | 2.x API |
| --- | --- |
| `Heartbeat()` | `MediaHeartbeat()` |
| `VideoPlayerPlugin()` | `MediaHeartbeatConfig()` |
| `AdobeAnalyticsPlugin()` | |
| `HeartbeatPlugin()` | |

#### Video player plugin initialization (1.x)

```js
this._playerPlugin = new VideoPlayerPlugin( new SampleVideoPlayerPluginDelegate(this._player));
var playerPluginConfig = new VideoPlayerPluginConfig();
playerPluginConfig.debugLogging = true;

// Set up the AppMeasurement plugin
this._aaPlugin = new AdobeAnalyticsPlugin( appMeasurement, new SampleAdobeAnalyticsPluginDelegate());
var aaPluginConfig = new AdobeAnalyticsPluginConfig();
aaPluginConfig.channel = Configuration.HEARTBEAT.CHANNEL;
aaPluginConfig.debuglogging = true;
this._aaPlugin.configure(aaPluginConfig);

// Set up the AdobeHeartbeat plugin
var ahPlugin = new AdobeHeartbeatPlugin( new SampleAdobeHeartbeatPluginDelegate());
var ahPluginConfig = new AdobeHeartbeatPluginConfig( configuration.HEARTBEAT.TRACKING_SERVER, configuration.HEARTBEAT.PUBLISHER);
ahPluginConfig.ovp = configuration.HEARTBEAT.OVP;
ahPluginConfig.sdk = configuration.HEARTBEAT.SDK;
ahPluginConfig.debugLogging = true;
ahPlugin.configure(ahPluginConfig);
var plugins = [this._playerPlugin, this._aaPlugin, ahPlugin];

// Set up and configure the heartbeat library
this._heartbeat = new Heartbeat(new SampleHeartbeatDelegate(), plugins);
var configData = new HeartbeatConfig();
configData.debugLogging = true;
this._heartbeat.configure(configData);
```

#### Media Heartbeat initialization (2.x)

```js
var mediaConfig = new MediaHeartbeatConfig();
mediaConfig.trackingServer = Configuration.HEARTBEAT.TRACKING_SERVER;
mediaConfig.playerName = Configuration.PLAYER.NAME;
mediaConfig.debugLogging = true;
mediaConfig.channel = Configuration.HEARTBEAT.CHANNEL;
mediaConfig.ssl = false;
mediaConfig.ovp = Configuration.HEARTBEAT.OVP;
mediaConfig.appVersion = Configuration.HEARTBEAT.SDK;
this._mediaHeartbeat = new MediaHeartbeat( new SampleMediaHeartbeatDelegate(this._player), mediaConfig, appMeasurement);
```

#### Delegates

| 1.x API | 2.x API |
| --- | --- |
| `VideoPlayerPluginDelegate()` | `MediaHeartbeatDelegate()` |
| `VideoPlayerPluginDelegate().getVideoInfo` | `MediaHeartbeatDelegate().getCurrentPlaybackTime` |
| `VideoPlayerPluginDelegate().getAdBreakInfo` | `MediaHeartbeatDelegate().getQoSObject` |
| `VideoPlayerPluginDelegate().getAdInfo` | |
| `VideoPlayerPluginDelegate().getChapterInfo` | |
| `VideoPlayerPluginDelegate().getQoSInfo` | |
| `VideoPlayerPluginDelegate().get.onError` | |
| `AdobeAnalyticsPluginDelegate()` | |

#### VideoPlayerPluginDelegate (1.x)

```js
$.extend(SampleVideoPlayerPluginDelegate.prototype, VideoPlayerPluginDelegate.prototype);

function SampleVideoPlayerPluginDelegate(player) {
    this._player = player;
}

SampleVideoPlayerPluginDelegate.prototype.getVideoInfo = function() {
    return this._player.getVideoInfo();
};

SampleVideoPlayerPluginDelegate.prototype.getAdBreakInfo = function() {
    return this._player.getAdBreakInfo();
};

SampleVideoPlayerPluginDelegate.prototype.getAdInfo = function() {
    return this._player.getAdInfo();
};

SampleVideoPlayerPluginDelegate.prototype.getChapterInfo = function() {
    return this._player.getChapterInfo();
};

SampleVideoPlayerPluginDelegate.prototype.getQoSInfo = function() {
    return this._player.getQoSInfo();
};
```

#### AdobeAnalyticsPluginDelegate (1.x)

```js
$.extend(SampleAdobeAnalyticsPluginDelegate.prototype, AdobeAnalyticsPluginDelegate.prototype);

function SampleAdobeAnalyticsPluginDelegate() {}

SampleAdobeAnalyticsPluginDelegate.prototype.onError = function(errorInfo) {
    console.log("AdobeAnalyticsPlugin error: " + errorInfo.getMessage() + " | " + errorInfo.getDetails());
};
```

#### HeartbeatDelegate (1.x)

```js
$.extend(SampleHeartbeatDelegate.prototype, HeartbeatDelegate.prototype);

function SampleHeartbeatDelegate() {}

SampleHeartbeatDelegate.prototype.onError = function(errorInfo) {
    console.log("Heartbeat error: " + errorInfo.getMessage() + " | " + errorInfo.getDetails());
};
```

#### MediaHeartbeatDelegate (2.x)

```js
ADB.core.extend(SampleMediaHeartbeatDelegate.prototype, MediaHeartbeatDelegate.prototype);

function SampleMediaHeartbeatDelegate(player) {
    this._player = player;
}

SampleMediaHeartbeatDelegate.prototype.getCurrentPlaybackTime = function() {
    return this._player.getCurrentPlaybackTime();
};

SampleMediaHeartbeatDelegate.prototype.getQoSObject = function() {
    return this._player.getQoSInfo();
};

this._mediaHeartbeat = new MediaHeartbeat(new SampleMediaHeartbeatDelegate(this._player), mediaConfig, appMeasurement);
```

### VHL Code Comparison: CORE PLAYBACK

#### Session Start

| VHL 1.x | VHL 2.x |
| --- | --- |
| `VideoPlayerPluginDelegate.trackVideoLoad()` | `MediaHeartbeat.createMediaObject()` |
| `VideoPlayerPluginDelegate.getVideoInfo()` | `MediaHeartbeat.trackSessionStart()` |

##### Session Start (1.x)

```js
VideoAnalyticsProvider.prototype._onLoad = function() {
    this._playerPlugin.trackVideoLoad();
};

SampleVideoPlayerPluginDelegate.prototype.getVideoInfo = function() {
    return this._player.getVideoInfo();
};

VideoPlayer.prototype.getVideoInfo = function() {
    this._videoInfo.playhead = vTime;
    return this._videoInfo;
};
```

##### Session Start (2.x)

```js
VideoAnalyticsProvider.prototype._onLoad = function() {
    var contextData = {};
    var videoInfo = this._player.getVideoInfo();
    var mediaInfo = MediaHeartbeat.createMediaObject(videoInfo.name, videoInfo.id, videoInfo.length, videoInfo.streamType);
    this._mediaHeartbeat.trackSessionStart(mediaInfo, contextData);
};
```

#### Standard Video Metadata

| 1.x API | 2.x API |
| --- | --- |
| `VideoMetadataKeys()` | `MediaHeartbeat.createMediaObject()` |
| `AdobeAnalyticsPlugin.setVideoMetadata()` | `MediaHeartbeat.trackSessionStart()` |

##### Standard Metadata (1.x)

```js
VideoAnalyticsProvider.prototype._onLoad = function() {
    var contextData = {};

    // Setting Standard Video Metadata
    contextData[VideoMetadataKeys.SEASON] = "sample season";
    contextData[VideoMetadataKeys.SHOW] = "sample show";
    contextData[VideoMetadataKeys.EPISODE] = "sample episode";
    contextData[VideoMetadataKeys.ASSET_ID] = "sample asset id";
    contextData[VideoMetadataKeys.GENRE] = "sample genre";
    contextData[VideoMetadataKeys.FIRST_AIR_DATE] = "sample air date";

    this._aaPlugin.setVideoMetadata(contextData);
    this._playerPlugin.trackVideoLoad();
};
```

##### Standard Metadata (2.x)

```js
VideoAnalyticsProvider.prototype._onLoad = function() {
    var contextData = {};
    var mediaInfo = MediaHeartbeat.createMediaObject(videoInfo.name, videoInfo.id, videoInfo.length, videoInfo.streamType);

    // Set standard Video Metadata
    var standardVideoMetadata = {};
    standardVideoMetadata[VideoMetadataKeys.SEASON] = "sample season";
    standardVideoMetadata[VideoMetadataKeys.SHOW] = "sample show";
    standardVideoMetadata[VideoMetadataKeys.EPISODE] = "sample episode";
    standardVideoMetadata[VideoMetadataKeys.ASSET_ID] = "sample asset id";
    standardVideoMetadata[VideoMetadataKeys.GENRE] = "sample genre";
    standardVideoMetadata[VideoMetadataKeys.FIRST_AIR_DATE] = "sample air date";

    mediaInfo.setValue(MediaHeartbeat.MediaObjectKey.StandardVideoMetadata, standardVideoMetadata);
    this._mediaHeartbeat.trackSessionStart(mediaInfo, contextData);
};
```

> Instead of setting the Standard Video Metadata through the `AdobeAnalyticsPlugin.setVideoMetadata()` API, in VHL 2.0, the Standard Video Metadata is set through the MediaObject key `MediaObject.MediaObjectKey.StandardVideoMetadata()`.

#### Custom Video Metadata

| 1.x API | 2.x API |
| --- | --- |
| `VideoMetadataKeys()` | `MediaHeartbeat.createMediaObject()` |
| `AdobeAnalyticsPlugin.setVideoMetadata()` | `MediaHeartbeat.trackSessionStart()` |

##### Custom Metadata (1.x)

```js
VideoAnalyticsProvider.prototype._onLoad = function() {
    var contextData = {
        isUserLoggedIn: "false",
        tvStation: "Sample TV station",
        programmer: "Sample programmer"
    };
    this._aaPlugin.setVideoMetadata(contextData);
    this._playerPlugin.trackVideoLoad();
};
```

##### Custom Metadata (2.x)

```js
VideoAnalyticsProvider.prototype._onLoad = function() {
    var contextData = {
        isUserLoggedIn: "false",
        tvStation: "Sample TV station",
        programmer: "Sample programmer"
    };
    var videoInfo = this._player.getVideoInfo();
    var mediaInfo = MediaHeartbeat.createMediaObject(videoInfo.name, videoInfo.id, videoInfo.length, videoInfo.streamType);
    mediaInfo.setValue(MediaHeartbeat.MediaObjectKey.StandardVideoMetadata, standardVideoMetadata);
    this._mediaHeartbeat.trackSessionStart(mediaInfo, contextData);
};
```

> Instead of setting the Custom Video Metadata through the `AdobeAnalyticsPlugin.setVideoMetadata()` API, in VHL 2.0, the Standard Video Metadata is set through the `MediaHeartbeat.trackSessionStart()` API.

#### Playback

| 1.x API | 2.x API |
| --- | --- |
| `VideoPlayerPlugin.trackPlay()` | `MediaHeartbeat.trackPlay()` |

##### Playback (1.x)

```js
VideoAnalyticsProvider.prototype._onSeekStart = function() {
    this._playerPlugin.trackSeekStart();
};
```

##### Playback (2.x)

```js
VideoAnalyticsProvider.prototype._onSeekStart = function() {
    this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.SeekStart);
};
```

#### Pause

| 1.x API | 2.x API |
| --- | --- |
| `VideoPlayerPlugin.trackPause()` | `MediaHeartbeat.trackPause()` |

##### Pause (1.x)

```js
VideoAnalyticsProvider.prototype._onPause = function() {
    this._playerPlugin.trackPause();
};
```

##### Pause (2.x)

```js
VideoAnalyticsProvider.prototype._onBufferComplete = function() {
    this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.BufferComplete);
};
```

#### Seek Complete

| 1.x API | 2.x API |
| --- | --- |
| `VideoPlayerPlugin.trackSeekComplete()` | `MediaHeartbeat.trackEvent(MediaHeartbeat.Event.SeekComplete)` |

##### Seeking (1.x)

```js
VideoAnalyticsProvider.prototype._onSeekComplete = function() {
    this._playerPlugin.trackSeekComplete();
};
```

##### Seeking (2.x)

```js
VideoAnalyticsProvider.prototype._onSeekComplete = function() {
    this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.SeekComplete);
};
```

#### Buffer Start

| 1.x API | 2.x API |
| --- | --- |
| `VideoPlayerPlugin.trackBufferStart()` | `MediaHeartbeat.trackEvent(MediaHeartbeat.Event.BufferStart)` |

##### Buffer Start (1.x)

```js
VideoAnalyticsProvider.prototype._onBufferStart = function() {
    this._playerPlugin.trackBufferStart();
};
```

##### Buffer Start (2.x)

```js
VideoAnalyticsProvider.prototype._onBufferStart = function() {
    this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.BufferStart);
};
```

#### Buffer Complete

| 1.x API | 2.x API |
| --- | --- |
| `VideoPlayerPlugin.trackBufferComplete()` | `MediaHeartbeat.trackEvent(MediaHeartbeat.Event.BufferComplete)` |

##### Buffer Complete (1.x)

```js
VideoAnalyticsProvider.prototype._onBufferComplete = function() {
    this._playerPlugin.trackBufferComplete();
};
```

##### Buffer Complete (2.x)

```js
VideoAnalyticsProvider.prototype._onBufferComplete = function() {
    this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.BufferComplete);
};
```

#### Playback Complete

| 1.x API | 2.x API |
| --- | --- |
| `VideoPlayerPlugin.trackComplete()` | `MediaHeartbeat.trackComplete()` |

##### Playback Complete (1.x)

```js
VideoAnalyticsProvider.prototype._onComplete = function() {
    this._playerPlugin.trackComplete(function() {
        console.log( "The completion of the content has been tracked.");
    });
};
```

##### Playback Complete (2.x)

```js
VideoAnalyticsProvider.prototype._onComplete = function() {
    this._mediaHeartbeat.trackComplete();
};
```

### VHL Code Comparison: AD PLAYBACK

#### Ad Start

| VHL 1.x | VHL 2.x |
| --- | --- |
| `VideoPlayerPlugin.trackAdStart()` | `MediaHeartbeat.createAdBreakObject()` |
| `VideoPlayerPluginDelegate.getAdBreakInfo()` | `MediaHeartbeat.createAdObject()` |
| `VideoPlayerPluginDelegate.getAdInfo()` | `MediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdBreakStart)` |
| | `MediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdStart)` |

##### Ad Start (1.x)

```js
VideoAnalyticsProvider.prototype._onAdStart = function() {
    this._playerPlugin.trackAdStart();
};

SampleVideoPlayerPluginDelegate.prototype.getAdInfo = function() {
    return this._player.getAdInfo();
};
```

##### Ad Start (2.x)

```js
VideoAnalyticsProvider.prototype._onAdStart = function() {
    var adContextData = {};

    var _adBreakInfo = this._player.getAdBreakInfo();
    var adBreakInfo = MediaHeartbeat.createAdBreakObject(_adBreakInfo.name, _adBreakInfo.position, _adBreakInfo.startTime);

    var _adInfo = this._player.getAdInfo();
    var adInfo = MediaHeartbeat.createAdObject(_adInfo.name, _adInfo.id, _adInfo.position, _adInfo.length);
    this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdBreakStart, adBreakInfo);
    this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdStart, adInfo, adContextData);
};
```

#### Standard Ad Metadata

| 1.x API | 2.x API |
| --- | --- |
| `AdMetadataKeys()` | `MediaHeartbeat.createAdObject()` |
| `AdobeAnalyticsPlugin.setAdMetadata()` | `MediaHeartbeat.trackAdStart()` |

##### Standard Ad Metadata (1.x)

```js
VideoAnalyticsProvider.prototype._onAdStart = function() {
    var contextData = {};
    contextData[AdMetadataKeys.ADVERTISER] = "sample advertiser";
    contextData[AdMetadataKeys.CAMPAIGN_ID] = "sample campaign";
    contextData[AdMetadataKeys.CREATIVE_ID] = "sample creative";
    contextData[AdMetadataKeys.CREATIVE_URL] = "sample url";
    contextData[AdMetadataKeys.SITE_ID] = "sample site";
    contextData[AdMetadataKeys.PLACEMENT_ID] = "sample placement";
    this._aaPlugin.setAdMetadata(contextData);
    this._playerPlugin.trackAdStart();
};
```

##### Standard Ad Metadata (2.x)

```js
VideoAnalyticsProvider.prototype._onAdStart = function() {
    var adContextData = { };

    var _adBreakInfo = this._player.getAdBreakInfo();
    var adBreakInfo = MediaHeartbeat.createAdBreakObject(_adBreakInfo.name, _adBreakInfo.position, _adBreakInfo.startTime);

    var _adInfo = this._player.getAdInfo();
    var adInfo = MediaHeartbeat.createAdObject(_adInfo.name, _adInfo.id, _adInfo.position, _adInfo.length);

    var standardAdMetadata = {};
    standardAdMetadata[MediaHeartbeat.AdMetadataKeys.ADVERTISER] = "Sample Advertiser";
    standardAdMetadata[MediaHeartbeat.AdMetadataKeys.CAMPAIGN_ID] = "Sample Campaign";
    adInfo.setValue(MediaHeartbeat.MediaObjectKey.StandardAdMetadata, standardAdMetadata);
    this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdBreakStart, adBreakInfo);
    this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdStart, adInfo, adContextData);
};
```

> Instead of setting the Standard Ad Metadata through the `AdobeAnalyticsPlugin.setVideoMetadata()` API, in VHL 2.0, the Standard Ad Metadata is set through the `AdMetadata` key `MediaObject.MediaObjectKey.StandardVideoMetadata`.

#### Custom Ad Metadata

| 1.x API | 2.x API |
| --- | --- |
| `AdobeAnalyticsPlugin.setAdMetadata()` | `MediaHeartbeat.createAdObject()` |
| | `MediaHeartbeat.trackAdStart()` |

##### Custom Ad Metadata (2.x)

```js
VideoAnalyticsProvider.prototype._onAdStart = function() {
    var adContextData = {
        affiliate: "Sample affiliate",
        campaign: "Sample ad campaign"
    };

    var _adBreakInfo = this._player.getAdBreakInfo();
    var adBreakInfo = MediaHeartbeat.createAdBreakObject(_adBreakInfo.name, _adBreakInfo.position, _adBreakInfo.startTime);

    var _adInfo = this._player.getAdInfo();
    var adInfo = MediaHeartbeat.createAdObject(_adInfo.name, _adInfo.id, _adInfo.position, _adInfo.length);
    this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdBreakStart, adBreakInfo);
    this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdStart, adInfo, adContextData);
};
```

> Instead of setting the Custom Ad Metadata through the `AdobeAnalyticsPlugin.setVideoMetadata` API, in VHL 2.0, the Standard Ad Metadata is set through the `MediaHeartbeat.trackAdStart()` API.

#### Ad Skip

##### Ad Skip (1.x)

```js
SampleVideoPlayerPluginDelegate.prototype.getAdInfo = function() {
    return this._player.getAdInfo();
};
```

##### Ad Skip (2.x)

```js
VideoAnalyticsProvider.prototype._onAdSkip = function() {
    this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdSkip);
};
```

> In VHL 1.5.X APIs, `getAdinfo()` and `getAdBreakInfo()` must return null if the player is outside the Ad break boundaries.

#### Ad Complete

| 1.x API | 2.x API |
| --- | --- |
| `VideoPlayerPlugin.trackAdComplete()` | `MediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdComplete)` |
| | `MediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdBreakComplete)` |

##### Ad Complete (1.x)

```js
VideoAnalyticsProvider.prototype._onAdComplete = function() {
    this._playerPlugin.trackAdComplete();
};
```

##### Ad Complete (2.x)

```js
VideoAnalyticsProvider.prototype._onAdComplete = function() {
    this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdComplete);
    this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdBreakComplete);
};
```

### VHL Code Comparison: CHAPTER PLAYBACK

#### Chapter Start

| VHL 1.x | VHL 2.x |
| --- | --- |
| `VideoPlayerPluginDelegate.getChapterInfo()` | `MediaHeartbeat.createChapterObject` |
| `VideoPlayerPlugin.trackChapterStart()` | `MediaHeartbeat.trackEvent(MediaHeartbeat.Event.ChapterStart)` |

##### Chapter Start (1.x)

```js
VideoAnalyticsProvider.prototype._onChapterStart = function() {
    this._playerPlugin.trackChapterStart();
};

SampleVideoPlayerPluginDelegate.prototype.getChapterInfo = function() {
    return this._player.getChapterInfo();
};
```

##### Chapter Start (2.x)

```js
VideoAnalyticsProvider.prototype._onChapterStart = function() {
    var chapterContextData = { };

    var _chapterInfo = this._player.getChapterInfo();
    var chapterInfo = MediaHeartbeat.createChapterObject(_chapterInfo.name, _chapterInfo.position, _chapterInfo.length, _chapterInfo.startTime);
    this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.ChapterStart, chapterInfo, chapterContextData);
};
```

#### Chapter Skip

| 1.x API | 2.x API |
| --- | --- |
| `VideoPlayerPluginDelegate.getChapterInfo()` | `MediaHeartbeat.trackEvent(MediaHeartbeat.Event.ChapterSkip)` |

##### Chapter Skip (1.x)

```js
SampleVideoPlayerPluginDelegate.prototype.getChapterInfo = function() {
    return this._player.getChapterInfo();
};
```

> In VHL 1.5.X APIs, `getChapterinfo()` must return null if the player is outside the Chapter boundaries.

##### Chapter Skip (2.x)

```js
VideoAnalyticsProvider.prototype._onChapterSkip = function() {
    this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.ChapterSkip);
};
```

#### Chapter Custom Metadata

| 1.x API | 2.x API |
| --- | --- |
| `VideoPlayerPlugin.trackChapterStart()` | `MediaHeartbeat.createChapterObject()` |
| `AdobeAnalyticsPlugin.setChapterMetadata()` | `MediaHeartbeat.trackEvent(MediaHeartbeat.Event.ChapterStart)` |

##### Chapter Custom Metadata (1.x)

```js
VideoAnalyticsProvider.prototype._onChapterStart = function() {
    this._aaPlugin.setChapterMetadata({
        segmentType: "Sample segment type"
    });
    this._playerPlugin.trackChapterStart();
};
```

##### Chapter Custom Metadata (2.x)

```js
VideoAnalyticsProvider.prototype._onChapterStart = function() {
    var chapterContextData = {
        segmentType: "Sample segment type"
    };

    var _chapterInfo = this._player.getChapterInfo();
    var chapterInfo = MediaHeartbeat.createChapterObject(_chapterInfo.name, _chapterInfo.position, _chapterInfo.length, _chapterInfo.startTime);
    this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.ChapterStart, chapterInfo, chapterContextData);
};
```

#### Chapter Complete

| 1.x API | 2.x API |
| --- | --- |
| `trackChapterComplete()` | `trackEvent(MediaHeartbeat.Event.ChapterComplete)` |

##### Chapter Complete (1.x)

```js
VideoAnalyticsProvider.prototype._onChapterComplete = function() {
    this._playerPlugin.trackChapterComplete();
};
```

##### Chapter Complete (2.x)

```js
VideoAnalyticsProvider.prototype._onChapterComplete = function() {
    this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.ChapterComplete);
};
```

### VHL Code Comparison: OTHER EVENTS

#### Bitrate Change

| VHL 1.x | VHL 2.x |
| --- | --- |
| `VideoPlayerPlugin.trackBitrateChange()` | `MediaHeartbeat.trackEvent(MediaHeartbeat.Event.BitrateChange)` |

##### Bitrate Change (1.x)

```js
VideoAnalyticsProvider.prototype._onBitrateChange = function() {
    // Update getQosInfo to return the updated bitrate
    this._playerPlugin.trackBitrateChange();
};
```

##### Bitrate Change (2.x)

```js
VideoAnalyticsProvider.prototype._onBitrateChange = function() {
    // Update getQosObject to return the updated bitrate
    this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.BitrateChange);
};
```

#### Video Resume

| 1.x API | 2.x API |
| --- | --- |
| `VideoInfo.resumed()` | `MediaObject()` |
| `VideoPlayerPluginDelegate.getVideoInfo()` | `MediaHeartbeat.trackSessionStart()` |
| `VideoPlayerPlugin.trackVideoLoad()` | |

##### Video Resume (1.x)

```js
this._videoInfo.resumed=true;

VideoPlayer.prototype.getVideoInfo = function() {
    this._videoInfo.playhead = vTime;
    return this._videoInfo;
};
```

##### Video Resume (2.x)

```js
VideoAnalyticsProvider.prototype._onLoad = function() {
    var contextData = {};
    var videoInfo = this._player.getVideoInfo();
    var mediaInfo = MediaHeartbeat.createMediaObject(videoInfo.playerName, videoInfo.id, videoInfo.length, videoInfo.streamType);
    mediaInfo.setValue(MediaHeartbeat.MediaObjectKey.VideoResumed, true);
    this._mediaHeartbeat.trackSessionStart(mediaInfo, contextData);
};
```

For more information on tracking video with 2.x, see [Track Core Video Playback](https://experienceleague.adobe.com/docs/media-analytics/using/use-cases/track-av-playback/track-core-overview.html).

---

## API Reference

### Media SDK 2.x API References

* [Android API Reference](https://adobe-marketing-cloud.github.io/media-sdks/reference/android/index.html)
* [iOS API Reference](https://adobe-marketing-cloud.github.io/media-sdks/reference/ios/index.html)
* [JS API Reference](https://adobe-marketing-cloud.github.io/media-sdks/reference/javascript/index.html)
* [Chromecast API Reference](https://adobe-marketing-cloud.github.io/media-sdks/reference/chromecast/index.html)

### Required Track* APIs

| VHL 1.x | VHL 2.x |
|---|---|
| `videoPlayerPlugin.trackVideoLoad()` | N/A |
| `videoPlayerPlugin.trackSessionStart()` | [mediaHeartbeat.trackSessionStart(mediaObject, mediaCustomMetadata)](https://adobe-marketing-cloud.github.io/media-sdks/reference/javascript/MediaHeartbeat.html#trackSessionStart) |
| `videoPlayerPlugin.trackPlay()` | [mediaHeartbeat.trackPlay()](https://adobe-marketing-cloud.github.io/media-sdks/reference/javascript/MediaHeartbeat.html#trackPlay) |
| `videoPlayerPlugin.trackPause()` | [mediaHeartbeat.trackPause()](https://adobe-marketing-cloud.github.io/media-sdks/reference/javascript/MediaHeartbeat.html#trackPause) |
| `videoPlayerPlugin.trackComplete()` | [mediaHeartbeat.trackComplete()](https://adobe-marketing-cloud.github.io/media-sdks/reference/javascript/MediaHeartbeat.html#trackComplete) |
| `videoPlayerPlugin.trackVideoUnload()` | [mediaHeartbeat.trackSessionEnd()](https://adobe-marketing-cloud.github.io/media-sdks/reference/javascript/MediaHeartbeat.html#trackSessionEnd) |
| `videoPlayerPlugin.trackApplicationError()` | N/A |
| `videoPlayerPlugin.trackVideoPlayerError()` | [mediaHeartbeat.trackError()](https://adobe-marketing-cloud.github.io/media-sdks/reference/javascript/MediaHeartbeat.html#trackError) |

All of the optional tracking APIs such as (Ads, Chapters, Bitrate change, Seeking, and Buffering) are now part of a single `trackEvent` API. The [trackEvent](https://adobe-marketing-cloud.github.io/media-sdks/reference/javascript/MediaHeartbeat.html#trackEvent) API receives a constant parameter that represents the type of event that it is intended to track:

### Optional trackEvent APIs

| VHL 1.x | VHL 2.x |
|---|---|
| Return a valid `AdBreakInfo` in `VideoPlayerPlugin.getAdBreakInfo()` | `trackEvent(Event.AdBreakStart)` |
| Return null in `VideoPlayerPlugin.getAdBreakInfo()` | `trackEvent(Event.AdBreakComplete)` |
| `playerPlugin.trackAdStart()` | `trackEvent(Event.AdStart, adObject, adCustomMetadata)` |
| `playerPlugin.trackAdComplete()` | `trackEvent(Event.AdComplete)` |
| Return null in `VideoPlayerPlugin.getAdInfo()` | `trackEvent(Event.AdSkip)` |
| `playerPlugin.trackChapterStart()` | `trackEvent(Event.ChapterStart, chapterObject, chapterCustomMetadata)` |
| `playerPlugin.trackChapterComplete()` | `trackEvent(Event.ChapterComplete)` |
| Return null in `VideoPlayerPlugin.getChapterInfo()` | `trackEvent(Event.ChapterSkip)` |
| `playerPlugin.trackSeekStart()` | `trackEvent(Event.SeekStart)` |
| `playerPlugin.trackSeekComplete()` | `trackEvent(Event.SeekComplete)` |
| `playerPlugin.trackBufferStart()` | `trackEvent(Event.BufferStart)` |
| `playerPlugin.trackBufferComplete()` | `trackEvent(Event.BufferComplete)` |
| `playerPlugin.trackBitrateChange()` | `trackEvent(Event.BitrateChange)` |
| `playerPlugin.trackTimedMetadata()` | `trackEvent(Event.TimedMetadataUpdate)` |
