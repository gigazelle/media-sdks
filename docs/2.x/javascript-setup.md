# Set up JavaScript 2.x

> **End of Support:** The Media SDK 2.x for JavaScript reached end of support on August 31, 2021. Migrate to the [Adobe Experience Platform Web SDK](https://experienceleague.adobe.com/docs/experience-platform/edge/home.html) or the [Edge Network implementation](https://experienceleague.adobe.com/docs/media-analytics/using/implementation/overview.html).

## Prerequisites

* **Obtain valid configuration parameters**: These parameters can be obtained from an Adobe representative after you set up your analytics account.
* **Implement `AppMeasurement` for JavaScript in your media application**: For more information about the Adobe Mobile SDK documentation, see [Implementing Analytics Using JavaScript.](https://experienceleague.adobe.com/docs/analytics/implementation/js/overview.html)

* **Provide the following capabilities in your media player:**

    * *An API to subscribe to player events*: The Media SDK requires that you call a set of simple APIs when events occur in your player.
    * *An API that provides player information*: This information includes details such as the media name and the play head position.

## SDK Implementation

1. Add your [downloaded](https://experienceleague.adobe.com/docs/media-analytics/using/getting-started/download-sdks.html) library to your project. Create local references to the classes for convenience.

   1. Expand the `MediaSDK-js-v2.*.zip` file that you downloaded.
   1. Verify that the `MediaSDK.min.js` file exists in the `libs` directory.
   1. Host the `MediaSDK.min.js` file.

      This core JavaScript file must be hosted on a web server that is accessible to all pages on your site. You need the path to these files for the next step.

   1. Reference `MediaSDK.min.js` on all site pages.

      Include `MediaSDK` for JavaScript by adding the following line of code in the `<head>` or `<body>` tag on each page. For example:

      ```html
      <script type="text/javascript"
      src="https://INSERT-DOMAIN-AND-PATH-TO-CODE-HERE/MediaSDK.min.js"></script>
      ```

   1. To quickly verify that the library was successfully imported, instantiate the `ADB.va.MediaHeartbeatConfig` class.

      > From Version 2.1.0, the JavaScript SDK is compliant with the AMD and CommonJS module specifications, and `VideoHeartbeat.min.js` can also be used with compatible module loaders.

1. For easy access to the APIs, create local references to the `MediaHeartbeat` classes.

   ```js
   var MediaHeartbeat = ADB.va.MediaHeartbeat;
   var MediaHeartbeatConfig = ADB.va.MediaHeartbeatConfig;
   var MediaHeartbeatDelegate = ADB.va.MediaHeartbeatDelegate;
   ```

1. Create a `MediaHeartbeatConfig` instance.

   This section helps you to understand `MediaHeartbeat` config parameters and how to set correct config values on your `MediaHeartbeat` instance, for accurate tracking.

   Here is a sample `MediaHeartbeatConfig` initialization:

   ```js
   //Media Heartbeat initialization
   var mediaConfig = new MediaHeartbeatConfig();
   mediaConfig.trackingServer = Configuration.HEARTBEAT.TRACKING_SERVER;
   mediaConfig.playerName = Configuration.PLAYER.NAME;
   mediaConfig.channel = Configuration.HEARTBEAT.CHANNEL;
   mediaConfig.debugLogging = true;
   mediaConfig.appVersion = Configuration.HEARTBEAT.SDK;
   mediaConfig.ssl = false;
   mediaConfig.ovp = Configuration.HEARTBEAT.OVP;
   ```

1. Implement the `MediaHeartbeatDelegate` protocol.

   ```js
   var mediaDelegate = new MediaHeartbeatDelegate();

   // Replace <currentPlaybackTime> with the video player current playback time
   mediaDelegate.getCurrentPlaybackTime = function() {
       return <currentPlaybackTime>;
   };

   // Replace <bitrate>, <startuptime>, <fps> and <droppedFrames> with the current playback QoS values.
   mediaDelegate.getQoSObject = function() {
       return MediaHeartbeat.createQoSObject(<bitrate>, <startuptime>, <fps>, <droppedFrames>);
   };
   ```

1. Create the `MediaHeartbeat` instance.

   Use the `MediaHeartbeatConfig` and `MediaHeartbeatDelegate` to create the `MediaHeartbeat` instance.

   ```js
   this.mediaHeartbeat = new MediaHeartbeat(mediaDelegate, mediaConfig, appMeasurement);
   ```

   > Make sure that your `MediaHeartbeat` instance is accessible and does not get deallocated until the end of the media session. This instance will be used for all of the following tracking events.

   > `MediaHeartbeat` requires an instance of `AppMeasurement` to send calls to Adobe Analytics. Here is an example of an `AppMeasurement` instance:

   ```js
   var appMeasurement = new AppMeasurement();
   appMeasurement.visitor = visitor;
   appMeasurement.trackingServer = "<visitor_namespace>.sc.omtrdc.net";
   appMeasurement.account = <rsid>;
   appMeasurement.pageName = <page_name>;
   appMeasurement.charSet = "UTF-8";
   ```

## Migrate from JavaScript 1.x to 2.x

In version 2.x, all of the public methods are consolidated into the `ADB.va.MediaHeartbeat` class to make it easier on developers. Also, all configs are now consolidated into the `ADB.va.MediaHeartbeatConfig` class.

For information about migrating from 1.x to 2.x, see the [1.x to 2.x migration guide](../migration/1x-to-2x.md).
