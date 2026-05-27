# VOD Playback with Pre-roll Ads

> **End of Support:** The Media SDK 2.x reached end of support on August 31, 2021. These scenario code examples apply to the legacy 2.x `MediaHeartbeat` API only.

## Scenario

In this scenario, pre-roll ads have been inserted before the main content. Unless specified, the network calls are the same as the calls in the [VOD playback with no ads](vod-no-intrs-details.md) scenario. The network calls happen at the same time, but the payload is different.

| Trigger | Heartbeat method | Network calls | Notes |
| --- | --- | --- | --- |
| The user clicks **Play** | `trackSessionStart` | Analytics Content Start, Heartbeat Content Start | The measurement library does not know that there is a pre-roll ad, so these network calls are still identical to the [VOD playback with no ads](vod-no-intrs-details.md) scenario. |
| The ad starts. | `trackEvent:AdBreakStart` / `trackEvent:AdStart` | Analytics Ad Start, Heartbeat Ad Start | |
| The frame of ad #1 is played. | `trackPlay` | Heartbeat Ad Play | The ad content plays before main content, and the heartbeats start when the ad starts. |
| The ad is played. | | Ad Heartbeats | |
| Ad #1 completes playing. | `trackEvent:trackAdComplete` | Heartbeat Ad Complete | The end of the ad is reached. |
| The first frame of ad #2 is played. | `trackEvent:AdStart` | Analytics Ad Start, Heartbeat Ad Start | |
| The ad plays. | | Ad Heartbeats | |
| Ad #2 completes playing. | `trackEvent:trackAdComplete` / `trackEvent:AdBreakComplete` | Heartbeat Ad Complete | The end of the ad and the pod is reached. |
| The content plays. | | Content Heartbeats | This network call is identical to the [VOD playback with no ads](vod-no-intrs-details.md) scenario. |
| The content is complete. | `trackComplete` | Heartbeat Content Complete | This network call is identical to the [VOD playback with no ads](vod-no-intrs-details.md) scenario. |
| The session is over | `trackSessionEnd` | | `SessionEnd` |

## Parameters

When ad playback begins, a `Heartbeat Ad Start` call is sent. If the beginning of the ad does not coincide with the 10-second timer, the `Heartbeat Ad Start` call is delayed by a few seconds, and the call goes to the next 10-second interval. When this happens, a `Content Heartbeat` goes out in the same interval, and you can differentiate between the two calls by looking at the event type and the asset type:

### Heartbeat Ad Start

| Parameter | Value | Notes |
|---|---|---|
| `s:event:type` | `start` | |
| `s:asset:type` | `ad` | |

Ads follow the same basic model as `Content Heartbeats`, so the `Ad Play` call is similar to the `Content Play` call.

### Heartbeat Ad Play Call

| Parameter | Value | Notes |
|---|---|---|
| `s:event:type` | `play` | |
| `s:asset:type` | `ad` | |

### Ad Heartbeats

| Parameter | Value | Notes |
|---|---|---|
| `s:event:type` | `play` | |
| `s:asset:type` | `ad` | |
| `s:asset:ad_id` | &lt;ad ID&gt; | |
| `s:asset:pod_id` | &lt;ad pod ID&gt; | |

### Heartbeat Ad Complete Call

| Parameter | Value | Notes |
|---|---|---|
| `s:event:type` | `complete` | |
| `s:asset:type` | `ad` | |

## Sample code for a pre-roll ad break

In this scenario, the VOD consists of a pre-roll ad, a second pre-roll ad, and then the content is played.

![](../../../help/use-cases/scenarios/assets/preroll-regular-playback.png)

### Android

To view this scenario in Android, set up the following code:

