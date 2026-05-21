# Set up Android

> **End of Support:** The Media SDK 2.x for Android reached end of support on August 31, 2021. Migrate to the [Adobe Experience Platform Mobile SDK](https://developer.adobe.com/client-sdks/home/) or the [Edge Network implementation](https://experienceleague.adobe.com/docs/media-analytics/using/implementation/overview.html). For additional information, see [Media Analytics SDK End-of-Support FAQs](https://experienceleague.adobe.com/docs/media-analytics/using/additional-resources/end-of-support-faqs.html).

Learn how to set up streaming media services for Android devices.

## Prerequisites

* **Obtain valid configuration parameters for the Media SDK**: These parameters can be obtained from an Adobe representative after you set up your analytics account.
* **Implement ADBMobile for Android in your application**: For more information about the Adobe Mobile SDK documentation, see the [Android SDK 4.x documentation](https://github.com/Adobe-Marketing-Cloud/mobile-services/blob/master/docs/android/overview.md)

* **Provide the following capabilities in your media player:**
   * *An API to subscribe to player events*: The Media SDK requires that you call a set of simple APIs when events occur in your player.
   * *An API that provides player information*: This information includes details such as the media name and the play head position.

## SDK Implementation

1. Add your downloaded Media SDK to your project.

    1. Expand the Android zip file (e.g., `MediaSDK-android-v2.*.zip`).
    1. Verify that the `MediaSDK.jar` file exists in the `libs/` directory.

    1. Add the library to your project.

       **IntelliJ IDEA:**

        1. Right click your project in the **Project navigation** panel.
        1. Select **Open Module Settings**.
        1. Under **Project Settings**, select **Libraries**.
        1. Click **+** to add a new library.
        1. Select **Java** and navigate to the `MediaSDK.jar` file.
        1. Select the modules in which you plan to use the mobile library.
        1. Click **Apply** and then **OK** to close the Module Settings window.

       **Eclipse:**

        1. In the Eclipse IDE, right-click on the project name.
        1. Click **Build Path** > **Add External Archives**.
        1. Select `MediaSDK.jar`.
        1. Click **Open**.
        1. Right-click the project again, and click **Build Path** > **Configure Build Path**.
        1. Click the **Order** and **Export** tabs.
        1. Ensure that the `MediaSDK.jar` file is selected.

1. Import the library.

   ```java
   import com.adobe.primetime.va.simple.MediaHeartbeat;
   import com.adobe.primetime.va.simple.MediaHeartbeat.MediaHeartbeatDelegate;
   import com.adobe.primetime.va.simple.MediaHeartbeatConfig;
   import com.adobe.primetime.va.simple.MediaObject;
   ```

1. Create the `MediaHeartbeatConfig` instance.

   Here is a sample `MediaHeartbeatConfig` initialization:

   ```java
   // Media Heartbeat Initialization
   config.trackingServer = _<SAMPLE_HEARTBEAT_TRACKING_SERVER>_;
   config.channel = <SAMPLE_HEARTBEAT_CHANNEL>;
   config.appVersion = <SAMPLE_HEARTBEAT_SDK_VERSION>;
   config.ovp =  <SAMPLE_HEARTBEAT_OVP_NAME>;
   config.playerName = <SAMPLE_PLAYER_NAME>;
   config.ssl = <true/false>;
   config.debugLogging = <true/false>;
   ```

1. Implement the `MediaHeartbeatDelegate` interface.

   ```java
   public class VideoAnalyticsProvider implements Observer, MediaHeartbeatDelegate{}
   ```

   ```java
   // Replace <bitrate>, <startupTime>, <fps>, and
   // <droppeFrames> with the current playback QoS values.
   @Override
   public MediaObject getQoSObject() {
       return MediaHeartbeat.createQoSObject(<bitrate>,
                                             <startupTime>,
                                             <fps>,
                                             <droppedFrames>);
   }

   //Replace <currentPlaybackTime> with the video player current playback time
   @Override
   public Double getCurrentPlaybackTime() {
       return <currentPlaybackTime>;
   }
   ```

1. Create the `MediaHeartbeat` instance.

   Use the `MediaHeartbeatConfig` instance and the `MediaHeartbeatDelegate` instance to create the `MediaHeartbeat` instance.

   ```java
   // Replace <MediaHeartbeatDelegate> with your delegate instance
   MediaHeartbeat _heartbeat =
     new MediaHeartbeat(<MediaHeartbeatDelegate>, config);
   ```

   > Make sure that your `MediaHeartbeat` instance is accessible and *does not get deallocated until the end of the session*. This instance is used for all of the following tracking events.

## Adding App Permissions

Your app using the Media SDK requires the following permissions to send data in tracking calls:

* `INTERNET`
* `ACCESS_NETWORK_STATE`

To add these permissions, add the following lines to your `AndroidManifest.xml` file in the application project directory:

* `<uses-permission android:name="android.permission.INTERNET" />`
* `<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />`

## Migrating from version 1.x to 2.x in Android

In versions 2.x, all of the public methods are consolidated into the `com.adobe.primetime.va.simple.MediaHeartbeat` class to make it easier on developers. Also, all configs are now consolidated into the `com.adobe.primetime.va.simple.MediaHeartbeatConfig` class.

For information about migrating from 1.x to 2.x, see the [1.x to 2.x migration guide](../migration/1x-to-2x.md).
