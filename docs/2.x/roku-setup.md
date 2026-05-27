# Set up Roku

> **End of Support:** The Media SDK 2.x for Roku reached end of support on August 31, 2021. For the recommended implementation, see the [AEP Roku SDK](https://github.com/adobe/aepsdk-roku). For additional information, see [Media Analytics SDK End-of-Support FAQs](https://experienceleague.adobe.com/docs/discontinued/using/media-sdk.html).

Roku SDK 2.x for CX Enterprise lets you measure Roku applications written in BrightScript, leverage and collect audience data through audience management, and measure video engagement through Video events.

## Prerequisites

* **Obtain valid configuration parameters for Adobe streaming media services**

   These parameters can be obtained from an Adobe representative after you set up your account for the Adobe Streaming Media Collection Add-on or the Adobe Analytics for Streaming Media Add-on.

* **Include the following APIs in your media player**

    * _An API to subscribe to player events_ — The Media SDK requires that you call a set of simple APIs when events occur in your player.
    * _An API that provides player information_ — This information includes details such as the media name and the play head position.

## SDK Implementation

1. Add your downloaded Roku library to your project.

    1. The `AdobeMobileLibrary-2.*-Roku.zip` download file consists of the following software components:

       * `adbmobile.brs`: This library file will be included in your Roku app source folder.
       * `ADBMobileConfig.json`: This SDK configuration file is customized for your app.

    1. Add the library file and JSON config file to your project source.

       The JSON that is used to configure Adobe Mobile has an exclusive key for media analytics called `mediaHeartbeat`. This is where the configuration parameters for media analytics belong.

       > **Tip:** A sample `ADBMobileConfig` JSON file is provided with the package. Contact your Adobe representatives for the settings.

       For example:

       ```json
       {
         "version":"1.0",
         "analytics":{
           "rsids":"",
           "server":"",
           "charset":"UTF-8",
           "ssl":true,
           "offlineEnabled":false,
           "lifecycleTimeout":30,
           "batchLimit":50,
           "privacyDefault":"optedin",
           "poi":[ ]
         },
         "marketingCloud":{
           "org":""
         },
         "target":{
           "clientCode":"",
           "timeout":5
         },
         "audienceManager":{
           "server":""
         },
         "acquisition":{
           "server":"example.com",
           "appid":"sample-app-id"
         },
         "mediaHeartbeat":{
           "server":"example.com",
           "publisher":"sample-publisher",
           "channel":"sample-channel",
           "ssl":true,
           "ovp":"sample-ovp",
           "sdkVersion":"sample-sdk",
           "playerName":"roku"
         }
       }
       ```

       | Config Parameter | Description |
       | --- | --- |
       | `server` | URL of the tracking endpoint on the backend. |
       | `publisher` | Content publisher unique identifier. |
       | `channel` | Name of the content distribution channel. |
       | `ssl` | Whether SSL should be used for tracking calls. |
       | `ovp` | Name of the video player provider. |
       | `sdkVersion` | Current version of the app/SDK. |
       | `playerName` | Name of the player. |

       > **Important:** If `mediaHeartbeat` is incorrectly configured, the media module (VHL) enters an error state and will stop sending tracking calls.

1. Configure Experience Cloud Visitor ID.

    The Experience Cloud Visitor ID service provides a universal Visitor ID across CX Enterprise. The Visitor ID service is required by Video events and other Marketing Cloud integrations.

    Verify that your `ADBMobileConfig` config contains your `marketingCloud` organization ID.

    ```json
    "marketingCloud": {
        "org": "YOUR-MCORG-ID"
    }
    ```

    IMS organization IDs uniquely identify each client company in the Adobe Marketing Cloud and appear similar to the following value: `016D5C175213CCA80A490D05@AdobeOrg`.

    > **Important:** Ensure that you include `@AdobeOrg`.

    After the configuration is complete, a visitor ID is generated and is included on all hits. Other Visitor IDs, such as `custom` and `automatically-generated`, continue to be sent with each hit.

    **Experience Cloud Visitor ID Service Methods**

    > **Tip:** Experience Cloud Visitor ID methods are prefixed with `visitor`.

    | Method | Description |
    | --- | --- |
    | `visitorMarketingCloudID` | Retrieves the visitor ID from the visitor ID service. `ADBMobile().visitorMarketingCloudID()` |
    | `visitorSyncIdentifiers` | With the visitor ID, you can set additional customer IDs for the same visitor with a customer type identifier. This method corresponds to `setCustomerIDs`. For example: `identifiers={}` / `identifiers["idType"]="idValue"` / `ADBMobile().visitorSyncIdentifiers(identifiers)` |
    | `setAdvertisingIdentifier` | Sets the Roku ID for Advertising (RIDA) on the SDK. Get the RIDA using the Roku SDK [getRIDA()](https://developer.roku.com/docs/references/brightscript/interfaces/ifdeviceinfo.md#getrida-as-dynamic) API. |
    | `getAllIdentifiers` | Returns all identifiers stored by the SDK including Analytics, Visitor, Audience Manager, and custom identifiers. `identifiers = ADBMobile().getAllIdentifiers()` |

    **Debug Logging**

    | Method | Description |
    | --- | --- |
    | `setDebugLogging` | Enables or disables debug logging. `ADBMobile().setDebugLogging(true)` |
    | `getDebugLogging` | Returns `true` if debug logging is enabled. `isDebugLoggingEnabled = ADBMobile().getDebugLogging()` |

    **Privacy Status**

    | Constant | Description |
    | --- | --- |
    | `PRIVACY_STATUS_OPT_IN` | Passed to `setPrivacyStatus` to opt in. `optInString = ADBMobile().PRIVACY_STATUS_OPT_IN` |
    | `PRIVACY_STATUS_OPT_OUT` | Passed to `setPrivacyStatus` to opt out. `optOutString = ADBMobile().PRIVACY_STATUS_OPT_OUT` |

    | Method | Description |
    | --- | --- |
    | `setPrivacyStatus` | Sets the privacy status. `ADBMobile().setPrivacyStatus(ADBMobile().PRIVACY_STATUS_OPT_IN)` |
    | `getPrivacyStatus` | Gets the current privacy status. `privacyStatus = ADBMobile().getPrivacyStatus()` |

    > **Important:** Ensure that you call `processMessages` and `processMediaMessages` in the main event loop every 250 ms to ensure that the SDK sends pings properly.

    | Method | Description |
    | --- | --- |
    | `processMessages` | Passes Analytics events to the SDK. `ADBMobile().processMessages()` |
    | `processMediaMessages` | Passes Media events to the SDK. `ADBMobile().processMediaMessages()` |