```java
// Set up  mediaObject 
MediaObject mediaInfo = MediaHeartbeat.createMediaObject( 
    Configuration.MEDIA_NAME,  
    Configuration.MEDIA_ID,  
    Configuration.MEDIA_LENGTH,  
    MediaHeartbeat.StreamType.VOD 
); 
 
HashMap<String, String> videoMetadata = new HashMap<String, String>(); 
videoMetadata.put(CUSTOM_KEY_1, CUSTOM_VAL_1); 
videoMetadata.put(CUSTOM_KEY_2, CUSTOM_VAL_2); 
 
// 1. Call trackSessionStart() when the user clicks Play or if autoplay is used,  
//    i.e., there's an intent to start playback.  
_mediaHeartbeat.trackSessionStart(mediaInfo, videoMetadata); 
 
...... 
...... 
 
// Pre-roll 
MediaObject adBreakInfo =  
  MediaHeartbeat.createAdBreakObject(ADBREAK_NAME,  
                                     ADBREAK_POSITION,  
                                     ADBREAK_START_TIME); 
MediaObject adInfo =  
  MediaHeartbeat.createAdObject(AD_NAME,  
                                AD_ID,  
                                AD_POSITION,  
                                AD_LENGTH); 
 
// Context ad data 
HashMap<String, String> adMetadata = new HashMap<String, String>(); 
adMetadata.put(CUSTOM_KEY_1, CUSTOM_VAL_1); 
adMetadata.put(CUSTOM_KEY_2, CUSTOM_VAL_2); 
 
// 2. Track the MediaHeartbeat.Event.AdBreakStart event when the pre-roll pod starts  
//    to play. Note that since this is a pre-roll, call must track the 
//    "MediaHeartbeat.Event.AdBreakStart" event before you call trackPlay().  
_mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdBreakStart, adBreakInfo, null); 
 
....... 
....... 
 
// 3. Track the MediaHeartbeat.Event.AdStart event when the pre-roll pod's ad starts  
//    to play. Note that since this is a pre-roll, you must track the  
//    "MediaHeartbeat.Event.AdStart" event before you call trackPlay(). 
_mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdStart, adInfo, adMetadata); 
 
....... 
....... 
 
// 4. Call trackPlay() when the playback actually starts, i.e., when the first frame  
//    of the ad video is rendered on the screen. 
_mediaHeartbeat.trackPlay(); 
 
....... 
....... 
 
// 5. Track the MediaHeartbeat.Event.AdComplete event when the ad reaches the end,  
//    i.e., when the ad completes and finishes playing.  
_mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdComplete, null, null); 
 
....... 
....... 
 
// 6. Track the MediaHeartbeat.Event.AdStart event when the pre-roll pod's second ad  
//    starts to play. 
_mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdStart, adInfo, adMetadata); 
 
....... 
....... 
 
// 7. Track the MediaHeartbeat.Event.AdComplete event when the second ad reaches the  
//    end, i.e., the second ad completes and finishes playing. 
_mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdComplete, null, null); 
 
....... 
....... 
 
// 8. Track the MediaHeartbeat.Event.AdBreakComplete event when all of the ads in the  
//    pod finish playing.  
_mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdBreakComplete, null, null); 
 
....... 
....... 
 
// 9. Call trackComplete() when the playback reaches the end, i.e., when the video 
//    completes and finishes playing. 
_mediaHeartbeat.trackComplete(); 
 
........ 
........ 
 
// 10. Call trackSessionEnd() when the playback session is over. This method must be  
//     called even if the user does not watch the video to completion.  
_mediaHeartbeat.trackSessionEnd(); 
 
........ 
........ 
```

### iOS

To view this scenario in iOS, set up the following code:

