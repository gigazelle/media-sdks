# Migrating from the standalone Media SDK to Adobe Experience Platform Tags

This documentation covers Media Analytics implementation differences between the standalone Media SDK and the Media Analytics Tags extension.

Tags gives customers a simple way to deploy and manage all of the analytics, marketing, and advertising solutions necessary to power relevant customer experiences across web and mobile. Tags empowers anyone to build and maintain their own integrations with Tags. These integrations are called extensions. Extensions are available to Tags customers in an app-store experience, so they can quickly install, configure, and deploy their solutions.

Adobe Media Analytics for Audio and Video is available as a Tags extension for both web and mobile:

* [Web Tags Documentation](https://experienceleague.adobe.com/docs/experience-platform/tags/extensions/adobe/media-analytics/overview.html)
* [Mobile Tags Documentation](https://developer.adobe.com/client-sdks/documentation/adobe-media-analytics/)

---

## Android

### Configuration

#### Standalone Media SDK

In the standalone Media SDK, you configure tracking in the app, and pass it to the SDK when you create the tracker.

```java
MediaHeartbeatConfig config = new MediaHeartbeatConfig();
config.trackingServer = "namespace.hb.omtrdc.net";
config.channel = "sample-channel";
config.appVersion = "v2.0.0";
config.ovp = "video-provider";
config.playerName = "native-player";
config.ssl = true;
config.debugLogging = true;

MediaHeartbeat tracker = new MediaHeartbeat(... , config);
```

#### Tags Extension

1. In Experience Platform Tags, click the **Extensions** tab for your mobile property.
1. On the **Catalog** tab, locate the Adobe Media Analytics for Audio and Video extension, and click **Install**.
1. In the extension settings page, configure the tracking parameters. The Media extension will use the configured parameters for tracking.

[Using Mobile Extensions](https://developer.adobe.com/client-sdks/documentation/adobe-media-analytics/)

### Tracker Creation

#### Standalone Media SDK

In the standalone Media SDK you manually create the `MediaHeartbeatConfig` object and configure the tracking parameters. Implement the delegate interface exposing `getQoSObject()` and `getCurrentPlaybackTime()` functions. Create a `MediaHeartbeat` instance for tracking.

```java
MediaHeartbeatConfig config = new MediaHeartbeatConfig();
config.trackingServer = "namespace.hb.omtrdc.net";
config.channel = "sample-channel";
config.appVersion = "v2.0";
config.ovp = "video-provider";
config.playerName = "native-player";
config.ssl = true;
config.debugLogging = true;

MediaHeartbeatDelegate delegate = new MediaHeartbeatDelegate() {
    @Override
    public MediaObject getQoSObject() {
        // When called should return the latest qos values.
        return MediaHeartbeat.createQoSObject(<bitrate>,
                                              <startupTime>,
                                              <fps>,
                                              <droppedFrames>);
    }

    @Override
    public Double getCurrentPlaybackTime() {
        // When called should return the current player time in seconds.
        return <currentPlaybackTime>;
    }

    MediaHeartbeat tracker = new MediaHeartbeat(delegate, config);
}
```

#### Tags Extension

[Media API reference - Create a Media Tracker](https://developer.adobe.com/client-sdks/documentation/adobe-media-analytics/api-reference/#createtracker)

Before you create the tracker, you should register the media extension and dependent extensions with the mobile core.

```java
// Register the extension once during app launch
try {
    // Media needs Identity and Analytics extension to function properly
    Identity.registerExtension();
    Analytics.registerExtension();

    // Initialize media extension.
    Media.registerExtension();
    MobileCore.start(new AdobeCallback () {
        @Override
        public void call(Object o) {
            // Launch mobile property to pick extension settings.
            MobileCore.configureWithAppID("LAUNCH_MOBILE_PROPERTY");
        }
    });
} catch (InvalidInitException ex) {
    ...
}
```

Once you register the media extension, create the tracker using the following API. The tracker automatically picks the configuration from the configured Tags property.

```java
Media.createTracker(new AdobeCallback<MediaTracker>() {
    @Override
    public void call(MediaTracker tracker) {
        // Use the instance for tracking media.
    }
});
```

### Updating Playhead and Quality of Experience values

#### Standalone Media SDK

In the standalone Media SDK, you pass a delegate object that implements the `MediaHeartbeatDelegate` interface during tracker creation. The implementation should return the latest QoE and playhead whenever the tracker calls the `getQoSObject()` and `getCurrentPlaybackTime()` interface methods.

#### Tags Extension

The implementation should update the current player playhead by calling the `updateCurrentPlayhead` method exposed by the tracker. For accurate tracking you should call this method at least once per second.

[Media API reference - Update Current Player](https://developer.adobe.com/client-sdks/documentation/adobe-media-analytics/api-reference/#updatecurrentplayhead)

The implementation should update the QoE information by calling the `updateQoEObject` method exposed by the tracker. Call this method whenever there is a change in the quality metrics.

[Media API reference - Update QoE Object](https://developer.adobe.com/client-sdks/documentation/adobe-media-analytics/api-reference/#createqoeobject)

### Passing standard media / ad metadata

#### Standalone Media SDK

* Standard Media Metadata:

   ```java
   MediaObject mediaInfo =
     MediaHeartbeat.createMediaObject("media-name",
                                      "media-id",
                                      60D,
                                      MediaHeartbeat.StreamType.VOD,
                                      MediaHeartbeat.MediaType.Video);

   Map <String, String> standardVideoMetadata = new HashMap<String, String>();
   standardVideoMetadata.put(MediaHeartbeat.VideoMetadataKeys.EPISODE, "Sample Episode");
   standardVideoMetadata.put(MediaHeartbeat.VideoMetadataKeys.SHOW, "Sample Show");
   standardVideoMetadata.put(MediaHeartbeat.VideoMetadataKeys.SEASON, "Sample Season");
   mediaInfo.setValue(MediaHeartbeat.MediaObjectKey.StandardMediaMetadata, standardVideoMetadata);

   HashMap<String, String> mediaMetadata = new HashMap<String, String>();
   mediaMetadata.put("isUserLoggedIn", "false");
   mediaMetadata.put("tvStation", "Sample TV Station");
   tracker.trackSessionStart(mediaInfo, mediaMetadata);
   ```

* Standard Ad Metadata:

   ```java
   MediaObject adInfo =
     MediaHeartbeat.createAdObject("ad-name", "ad-id", 1L, 15D);

   Map <String, String> standardAdMetadata = new HashMap<String, String>();
   standardAdMetadata.put(MediaHeartbeat.AdMetadataKeys.ADVERTISER, "Sample Advertiser");
   standardAdMetadata.put(MediaHeartbeat.AdMetadataKeys.CAMPAIGN_ID, "Sample Campaign");
   adInfo.setValue(MediaHeartbeat.MediaObjectKey.StandardAdMetadata, standardAdMetadata);

   HashMap<String, String> adMetadata = new HashMap<String, String>();
   adMetadata.put("affiliate", "Sample affiliate");
   tracker.trackEvent(MediaHeartbeat.Event.AdStart, adObject, adMetadata);
   ```

#### Tags Extension

* Standard Media Metadata:

   ```java
   HashMap<String, Object> mediaObject =
     Media.createMediaObject("media-name", "media-id", 60D,
                             MediaConstants.StreamType.VOD, Media.MediaType.Video);

   HashMap<String, String> mediaMetadata = new HashMap<String, String>();
   mediaMetadata.put(MediaConstants.VideoMetadataKeys.EPISODE, "Sample Episode");
   mediaMetadata.put(MediaConstants.VideoMetadataKeys.SHOW, "Sample Show");
   mediaMetadata.put("isUserLoggedIn", "false");
   mediaMetadata.put("tvStation", "Sample TV Station");

   tracker.trackSessionStart(mediaInfo, mediaMetadata);
   ```

* Standard Ad Metadata:

   ```java
   HashMap<String, Object> adObject =
     Media.createAdObject("ad-name", "ad-id", 1L, 15D);
   HashMap<String, String> adMetadata = new HashMap<String, String>();
   adMetadata.put(MediaConstants.AdMetadataKeys.ADVERTISER, "Sample Advertiser");
   adMetadata.put(MediaConstants.AdMetadataKeys.CAMPAIGN_ID, "Sample Campaign");
   adMetadata.put("affiliate", "Sample affiliate");
   _tracker.trackEvent(Media.Event.AdStart, adObject, adMetadata);
   ```

---

## iOS

### Configuration

#### Standalone Media SDK

In the standalone Media SDK, you configure the tracking configuration in the app, and pass it to the SDK when you create the tracker.

```objective-c
ADBMediaHeartbeatConfig *config = [[ADBMediaHeartbeatConfig alloc] init];

config.trackingServer = @"namespace.hb.omtrdc.net";
config.channel = @"sample-channel";
config.appVersion = @"v2.0.0";
config.ovp = @"video-provider";
config.playerName = @"native-player";
config.ssl = YES;
config.debugLogging = YES;

ADBMediaHeartbeat* tracker =
  [[ADBMediaHeartbeat alloc] initWithDelegate:self config:config];
```

#### Tags Extension

1. In Experience Platform Tags, click the **Extensions** tab for your mobile property.
1. On the **Catalog** tab, locate the Adobe Media Analytics for Audio and Video extension, and click **Install**.
1. In the extension settings page, configure the tracking parameters. The Media extension will use the configured parameters for tracking.

[Configure the Media Analytics extension](https://developer.adobe.com/client-sdks/documentation/adobe-media-analytics/)

### Tracker Creation

#### Standalone Media SDK

In the standalone Media SDK you manually create the `ADBMediaHeartbeatConfig` object and configure the tracking parameters. Implement the delegate interface exposing the `getQoSObject()` and `getCurrentPlaybackTime()` functions.

Create a MediaHeartbeat instance for tracking:

```objective-c
@interface PlayerDelegate : NSObject<ADBMediaHeartbeatDelegate>
@end

@implementation PlayerDelegate {
}

- (NSTimeInterval) getCurrentPlaybackTime {
    // When called should return the current player time in seconds.
    return playhead_;
}

- (nonnull ADBMediaObject*) getQoSObject {
    // When called should return the latest qos values.
    return qosObject_;
}
@end

ADBMediaHeartbeatConfig *config = [[ADBMediaHeartbeatConfig alloc] init];
config.trackingServer = @"namespace.hb.omtrdc.net";
config.channel = @"sample-channel";
config.appVersion = @"app-version";
config.ovp = @"video-provider";
config.playerName = @"native-player";
config.ssl = YES;
config.debugLogging = YES;
ADBMediaHeartbeatDelegate* delegate = [[PlayerDelegate alloc] init];

ADBMediaHeartbeat* tracker =
  [[ADBMediaHeartbeat alloc] initWithDelegate:delegate config:config];
```

#### Tags Extension

[Media API reference - Create Media Tracker](https://developer.adobe.com/client-sdks/documentation/adobe-media-analytics/api-reference/#createtracker)

Before creating the tracker, register the media extension and dependent extensions with the mobile core.

```objective-c
// Register the extension once during app launch
#import <ACPCore.h>
#import <ACPAnalytics.h>
#import <ACPMedia.h>
#import <ACPIdentity.h>

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    [ACPCore setLogLevel:ACPMobileLogLevelDebug];
    [ACPCore configureWithAppId:@"your-launch-app-id"];
    [ACPMedia registerExtension];
    [ACPAnalytics registerExtension];
    [ACPIdentity registerExtension];
    [ACPCore start:nil];
    return YES;
}
```

Once the media extension is registered, tracker can be created using the following API. The tracker automatically picks the configuration from the configured Tags property.

```objective-c
[ACPMedia createTracker:^(ACPMediaTracker * _Nullable mediaTracker) {
    // Use the instance for tracking media.
}];
```

### Updating Playhead and Quality of Experience values

#### Standalone Media SDK

In the standalone Media SDK, a delegate object that implements the `ADBMediaHeartbeatDelegate` protocol is passed during tracker creation. The implementation should return the latest QoE and playhead whenever the tracker calls the `getQoSObject()` and `getCurrentPlaybackTime()` interface methods.

#### Tags Extension

The implementation should update the current player playhead by calling the `updateCurrentPlayhead` method exposed by the tracker. For accurate tracking you should call this method at least once per second.

[Media API reference - Update Current Playhead](https://developer.adobe.com/client-sdks/documentation/adobe-media-analytics/api-reference/#updatecurrentplayhead)

The implementation should update the QoE information by calling the `updateQoEObject` method exposed by the tracker. Call this method whenever there is a change in the quality metrics.

[Media API reference - Update QoE Object](https://developer.adobe.com/client-sdks/documentation/adobe-media-analytics/api-reference/#createqoeobject)

### Passing standard media / ad metadata

#### Standalone Media SDK

* Standard Media Metadata:

   ```objective-c
   ADBMediaObject *mediaObject =
     [ADBMediaHeartbeat createMediaObjectWithName:@"media-name"
                        mediaId:@"media-id"
                        length:60
                        streamType:ADBMediaHeartbeatStreamTypeVod
                        mediaType:ADBMediaTypeVideo];

   NSMutableDictionary *standardMetadata = [[NSMutableDictionary alloc] init];
   [standardMetadata setObject:@"Sample show" forKey:ADBVideoMetadataKeySHOW];
   [standardMetadata setObject:@"Sample season" forKey:ADBVideoMetadataKeySEASON];
   [mediaObject setValue:standardMetadata forKey:ADBMediaObjectKeyStandardMediaMetadata];

   NSMutableDictionary *videoMetadata = [[NSMutableDictionary alloc] init];
   [videoMetadata setObject:@"false" forKey:@"isUserLoggedIn"];
   [videoMetadata setObject:@"Sample TV station" forKey:@"tvStation"];

   [tracker trackSessionStart:mediaObject data:videoMetadata];
   ```

* Standard Ad Metadata:

   ```objective-c
   ADBMediaObject* adObject =
     [ADBMediaHeartbeat createAdObjectWithName:[adData objectForKey:@"name"]
                        adId:[adData objectForKey:@"id"]
                        position:[[adData objectForKey:@"position"] doubleValue]
                        length:[[adData objectForKey:@"length"] doubleValue]];

   NSMutableDictionary *standardMetadata = [[NSMutableDictionary alloc] init];
   [standardMetadata setObject:@"Sample Advertiser" forKey:ADBAdMetadataKeyADVERTISER];
   [standardMetadata setObject:@"Sample Campaign" forKey:ADBAdMetadataKeyCAMPAIGN_ID];
   [adObject setValue:standardMetadata forKey:ADBMediaObjectKeyStandardAdMetadata];

   NSMutableDictionary *adDictionary = [[NSMutableDictionary alloc] init];
   [adDictionary setObject:@"Sample affiliate" forKey:@"affiliate"];

   [tracker trackEvent:ADBMediaHeartbeatEventAdStart
            mediaObject:adObject
            data:adDictionary];
   ```

#### Tags Extension

* Standard Media Metadata:

   ```objective-c
   NSDictionary *mediaObject =
     [ACPMedia createMediaObjectWithName:@"media-name"
               mediaId:@"media-id"
               length:60
               streamType:ACPMediaStreamTypeVod
               mediaType:ACPMediaTypeVideo];

   NSMutableDictionary *mediaMetadata = [[NSMutableDictionary alloc] init];
   [mediaMetadata setObject:@"Sample show" forKey:ACPVideoMetadataKeyShow];
   [mediaMetadata setObject:@"Sample season" forKey:ACPVideoMetadataKeySeason];
   [mediaMetadata setObject:@"false" forKey:@"isUserLoggedIn"];
   [mediaMetadata setObject:@"Sample TV station" forKey:@"tvStation"];
   [_tracker trackSessionStart:mediaObject data:mediaMetadata];
   ```

* Standard Ad Metadata:

   ```objective-c
   NSDictionary* adObject =
     [ACPMedia createAdObjectWithName:@"ad-name"
               adId:@"ad-id"
               position:1
               length:15];

   NSMutableDictionary* adMetadata = [[NSMutableDictionary alloc] init];
   [adMetadata setObject:@"Sample Advertiser" forKey:ACPAdMetadataKeyAdvertiser];
   [adMetadata setObject:@"Sample Campaign" forKey:ACPAdMetadataKeyCampaignId];
   [adMetadata setObject:@"Sample affiliate" forKey:@"affiliate"];

   [tracker trackEvent:ACPMediaEventAdStart mediaObject:adObject data:adMetadata];
   ```

---

## Web (JavaScript)

### Feature differences

* **Tags**: Tags provides you with a UI that walks you through setting up, configuring, and deploying your web-based media tracking solutions. Tags improves upon Dynamic Tag Management (DTM).
* **Media SDK**: The Media SDK provides you with media tracking libraries designed for specific platforms (e.g.: Android, iOS, etc.). Adobe recommends Media SDK for tracking media usage in your Mobile Apps.

### Configuration

#### Standalone Media SDK

In the standalone Media SDK, you configure the tracking configuration in the app and pass it to the SDK when you create the tracker.

```javascript
//Media Heartbeat initialization
var mediaConfig = new MediaHeartbeatConfig();
mediaConfig.trackingServer = "namespace.hb.omtrdc.net";
mediaConfig.playerName = "html5-player";
mediaConfig.channel = "sample-channel";
mediaConfig.ovp = "video-provider";
mediaConfig.appVersion = "v2.0.0"
mediaConfig.ssl = true;
mediaConfig.debugLogging = true;
```

In addition to the `MediaHeartbeat` configuration, the page must configure and pass the `AppMeasurement` instance and `VisitorAPI` instance for media tracking in order to work properly.

#### Tags Extension

1. In Experience Platform Tags, click the **Extensions** tab for your web property.
1. On the **Catalog** tab, locate the Adobe Media Analytics for Audio and Video extension, and click **Install**.
1. In the extension settings page, configure the tracking parameters. The Media extension will use the configured parameters for tracking.

[Tags User Guide - Install & configure the media extension](https://experienceleague.adobe.com/docs/experience-platform/tags/extensions/adobe/media-analytics/overview.html#install-and-configure-the-ma-extension)

### Tracker creation differences

#### Media SDK

1. Add the Media Analytics library to your development project.
1. Create a config object (`MediaHeartbeatConfig`).
1. Implement the delegate protocol, exposing the `getQoSObject()` and `getCurrentPlaybackTime()` functions.
1. Create a Media Heartbeat instance (`MediaHeartbeat`).

```javascript
// Media Heartbeat initialization
var mediaConfig = new MediaHeartbeatConfig();
// Configuration settings
mediaConfig.trackingServer = Configuration.HEARTBEAT.TRACKING_SERVER;
// Implement Media Delegate (Quality of Service and Playhead)
var mediaDelegate = new MediaHeartbeatDelegate();
mediaDelegate.getQoSObject = function() {
    return MediaHeartbeat.createQoSObject(<bitrate>, <startuptime>, <fps>, <droppedFrames>);
}
// Create your tracker
this.mediaHeartbeat = new MediaHeartbeat(mediaDelegate, mediaConfig, appMeasurement);
```

#### Tags

Tags offers two approaches to creating the tracking infrastructure. Both approaches use the Media Analytics Tags Extension:

1. Use the media tracking APIs from a web page.

    In this scenario, the Media Analytics Extension exports the media tracking APIs to a configured variable in the global window object:

    ```javascript
    window["CONFIGURED_VARIABLE_NAME"].MediaHeartbeat.getInstance
    ```

1. Use the media tracking APIs from another Tags extension.

    In this scenario, you use the media tracking APIs exposed by the `get-instance` and `media-heartbeat` Shared Modules.

    > Shared Modules are not available for use in web pages. You can only use Shared Modules from another extension.

    Create a `MediaHeartbeat` instance using the `get-instance` Shared Module. Pass a delegate object to `get-instance` that exposes `getQoSObject()` and `getCurrentPlaybackTime()` functions.

    ```javascript
    var getMediaHeartbeatInstance =
    turbine.getSharedModule('adobe-video-analytics', 'get-instance');
    ```

    Access `MediaHeartbeat` constants via the `media-heartbeat` Shared Module.

### Related Documentation

* [Set up JavaScript 2.x](../2.x/javascript-setup.md)
* [Media SDK JS API](https://adobe-marketing-cloud.github.io/media-sdks/reference/javascript/MediaHeartbeat.html)
* [Tags overview](https://experienceleague.adobe.com/docs/experience-platform/tags/home.html)
* [Media Analytics Extension](https://experienceleague.adobe.com/docs/experience-platform/tags/extensions/adobe/media-analytics/overview.html)
