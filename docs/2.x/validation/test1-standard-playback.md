# Test 1: Standard playback

This test case validates general playback and sequencing.

Media Analytics implementations include two types of tracking calls:
* **Calls made directly to your Adobe Analytics (AppMeasurement) server**: These calls occur on "Media Start" and "Ad Start" events.
* **Calls made to the Media Analytics (heartbeats) server**: These include in-band and out-of-band calls:
   * In-band: The SDK sends timed play calls or "pings" at 10-second intervals during content playback, and at one-second intervals during ads.
   * Out-of-band: These calls can happen at any point, and include Pause, Buffering, errors, content complete, ad complete, etc.

> Media tracking behaves the same across all platforms.

## Test procedure

Complete and record the following actions (in order):

1. **Load the page or app**

    **Tracking Servers** (For all website and mobile apps):

    * **Adobe Analytics (AppMeasurement) server**: An RDC tracking server or CNAME that resolves to an RDC tracking server is required for the Experience Cloud Visitor ID service. The Adobe Analytics tracking server should end in "`.sc.omtrdc.net`" or be a CNAME.

    * **Media Analytics (Heartbeats) server**: This server always has the format "`[namespace].hb.omtrdc.net`", where `[namespace]` specifies your company name. This name is provided by Adobe.

    You need to validate certain key variables that are universal across all tracking calls:

    **Visitor ID (`mid`)**: The `mid` variable is used to capture the value set in the AMCV cookie. The `mid` variable is the primary identification value for both websites and mobile apps, and also indicates that the Experience Cloud Visitor ID service is set up properly. It is found in both Adobe Analytics (AppMeasurement) and Media Analytics (heartbeats) calls.

    * **Adobe Analytics Start call**

       | Parameter | Value (sample) |
       |---|---|
       | `pev2` | ms_s |
       | `mid` | 30250035503789876473484580554595324209 |

    * **Website Page call**

       | Parameter | Value (sample) |
       |---|---|
       | `mid` | 30250035503789876473484580554595324209 |

    * **Lifecycle call**

       | Parameter | Value (sample) |
       |---|---|
       | `pev2` | ADBINTERNAL:Lifecycle |
       | `mid` | 30250035503789876473484580554595324209 |

    * **Media Analytics Start call**

       | Parameter | Value (sample) |
       |---|---|
       | `s:event:type` | start |

       > On Media Analytics Start calls (`s:event:type=start`) the `mid` values may not be present. This is OK. They may not appear until the Media Analytics Play calls (`s:event:type=play`).

    * **Media Analytics Play call**

       | Parameter | Value (sample) |
       |---|---|
       | `s:event:type` | play |
       | `s:user:mid` | 30250035503789876473484580554595324209 |

1. **Start the media player**

    When the media player starts, the Media SDK sends the key calls to the two servers in the following order:

    1. Adobe Analytics server — Start call
    1. Media Analytics server — Start call
    1. Media Analytics server — "Adobe Analytics Start call requested"

    The first two calls above contain additional metadata and variables. For call parameters and metadata, see [Test call details.](./test-call-details.md#start-the-media-player)

    The third call above tells the Media Analytics server that the Media SDK requested that the Adobe Analytics Start call (`pev2=ms_s`) be sent to the Adobe Analytics server.

1. **View ad break if available**

    * **Ad Start**

    When the ad starts, the following key calls are sent in the following order:

    1. Adobe Analytics server — Ad Start call
    1. Media Analytics server — Ad Start call
    1. Media Analytics server — "Adobe Analytics Ad Start call requested"

    The first two calls contain additional metadata and variables. For call parameters and metadata, see [Test call details.](./test-call-details.md#view-ad-playback)

    The third call tells the Media Analytics server that the Media SDK requested that the Adobe Analytics Ad Start call (`pev2=msa_s`) be sent to the Adobe Analytics server.

    * **Ad Play**

       During ad playback, the Media Analytics SDK sends play events of type "ad" to the Media Analytics server every second.

    * **Ad Complete**

       At the 100% point of an ad, a Media Analytics Complete call should be sent.

1. **Pause ad playback for 30 seconds, if available.** **Ad Pause**

    During Ad Pause, Media Analytics heartbeat or "ping" calls are sent by the SDK to the Media Analytics server every second.

    > The playhead value should remain constant during the pause.

    For call parameters and metadata, see [Test call details.](./test-call-details.md#view-ad-playback)

1. **Play main content for 10 seconds uninterrupted.** **Content Play**

    During main content playback, the Media SDK sends heartbeats (Play calls) to the Media Analytics server every 10 seconds.

    Notes:

     * The playhead position should increment by 10 with every Play call.
     * The `l:event:duration` value represents the number of milliseconds since the last tracking call and should be roughly the same value on each 10 second call.

       For call parameters and metadata, see [Test call details.](./test-call-details.md#play-main-content)

1. **Pause during playback for at least 30 seconds.** On pause of the media player, pause event calls will be sent by the SDK to the Media Analytics server every 10 seconds. After pause ends, the play events should resume.

    For call parameters and metadata, see [Test call details.](./test-call-details.md#pause-main-content)

1. **Seek/scrub media.** On scrubbing of media playhead, no special tracking calls are sent, however, when media playback resumes after scrubbing, the playhead value should reflect the new position within the main content.

1. **Replay media (VOD only).** When media is replayed, a new set of Media Start calls should be sent (as if it were a fresh start).

1. **View next media in playlist.** On Media Start of the next media in a playlist, a new set of Media Start calls should be sent.

1. **Switch media or stream.** When switching live streams, a Media Analytics complete call for the first stream should not be sent. The Media Start calls and Play calls should begin with the new show and stream name, and with the correct playhead and duration values for the new show.