```
//  Set up mediaObject 
ADBMediaObject *mediaObject =  
  [ADBMediaHeartbeat createMediaObjectWithName:MEDIA_NAME  
                     length:MEDIA_LENGTH  
                     streamType:ADBMediaHeartbeatStreamTypeVOD]; 
    
NSMutableDictionary *videoContextData = [[NSMutableDictionary alloc] init]; 
[videoContextData setObject:CUSTOM_VAL_1 forKey:CUSTOM_KEY_1]; 
[videoContextData setObject:CUSTOM_VAL_2 forKey:CUSTOM_KEY_2]; 
   
// 1. Call trackSessionStart when the user clicks Play or if autoplay is used,  
//    i.e., there is an intent to start playback. 
[_mediaHeartbeat trackSessionStart:mediaObject data:videoContextData]; 
....... 
....... 
  
// Pre-roll 
ADBMediaObject *adBreakInfo =  
  [ADBMediaHeartbeat createAdBreakObjectWithName:AD_BREAK_NAME  
                     position:AD_BREAK_POSITION  
                     startTime:AD_BREAK_START_TIME]; 
ADBMediaObject *adInfo =  
  [ADBMediaHeartbeat createAdObjectWithName:AD_NAME  
                     adId:AD_ID  
                     position:AD_POSITION  
                     length:AD_LENGTH]; 
  
// context ad data 
NSMutableDictionary *adDictionary = [[NSMutableDictionary alloc] init]; 
[adDictionary setObject:@"custom-val1" forKey:@"custom-key1"]; 
[adDictionary setObject:@"custom-val2" forKey:@"custom-key2"]; 
  
// 2. Track the ADBMediaHeartbeatEventAdBreakStart event when the pre-roll pod  
//    starts to play. Note that since this is a pre-roll, you must track the  
//    "ADBMediaHeartbeatEventAdBreakStart" event before you call trackPlay. 
[_mediaHeartbeat trackEvent:ADBMediaHeartbeatEventAdBreakStart  
                 mediaObject:adBreakObject  
                 data:nil]; 
....... 
....... 
  
// 3. Track the ADBMediaHeartbeatEventAdStart event when the pre-roll pod's  
//    ad starts to play. Note that since this is a pre-roll, you must track  
//    the "ADBMediaHeartbeatEventAdStart" event before you call trackPlay. 
[_mediaHeartbeat trackEvent:ADBMediaHeartbeatEventAdStart  
                  mediaObject:adObject  
                  data:adDictionary]; 
....... 
....... 
  
// 4. Call trackPlay when the playback actually starts, i.e., when the   
//    first frame of the main content is rendered on the screen. 
[_mediaHeartbeat trackPlay]; 
....... 
....... 
  
// 5. Track the ADBMediaHeartbeatEventAdComplete event when the ad reaches  
//    the end, i.e., when the video completes and finishes playing. 
[_mediaHeartbeat trackEvent:ADBMediaHeartbeatEventAdComplete  
                  mediaObject:nil  
                  data:nil]; 
....... 
....... 
  
// 6. Track the ADBMediaHeartbeatEventAdStart event when the pre-roll pod's  
//    second ad starts to play. 
[_mediaHeartbeat trackEvent:ADBMediaHeartbeatEventAdStart  
                  mediaObject:adBreakObject  
                  data:nil]; 
....... 
....... 
  
// 7. Track the ADBMediaHeartbeatEventAdComplete event when the second ad  
//    reaches the end, i.e., it completes and finishes playing. 
[_mediaHeartbeat trackEvent:ADBMediaHeartbeatEventAdComplete  
                  mediaObject:nil  
                  data:nil]; 
....... 
....... 
  
// 8. Track the ADBMediaHeartbeatEventAdBreakComplete event when all the  
//    ads in the pod finish playing. 
[_mediaHeartbeat trackEvent:ADBMediaHeartbeatEventAdBreakComplete  
                  mediaObject:adBreakObject  
                  data:nil]; 
....... 
....... 
  
// 9. Call trackComplete when the playback reaches the end, i.e., when the  
//    video completes and finishes playing. 
[_mediaHeartbeat trackComplete]; 
....... 
....... 
  
// 10. Call trackSessionEnd when the playback session is over. This method  
//     must be called even if the user does not watch the video to completion. 
[_mediaHeartbeat trackSessionEnd]; 
....... 
....... 
```

### JavaScript

To view this scenario in JavaScript, enter the following text:

```js
// Set up mediaObject 
var mediaInfo =  
  MediaHeartbeat.createMediaObject(Configuration.MEDIA_NAME,  
                                   Configuration.MEDIA_ID,  
                                   Configuration.MEDIA_LENGTH,  
                                   MediaHeartbeat.StreamType.VOD); 
var videoMetadata = { 
    CUSTOM_KEY_1 : CUSTOM_VAL_1,  
    CUSTOM_KEY_2 : CUSTOM_VAL_2, 
    CUSTOM_KEY_3 : CUSTOM_VAL_3 
}; 
 
// 1. Call trackSessionStart() when Play is clicked or if autoplay is used,  
//    i.e., there's an intent to start playback. 
this._mediaHeartbeat.trackSessionStart(mediaInfo, videoMetadata); 
 
...... 
...... 
 
// Preroll 
var adBreakInfo =  
  MediaHeartbeat.createAdBreakObject(ADBREAK_NAME, ADBREAK_POSITION, ADBREAK_START_TIME); 
var adInfo =  
  MediaHeartbeat.createAdObject(AD_NAME, AD_ID, AD_POSITION, AD_LENGTH); 
 
// Custom ad metadata 
var adMetadata = { 
    CUSTOM_AD_KEY_1 : CUSTOM_AD_VAL_1,  
    CUSTOM_AD_KEY_2 : CUSTOM_AD_VAL_2 
}; 
 
// 2. Track the MediaHeartbeat.Event.AdBreakStart event when the preroll pod starts to play.  
//    Note that since this is a preroll, track the MediaHeartbeat.Event.AdBreakStart  
//    event before you call trackPlay(). 
this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdBreakStart, adBreakInfo, null); 
 
....... 
....... 
 
// 3. Track the MediaHeartbeat.Event.AdStart event when the preroll pod's ad starts to play.  
//    Note that since this is a preroll, track the MediaHeartbeat.Event.AdStart event before  
//    you call trackPlay(). 
this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdStart, adInfo, adMetadata); 
 
....... 
....... 
 
// 4. Call trackPlay() when the main content actually starts, i.e., when the  
//    first frame of the video content is rendered on the screen.  
this._mediaHeartbeat.trackPlay(); 
 
....... 
....... 
 
// 5. Track event MediaHeartbeat.Event.AdComplete when the ad reaches the end,  
//    i.e., when it completes and finishes playing. 
this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdComplete); 
 
....... 
....... 
 
// 6. Track the MediaHeartbeat.Event.AdStart event when the preroll pod's second  
//    ad starts to play. 
this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdStart, adInfo, adMetadata); 
 
....... 
....... 
 
// 7. Track the MediaHeartbeat.Event.AdComplete event when the second ad reaches  
//    the end, i.e., when it completes and finishes playing. 
this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdComplete); 
 
....... 
....... 
 
// 8. Track the MediaHeartbeat.Event.AdBreakComplete event when all the ads  
//    in the pod finish playing. 
this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdBreakComplete); 
 
....... 
....... 
 
// 9. Call trackComplete() when the playback reaches the end, i.e., when it 
//    completes and finishes playing.  
this._mediaHeartbeat.trackComplete(); 
 
// 10. Call trackSessionEnd() when the playback session is over. This method must  
//     be called even if the user does not watch the video to completion. 
this._mediaHeartbeat.trackSessionEnd(); 
 
....... 
....... 
```

