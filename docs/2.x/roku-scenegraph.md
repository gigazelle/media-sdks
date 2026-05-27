# Roku — Tracking in SceneGraph

> **End of Support:** The Media SDK 2.x for Roku reached end of support on August 31, 2021. For the recommended implementation, see the [AEP Roku SDK](https://github.com/adobe/aepsdk-roku). For additional information, see [Media Analytics SDK End-of-Support FAQs](https://experienceleague.adobe.com/docs/discontinued/using/media-sdk.html).

## Introduction

You can use the Roku SceneGraph XML programming framework to develop applications. This framework features two key concepts:

* SceneGraph rendering of the application screens
* XML configuration of the SceneGraph screens

The Adobe Mobile SDK for Roku is written in BrightScript. The SDK uses many components that are not available for an app running on SceneGraph (for example, threads). Therefore, a Roku app developer intending to use the SceneGraph framework cannot call Adobe Mobile SDK APIs (the latter are similar to those available in legacy BrightScript apps).

## Architecture

To add SceneGraph support to the AdobeMobile SDK, Adobe has added a new API that creates a connector bridge between the AdobeMobile SDK and `adbmobileTask`. The latter is a SceneGraph node used for the SDK's API execution.

The connector bridge is designed to perform as follows:

* The bridge returns a SceneGraph-compatible instance of the AdobeMobile SDK. The SceneGraph-compatible SDK has all of the APIs that the legacy SDK exposes.
* You use the AdobeMobile SDK APIs in SceneGraph in a very similar way to how you used the legacy APIs.
* The bridge also exposes a mechanism to listen for callbacks for APIs that return some data.

## Components

**SceneGraph Application:**

* Consumes `AdobeMobileLibrary` APIs via the SceneGraph connector bridge APIs.
* Registers for response callbacks on `adbmobileTask` for expected output data variables.

**AdobeMobileLibrary:**

* Exposes a set of public APIs (Legacy), including the connector bridge API.
* Returns a SceneGraph connector instance that wraps all legacy public APIs.
* Communicates with an `adbmobileTask` SceneGraph node for execution of APIs.

**`adbmobileTask` Node:**

* A SceneGraph task node that executes `AdobeMobileLibrary` APIs on a background thread.
* Serves as a delegate to return data back to application scenes.

## Public SceneGraph APIs

### ADBMobileConnector

#### Constants

| Method | Description |
|---|---|
| `sceneGraphConstants` | Returns an object containing `SceneGraphConstants`. Refer to the table below for details. |

#### Debug logging

| Method | Description |
|---|---|
| `setDebugLogging` | SceneGraph API to set debug logging on the ADBMobile SDK. |
| `getDebugLogging` | SceneGraph API to get debug logging from the ADBMobile SDK. |

#### Privacy status / opt-out

| Method | Description |
|---|---|
| `setPrivacyStatus` | SceneGraph API to set privacy status on the ADBMobile SDK. |
| `getPrivacyStatus` | SceneGraph API to get privacy status from the ADBMobile SDK. |

#### Analytics

| Method | Description |
|---|---|
| `trackState` | SceneGraph API to track state on the ADBMobile SDK. |
| `trackAction` | SceneGraph API to track action on the ADBMobile SDK. |
| `trackingIdentifier` | SceneGraph API to get a tracking identifier from the ADBMobile SDK. |
| `userIdentifier` | SceneGraph API to get a user identifier from the ADBMobile SDK. |
| `setUserIdentifier` | SceneGraph API to set the user identifier on the ADBMobile SDK. |
| `getAllIdentifiers` | SceneGraph API that retrieves all user identities known and persisted by the Roku SDK. |

#### CX Enterprise

| Method | Description |
|---|---|
| `visitorSyncIdentifiers` | SceneGraph API to sync CX Enterprise identifiers on the ADBMobile SDK. |
| `visitorMarketingCloudID` | SceneGraph API to get the ECID from the ADBMobile SDK. |

#### Audience Manager

| Method | Description |
|---|---|
| `audienceSubmitSignal` | SceneGraph API to send an audience management signal with trait. |
| `audienceVisitorProfile` | SceneGraph API to get an audience manager visitor profile from the ADBMobile SDK. |
| `audienceDpid` | SceneGraph API to get an audience `Dpid` from the ADBMobile SDK. |
| `audienceDpuuid` | SceneGraph API to get an audience `Dpuuid` from the ADBMobile SDK. |
| `audienceSetDpidAndDpuuid` | SceneGraph API to set audience `Dpid` and `Dpuuid` on the ADBMobile SDK. |

#### MediaHeartbeat

| Method | Description |
|---|---|
| `mediaTrackLoad` | SceneGraph API to load video content for MediaHeartbeat tracking. |
| `mediaTrackStart` | SceneGraph API to start video tracking session using MediaHeartbeat. |
| `mediaTrackUnload` | SceneGraph API to unload video content from MediaHeartbeat tracking. |
| `mediaTrackPlay` | SceneGraph API to track playback of video content. |
| `mediaTrackPause` | SceneGraph API to track paused video content. |
| `mediaTrackComplete` | SceneGraph API to track playback complete for video content. |
| `mediaTrackError` | SceneGraph API to track playback errors. |
| `mediaTrackEvent` | SceneGraph API to track playback events during tracking. For example: ads, chapters. |
| `mediaUpdatePlayhead` | SceneGraph API to send playhead updates to MediaHeartbeat during video tracking. |
| `mediaUpdateQoS` | SceneGraph API to send QoS updates to MediaHeartbeat during video tracking. |

### SceneGraphConstants

| Constant Name | Description |
|---|---|
| `API_RESPONSE` | Used to retrieve the response object from the `adbmobileTask` node's `adbmobileApiResponse` field |
| `DEBUG_LOGGING` | Used as `apiName` for `getDebugLogging` |
| `PRIVACY_STATUS` | Used as `apiName` for `getPrivacyStatus` |
| `TRACKING_IDENTIFIER` | Used as `apiName` for `trackingIdentifier` |
| `USER_IDENTIFIER` | Used as `apiName` for `userIdentifier` |
| `VISITOR_MARKETING_CLOUD_ID` | Used as `apiName` for `visitorMarketingCloudID` |
| `AUDIENCE_VISITOR_PROFILE` | Used as `apiName` for `audienceVisitorProfile` |
| `AUDIENCE_DPID` | Used as `apiName` for `audienceDpid` |
| `AUDIENCE_DPUUID` | Used as `apiName` for `audienceDpuuid` |

### `adbmobileTask` Node

| Field | Type | Default | Usage |
|---|---|---|---|
| `adbmobileApiCall` | `assocarray` | `Invalid` | Do NOT modify this field or let it be used by the application. This field is used by the ADBMobile SceneGraph connector to route API calls via SceneGraph nodes and to fetch responses. This key/field is reserved for AdobeMobileSDK for SceneGraph compatibility. **Important:** Any modifications to this field may result in AdobeMobileSDK functioning incorrectly. |
| `adbmobileApiResponse` | `assocarray` | `Invalid` | Read-only. All APIs executed on AdobeMobileSDK return responses on this field. Register for a callback to listen for updates to this field to receive response objects. The response format is: `{ "apiName": <SceneGraphConstants.API_NAME>, "returnValue": <API_RESPONSE> }`. Response data can also be `invalid` when no value is available. |

### `adbmobile.brs`

#### `getADBMobileConnectorInstance`

API Signature: `ADBMobile().getADBMobileConnectorInstance(adbmobileTask)`  
Input: `adbmobileTask`  
Return Type: `ADBMobileConnector`

#### `sgConstants`

API Signature: `ADBMobile().sgConstants()`  
Input: None  
Return Type: `SceneGraphConstants`

### ADBMobile Constants

| Feature | Constant Name | Description |
|---|---|---|
| Versioning | `version` | Constant for retrieving AdobeMobileLibrary version info |
| Privacy/opt-out | `PRIVACY_STATUS_OPT_IN` | Constant for privacy status opted in |
| | `PRIVACY_STATUS_OPT_OUT` | Constant for privacy status opted out |
| MediaHeartbeat constants | See [roku-setup.md](roku-setup.md) | Use these constants with MediaHeartbeat APIs |
| Standard metadata | See [Standard Metadata Parameters](https://experienceleague.adobe.com/docs/media-analytics/using/implementation/variables/standard-metadata/) | Use these constants to attach standard video/ad metadata in MediaHeartbeat APIs |

Globally defined utility `MediaHeartbeat` APIs on the legacy AdobeMobileLibrary are accessible as-is in the SceneGraph environment because they do not use any BrightScript components that are unavailable in SceneGraph nodes.

### Global Methods for MediaHeartbeat

#### `adb_media_init_mediainfo`

Returns an initialized Media Information object.

```brightscript
Function adb_media_init_mediainfo(name As String, id As String, length As Double, streamType As String) As Object
```

| Variable Name | Type | Description |
|---|---|---|
| `name` | `String` | Media name |
| `id` | `String` | Media ID |
| `length` | `Double` | Media length in seconds |
| `streamType` | `String` | Stream type (for example, `"vod"`, `"live"`) |

#### `adb_media_init_adinfo`

Returns an initialized Ad Information object.

```brightscript
Function adb_media_init_adinfo(name As String, id As String, position As Double, length As Double) As Object
```

| Variable Name | Type | Description |
|---|---|---|
| `name` | `String` | Ad name |
| `id` | `String` | Ad ID |
| `position` | `Double` | Position of the ad within the ad break, starting with 1 |
| `length` | `Double` | Ad length in seconds |

#### `adb_media_init_chapterinfo`

Returns an initialized Chapter Information object.

```brightscript
Function adb_media_init_chapterinfo(name As String, position As Double, length As Double, startTime As Double) As Object
```

| Variable Name | Type | Description |
|---|---|---|
| `name` | `String` | Chapter name |
| `position` | `Double` | Position of the chapter within the content, starting with 1 |
| `length` | `Double` | Chapter length in seconds |
| `startTime` | `Double` | Playhead value at the start of the chapter |

#### `adb_media_init_adbreakinfo`

Returns an initialized AdBreak Information object.

```brightscript
Function adb_media_init_adbreakinfo(name As String, startTime As Double, position As Double) As Object
```

| Variable Name | Type | Description |
|---|---|---|
| `name` | `String` | Ad break name (for example, `"pre-roll"`, `"mid-roll"`, `"post-roll"`) |
| `startTime` | `Double` | Playhead value at the start of the ad break |
| `position` | `Double` | Position of the ad break within the content, starting with 1 |

#### `adb_media_init_qosinfo`

Returns an initialized QoS Information object.

```brightscript
Function adb_media_init_qosinfo(bitrate As Double, startupTime As Double, fps As Double, droppedFrames As Double) As Object
```

| Variable Name | Type | Description |
|---|---|---|
| `bitrate` | `Double` | Current bitrate in bits per second |
| `startupTime` | `Double` | Startup time in seconds |
| `fps` | `Double` | Frames per second |
| `droppedFrames` | `Double` | Number of dropped frames |

## Implementation

1. **Download the Roku Library** — Download the [latest Roku library](https://github.com/Adobe-Marketing-Cloud/media-sdks/releases/tag/roku-v2.2.2).

1. **Set up your development environment**

    1. Copy `adbmobile.brs` (AdobeMobileLibrary) into your `pkg:/source/` directory.

    1. For SceneGraph support, copy `adbmobileTask.brs` and `adbMobileTask.xml` into your `pkg:/components/` directory.

1. **Initialize**

    1. Import `adbmobile.brs` into your Scene.

       ```brightscript
       <script type="text/brightscript" uri="pkg:/source/adbmobile.brs" />
       ```

    1. Create an instance of `adbmobileTask` node into your Scene.

       ```brightscript
       m.adbmobileTask = createObject("roSGNode", "adbmobileTask")
       ```

    1. Get an instance of `adbmobile` connector for SceneGraph using the `adbmobileTask` instance.

       ```brightscript
       m.adbmobile = ADBMobile().getADBMobileConnectorInstance(m.adbmobileTask)
       ```

    1. Get `adbmobile` SG constants.

       ```brightscript
       m.adbmobileConstants = m.adbmobile.sceneGraphConstants()
       ```

    1. Register a callback for receiving response objects for all `AdbMobile` API calls.

       ```brightscript
       m.adbmobileTask.ObserveField(m.adbmobileConstants.API_RESPONSE, "onAdbmobileApiResponse")

       ' Sample implementation of the callback
       function onAdbmobileApiResponse() as void
           responseObject = m.adbmobileTask[m.adbmobileConstants.API_RESPONSE]

           if responseObject <> invalid
               methodName = responseObject.apiName
               retVal = responseObject.returnValue

               if methodName = m.adbmobileConstants.DEBUG_LOGGING
                   if retVal
                       print "API Response: DEBUG LOGGING: " + "True"
                   else
                       print "API Response: DEBUG LOGGING: " + "False"
                   endif
               else if methodName = m.adbmobileConstants.PRIVACY_STATUS
                   print "API Response: PRIVACY STATUS: " + retVal
               else if methodName = m.adbmobileConstants.TRACKING_IDENTIFIER
                   if retVal <> invalid
                       print "API Response: TRACKING IDENTIFIER: " + retVal
                   else
                       print "API Response: TRACKING IDENTIFIER: " + "invalid"
                   endif
               else if methodName = m.adbmobileConstants.USER_IDENTIFIER
                   if retVal <> invalid
                       print "API Response: USER IDENTIFIER: " + retVal
                   else
                       print "API Response: USER IDENTIFIER: " + "invalid"
                   endif
               else if methodName = m.adbmobileConstants.VISITOR_MARKETING_CLOUD_ID
                   if retVal <> invalid
                       print "API Response: MCID: " + retVal
                   else
                       print "API Response: MCID: " + "invalid"
                   endif
               else if methodName = m.adbmobileConstants.AUDIENCE_DPID
                   if retVal <> invalid
                       print "API Response: AUDIENCE DPID: " + retVal
                   else
                       print "API Response: AUDIENCE DPID: " + "invalid"
                   endif
               else if methodName = m.adbmobileConstants.AUDIENCE_DPUUID
                   if retVal <> invalid
                       print "API Response: AUDIENCE DPUUID: " + retVal
                   else
                       print "API Response: AUDIENCE DPUUID: " + "invalid"
                   endif
               else if methodName = m.adbmobileConstants.AUDIENCE_VISITOR_PROFILE
                   if retVal <> invalid
                       print "API Response: AUDIENCE VISITOR PROFILE: Valid Object"
                   else
                       print "API Response: AUDIENCE VISITOR PROFILE: " + "invalid"
                   endif
               endif
           endif
       end function
       ```

## Sample implementation

**Sample API calls on legacy SDK**

```brightscript
' get an instance of SDK
m.adbmobile = ADBMobile()

' execute setter APIs
m.adbmobile.setDebugLogging(true)

' execute getter APIs
debugLogging = m.adbmobile.getDebugLogging()
```

**Sample API calls on SG SDK**

```brightscript
' create adbmobileTask instance
m.adbmobileTask = createObject("roSGNode", "adbmobileTask")

' get an instance of SDK using task instance
m.adbmobile = ADBMobile().getADBMobileConnectorInstance(m.adbmobileTask)
m.adbmobileConstants = m.adbmobile.sceneGraphConstants()

' execute setter APIs
m.adbmobile.setDebugLogging(true)

' execute getter APIs
m.adbmobileTask.ObserveField(m.adbmobileConstants.API_RESPONSE, "onAdbmobileApiResponse")
m.adbmobile.getDebugLogging()

' listen for return data in registered callbacks
function onAdbmobileApiResponse() as void
    responseObject = m.adbmobileTask[m.adbmobileConstants.API_RESPONSE]

    if responseObject <> invalid
        methodName = responseObject.apiName
        retVal = responseObject.returnValue

        if methodName = m.adbmobileConstants.DEBUG_LOGGING
            if retVal
                print "API Response: DEBUG LOGGING: " + "True"
            else
                print "API Response: DEBUG LOGGING: " + "False"
            endif
        endif
    endif
end function
```