## Sample code for multiple ad breaks

In this scenario, VOD content is played back with a pre-roll ad, the content, a mid-roll ad, the content, and a post-roll ad.

![](../../../help/use-cases/scenarios/assets/ad-content-regular-playback.png)

### Android

To view this scenario in Android, set up the following code:

```java
// Set up mediaObject 
MediaObject mediaInfo = MediaHeartbeat.createMediaObject( 
    Configuration.MEDIA_NAME,  
    Configuration.MEDIA_ID,  
    Configuration.MEDIA_LENGTH,  
    MediaHeartbeat.StreamType.VOD 
); 
 
HashMap<String, String> videoMetadata = new HashMap<String, String>(); 
videoMetadata.put(CUSTOM_KEY_1, CUSTOM_VAL_1); 
videoMetadata.put(CUSTOM_KEY_2, CUSTOM_VAL_2); 
 
// 1. Call trackSessionStart() when the user clicks Play or if autoplay is used,  
//   i.e., there's an intent to start playback. 
_mediaHeartbeat.trackSessionStart(mediaInfo, videoMetadata); 
 
...... 
...... 
 
// Pre-roll 
MediaObject adBreakInfo =  
  MediaHeartbeat.createAdBreakObject(ADBREAK_NAME,  
                                     ADBREAK_POSITION,  
                                     ADBREAK_START_TIME); 
MediaObject adInfo = MediaHeartbeat.createAdObject(AD_NAME,  
                                                   AD_ID,  
                                                   AD_POSITION,  
                                                   AD_LENGTH); 
 
// Context ad data 
HashMap<String, String> adMetadata = new HashMap<String, String>(); 
adMetadata.put(CUSTOM_KEY_1, CUSTOM_VAL_1); 
adMetadata.put(CUSTOM_KEY_2, CUSTOM_VAL_2); 
 
// 2. Track the MediaHeartbeat.Event.AdBreakStart event when the pre-roll pod  
//    starts to play. Note that since this is a pre-roll, you must track the  
//    "MediaHeartbeat.Event.AdBreakStart" event before you call trackPlay().  
_mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdBreakStart, adBreakInfo, null); 
 
....... 
....... 
 
// 3. Track the MediaHeartbeat.Event.AdStart event when the pre-roll pod's ad  
//    starts to play. Note that since this is a pre-roll, you must track the  
//    "MediaHeartbeat.Event.AdStart" event before you call trackPlay().  
_mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdStart, adInfo, adMetadata); 
 
....... 
....... 
 
// 4. Call trackPlay() when the playback actually starts, i.e., when the first  
//    frame of the main content is rendered on the screen.  
_mediaHeartbeat.trackPlay(); 
 
....... 
....... 
 
// 5. Track the MediaHeartbeat.Event.AdComplete event when the ad reaches the end,  
//    i.e., when the ad completes and finishes playing.  
_mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdComplete, null, null); 
 
....... 
....... 
 
// 6. Track the MediaHeartbeat.Event.AdBreakComplete event when all of the ads in  
//    the pod finish playing. 
_mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdBreakComplete, null, null); 
 
....... 
....... 
 
// Mid-roll 
MediaObject midrollAdBreakInfo =  
  MediaHeartbeat.createAdBreakObject(MIDROLL_BREAK_NAME,  
                                     MIDROLL_BREAK_POSITION,  
                                     MIDROLL_BREAK_START_TIME); 
MediaObject midrollAdInfo =  
  MediaHeartbeat.createAdObject(MIDROLL_AD_NAME,  
                                MIDROLL_AD_ID,  
                                MIDROLL_AD_POSITION,  
                                MIDROLL_AD_LENGTH); 
 
// Context ad data 
HashMap<String, String> midrollAdMetadata = new HashMap<String, String>(); 
midrollAdMetadata.put(CUSTOM_KEY_1, CUSTOM_VAL_1); 
midrollAdMetadata.put(CUSTOM_KEY_2, CUSTOM_VAL_2); 
 
// 7. Track the MediaHeartbeat.Event.AdBreakStart event when the mid-roll pod  
//    starts to play.  
_mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdBreakStart, midrollAdBreakInfo, null); 
 
....... 
....... 
 
// 8. Track the MediaHeartbeat.Event.AdStart event when the mid-roll pod's ad  
//    starts to play.  
_mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdStart, midrollAdInfo, midrollAdMetadata); 
 
....... 
....... 
 
// 9. Track the MediaHeartbeat.Event.AdComplete event when the ad reaches the end,  
//    i.e., when the ad completes and finishes playing.  
_mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdComplete, null, null); 
 
....... 
....... 
 
// 10. Track the MediaHeartbeat.Event.AdBreakComplete event when all the ads in the  
//     mid-roll pod finish playing.  
_mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdBreakComplete, null, null); 
 
....... 
....... 
 
// Post-roll 
MediaObject postrollAdBreakInfo =  
  MediaHeartbeat.createAdBreakObject(POSTROLL_BREAK_NAME,  
                                     POSTROLL_BREAK_POSITION,  
                                     POSTROLL_BREAK_START_TIME); 
MediaObject postrollAdInfo =  
  MediaHeartbeat.createAdObject(POSTROLL_AD_NAME,  
                                POSTROLL_AD_ID,  
                                POSTROLL_AD_POSITION,  
                                POSTROLL_AD_LENGTH); 
 
// Context ad data 
HashMap<String, String> postrollAdMetadata = new HashMap<String, String>(); 
postrollAdMetadata.put(CUSTOM_KEY_1, CUSTOM_VAL_1); 
postrollAdMetadata.put(CUSTOM_KEY_2, CUSTOM_VAL_2); 
 
// 11. Track the MediaHeartbeat.Event.AdBreakStart event when the post-roll pod  
//     starts to play.  
_mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdBreakStart, postrollAdBreakInfo, null); 
 
....... 
....... 
 
// 12. Track the MediaHeartbeat.Event.AdStart event when the post-roll pod's  
//     ad starts to play.  
_mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdStart, postrollAdInfo, postrollAdMetadata); 
 
....... 
....... 
 
// 13. Track the MediaHeartbeat.Event.AdComplete event when the ad reaches the  
//     end, i.e., when the ad completes and finishes playing. 
_mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdComplete, null, null); 
 
....... 
....... 
 
// 14. Track the MediaHeartbeat.Event.AdBreakComplete event when all the ads in  
//     the post-roll pod finish playing.  
_mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdBreakComplete, null, null); 
 
....... 
....... 
 
// 15. Call trackComplete() when the playback reaches the end, i.e., when the 
//     video completes and finishes playing. 
_mediaHeartbeat.trackComplete(); 
 
........ 
........ 
 
// 16. Call trackSessionEnd() when the playback session is over. This method  
//     must be called even if the user does not watch the video to completion.  
_mediaHeartbeat.trackSessionEnd(); 
 
........ 
........ 
```

### iOS

To view this scenario in iOS, set up the following code:

```
//  Set up mediaObject 
ADBMediaObject *mediaObject =  
  [ADBMediaHeartbeat createMediaObjectWithName:MEDIA_NAME  
                     length:MEDIA_LENGTH  
                     streamType:ADBMediaHeartbeatStreamTypeVOD]; 
   
NSMutableDictionary *videoContextData = [[NSMutableDictionary alloc] init]; 
[videoContextData setObject:CUSTOM_VAL_1 forKey:CUSTOM_KEY_1]; 
[videoContextData setObject:CUSTOM_VAL_2 forKey:CUSTOM_KEY_2]; 
  
// 1. Call trackSessionStart when the user clicks Play or if autoplay is used,  
//    i.e., there is an intent to start playback. 
[_mediaHeartbeat trackSessionStart:mediaObject data:videoContextData]; 
....... 
....... 
  
// Pre-roll 
ADBMediaObject *adBreakInfo =  
  [ADBMediaHeartbeat createAdBreakObjectWithName:AD_BREAK_NAME  
                     position:AD_BREAK_POSITION  
                     startTime:AD_BREAK_START_TIME]; 
ADBMediaObject *adInfo =  
  [ADBMediaHeartbeat createAdObjectWithName:AD_NAME  
                     adId:AD_ID  
                     position:AD_POSITION  
                     length:AD_LENGTH]; 
  
// Context ad data 
NSMutableDictionary *adDictionary = [[NSMutableDictionary alloc] init]; 
[adDictionary setObject:@"custom-val1" forKey:@"custom-key1"]; 
[adDictionary setObject:@"custom-val2" forKey:@"custom-key2"]; 
  
// 2. Track the ADBMediaHeartbeatEventAdBreakStart event when the  
//    pre-roll pod starts to play. Note that since this is a pre-roll,  
//    you must track the ADBMediaHeartbeatEventAdBreakStart event  
//    before you call trackPlay. 
[_mediaHeartbeat trackEvent:ADBMediaHeartbeatEventAdBreakStart  
                 mediaObject:adBreakObject  
                 data:adDictionary]; 
....... 
....... 
  
// 3. Track the ADBMediaHeartbeatEventAdStart when the pre-roll  
//    pod's ad starts to play. Note that since this is a pre-roll,  
//    you must track the ADBMediaHeartbeatEventAdStart before you 
//    call trackPlay. 
[_mediaHeartbeat trackEvent:ADBMediaHeartbeatEventAdStart  
                  mediaObject:adObject  
                  data:adDictionary]; 
....... 
....... 
  
// 4. Call trackPlay when the playback actually starts, i.e., when 
//    the first frame of the main content is rendered on the screen. 
[_mediaHeartbeat trackPlay]; 
....... 
....... 
  
// 5. Track the ADBMediaHeartbeatEventAdComplete event when the ad  
//    reaches the end, i.e., when it completes and finishes playing. 
[_mediaHeartbeat trackEvent:ADBMediaHeartbeatEventAdComplete  
                  mediaObject:nil  
                  data:nil]; 
....... 
....... 
  
// 6. Track the ADBMediaHeartbeatEventAdBreakComplete event when all  
//    of the ads in the pod finish playing. 
[_mediaHeartbeat trackEvent:ADBMediaHeartbeatEventAdBreakComplete  
                  mediaObject:nil  
                  data:nil]; 
....... 
....... 
  
// Mid-roll 
ADBMediaObject *midrollAdBreakInfo =  
  [ADBMediaHeartbeat createAdBreakObjectWithName:MIDROLL_BREAK_NAME  
                     position:MIDROLL_BREAK_POSITION  
                     startTime:MIDROLL_BREAK_START_TIME]; 
ADBMediaObject *midrollAdInfo =  
  [ADBMediaHeartbeat createAdObjectWithName:MIDROLL_AD_NAME  
                     adId:MIDROLL_AD_ID  
                     position:MIDROLL_AD_POSITION  
                     length:MIDROLL_AD_LENGTH]; 
  
// context ad data 
NSMutableDictionary *midrollAdDictionary = [[NSMutableDictionary alloc] init]; 
[midrollAdDictionary setObject:@"custom-val1" forKey:@"custom-key1"]; 
[midrollAdDictionary setObject:@"custom-val2" forKey:@"custom-key2"]; 
  
// 7. Track the ADBMediaHeartbeatEventAdBreakStart event when the mid-roll pod  
//    starts to play. 
[_mediaHeartbeat trackEvent:ADBMediaHeartbeatEventAdBreakStart  
                  mediaObject:midrollAdBreakInfo  
                  data:nil]; 
....... 
....... 
  
// 8. Track the ADBMediaHeartbeatEventAdStart event when the mid-roll pod's  
//    ad starts to play. 
[_mediaHeartbeat trackEvent:ADBMediaHeartbeatEventAdStart  
                 mediaObject:midrollAdInfo  
                 data:midrollAdDictionary]; 
....... 
....... 
  
// 9. Track the ADBMediaHeartbeatEventAdComplete event when the ad reaches  
//    the end, i.e., when it completes and finishes playing. 
[_mediaHeartbeat trackEvent:ADBMediaHeartbeatEventAdComplete  
                 mediaObject:nil  
                 data:nil]; 
....... 
....... 
  
// 10. Track the ADBMediaHeartbeatEventAdBreakComplete event when all the  
//     ads in the mid-roll pod finish playing. 
[_mediaHeartbeat trackEvent:ADBMediaHeartbeatEventAdBreakComplete  
                 mediaObject:nil  
                 data:nil]; 
....... 
....... 
  
// Post-roll 
ADBMediaObject *postrollBreakInfo =  
  [ADBMediaHeartbeat createAdBreakObjectWithName:POSTROLL_BREAK_NAME  
                     position:POSTROLL_BREAK_POSITION  
                     startTime:POSTROLL_BREAK_START_TIME]; 
ADBMediaObject *postrollAdInfo =  
  [ADBMediaHeartbeat createAdObjectWithName:POSTROLL_AD_NAME  
                     adId:POSTROLL_AD_ID  
                     position:POSTROLL_AD_POSITION  
                     length:POSTROLL_AD_LENGTH]; 
  
// Context ad data 
NSMutableDictionary *postrollAdDictionary = [[NSMutableDictionary alloc] init]; 
[postrollAdDictionary setObject:@"custom-val1" forKey:@"custom-key1"]; 
[postrollAdDictionary setObject:@"custom-val2" forKey:@"custom-key2"]; 
  
// 11. Track the ADBMediaHeartbeatEventAdBreakStart event when the  
//     post-roll pod starts to play. 
[_mediaHeartbeat trackEvent:ADBMediaHeartbeatEventAdBreakStart  
                 mediaObject:postrollBreakInfo  
                 data:nil]; 
....... 
....... 
  
// 12. Track the ADBMediaHeartbeatEventAdStart event when the  
//     post-roll pod's ad starts to play. 
[_mediaHeartbeat trackEvent:ADBMediaHeartbeatEventAdStart  
                 mediaObject:postrollAdInfo  
                 data:postrollAdDictionary]; 
....... 
....... 
  
// 13. Track the ADBMediaHeartbeatEventAdComplete event when the  
//     post-roll pod's ad finishes playing. 
[_mediaHeartbeat trackEvent:ADBMediaHeartbeatEventAdComplete  
                 mediaObject:nil  
                 data:nil]; 
....... 
....... 
  
// 14. Track the ADBMediaHeartbeatEventAdBreakComplete event when  
//     all the ads in the post-roll pod finish playing. 
[_mediaHeartbeat trackEvent:ADBMediaHeartbeatEventAdBreakComplete  
                 mediaObject:nil data:nil]; 
....... 
....... 
  
// 15. Call trackComplete when the playback reaches the end,  
//     i.e., when the video completes and finishes playing. 
[_mediaHeartbeat trackComplete]; 
....... 
....... 
  
// 16. Call trackSessionEnd when the playback session is over. This method  
//     must be called even if the user does not watch the video to completion. 
[_mediaHeartbeat trackSessionEnd]; 
....... 
....... 
```

### JavaScript

To view this scenario in JavaScript, enter the following text:

```js
// Set up mediaObject 
var mediaInfo = MediaHeartbeat.createMediaObject( 
    Configuration.MEDIA_NAME,  
    Configuration.MEDIA_ID,  
    Configuration.MEDIA_LENGTH,  
    MediaHeartbeat.StreamType.VOD 
); 
 
var videoMetadata = { 
    CUSTOM_KEY_1 : CUSTOM_VAL_1,  
    CUSTOM_KEY_2 : CUSTOM_VAL_2,  
    CUSTOM_KEY_3 : CUSTOM_VAL_3 
}; 
 
// 1. Call trackSessionStart() when Play is clicked or if autoplay is used,  
//    i.e., when there's an intent to start playback.  
this._mediaHeartbeat.trackSessionStart(mediaInfo, videoMetadata); 
 
...... 
...... 
 
// Preroll 
var adBreakInfo =  
  MediaHeartbeat.createAdBreakObject(ADBREAK_NAME,  
                                     ADBREAK_POSITION,  
                                     ADBREAK_START_TIME); 
var adInfo =  
  MediaHeartbeat.createAdObject(AD_NAME,  
                                AD_ID,  
                                AD_POSITION,  
                                AD_LENGTH); 
 
// Custom ad metadata 
var adMetadata = { 
    CUSTOM_KEY_1 : CUSTOM_VAL_1,  
    CUSTOM_KEY_2 : CUSTOM_VAL_2 
}; 
 
// 2. Track the MediaHeartbeat.Event.AdBreakStart event when the preroll pod  
//    starts to play. Note that since this is a preroll, you must track the  
//    MediaHeartbeat.Event.AdBreakStart event before you call trackPlay().  
this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdBreakStart, adBreakInfo); 
 
....... 
....... 
 
// 3. Track the MediaHeartbeat.Event.AdStart event when the preroll pod's ad  
//    starts to play. Note that since this is a preroll, you must track   
//    the MediaHeartbeat.Event.AdStart event before you call trackPlay(). 
this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdStart, adInfo, adMetadata); 
 
....... 
....... 
 
// 4. Call trackPlay() when the playback actually starts, i.e., when the first  
//    frame of the main content is rendered on the screen.  
this._mediaHeartbeat.trackPlay(); 
 
....... 
....... 
 
// 5. Track the MediaHeartbeat.Event.AdComplete event when the ad reaches  
//    the end, i.e., when the ad completes and finishes playing. 
this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdComplete); 
 
....... 
....... 
 
// 6. Track the MediaHeartbeat.Event.AdBreakComplete event when all  
//    of the ads in the pod finish playing. 
this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdBreakComplete); 
 
....... 
....... 
 
// Mid-roll 
var midrollAdBreakInfo =  
  MediaHeartbeat.createAdBreakObject(MIDROLL_BREAK_NAME, 
                                     MIDROLL_BREAK_POSITION,  
                                     MIDROLL_BREAK_START_TIME); 
var midrollAdInfo =  
  MediaHeartbeat.createAdObject(MIDROLL_AD_NAME,  
                                MIDROLL_AD_ID,  
                                MIDROLL_AD_POSITION,  
                                MIDROLL_AD_LENGTH); 
 
// Custom ad metadata 
var midrollAdMetadata = { 
   CUSTOM_KEY_1 : CUSTOM_VAL_1,  
   CUSTOM_KEY_2 : CUSTOM_VAL_2 
}; 
 
// 7. Track the MediaHeartbeat.Event.AdBreakStart event when the midroll  
//    pod starts to play. 
this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdBreakStart, midrollAdBreakInfo); 
 
....... 
....... 
 
// 8. Track the MediaHeartbeat.Event.AdStart event when the midroll pod's  
//    ad starts to play. 
this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdStart, midrollAdInfo, midrollAdMetadata); 
 
....... 
....... 
 
// 9. Track the MediaHeartbeat.Event.AdComplete event when the ad reaches  
//    the end, i.e., when the ad completes and finishes playing. 
this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdComplete); 
 
....... 
....... 
 
// 10. Track the MediaHeartbeat.Event.AdBreakComplete event when all of  
//     the ads in the midroll pod finish playing. 
this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdBreakComplete); 
 
....... 
....... 
 
// Postroll 
var postrollAdBreakInfo = MediaHeartbeat.createAdBreakObject(POSTROLL_BREAK_NAME,  
                                                             POSTROLL_BREAK_POSITION,  
                                                             POSTROLL_BREAK_START_TIME); 
var postrollAdInfo = MediaHeartbeat.createAdObject(POSTROLL_AD_NAME,  
                                                   POSTROLL_AD_ID,  
                                                   POSTROLL_AD_POSITION,  
                                                   POSTROLL_AD_LENGTH); 
 
// Custom ad metadata 
var postrollAdMetadata = { 
   CUSTOM_KEY_1 : CUSTOM_VAL_1,  
   CUSTOM_KEY_2 : CUSTOM_VAL_2 
}; 
 
// 11. Track the MediaHeartbeat.Event.AdBreakStart event when the postroll  
//     pod starts to play. 
this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdBreakStart, postrollAdBreakInfo); 
 
....... 
....... 
 
// 12. Track the MediaHeartbeat.Event.AdStart event when the postroll pod's ad  
//     starts to play. 
this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdStart, postrollAdInfo, postrollAdMetadata); 
 
....... 
....... 
 
// 13. Track the MediaHeartbeat.Event.AdComplete event when the ad reaches   
//     the end, i.e., when the ad completes and finishes playing. 
this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdComplete); 
 
....... 
....... 
 
// 14. Track the MediaHeartbeat.Event.AdBreakComplete event when all of  
//     the ads in the postroll pod finish playing. 
this._mediaHeartbeat.trackEvent(MediaHeartbeat.Event.AdBreakComplete); 
 
....... 
....... 
 
// 15. Call trackComplete() when the playback reaches the end, i.e., when playback 
//     completes and finishes playing. 
this._mediaHeartbeat.trackComplete(); 
 
........ 
........ 
 
// 16. Call trackSessionEnd() when the playback session is over. This method must be  
//     called even if the user does not watch the video to completion. 
this._mediaHeartbeat.trackSessionEnd(); 
 
........ 
........ 
```
