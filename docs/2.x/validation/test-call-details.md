# Test call details

## Start the media player

### Adobe Analytics (AppMeasurement) Start call

| Parameter | Value (sample) |
|---|---|
| `pev2` | ms_s |
| `a.media.friendlyName` | Episode Title |
| `a.media.name` | 123456 |
| `a.media.length` | 120 |
| `a.media.playerName` | HTML5 |
| `a.media.view` | true |
| `a.contentType` | vod |
| `custom.[value]` | Custom metadata fields |
| `a.media.[value]` | Standard metadata fields |

Notes:

* Additional context data variables should be present and contain metadata. See metadata details below.
* Length for linear streams should be set to the best estimate for the current show.

### Standard metadata in Adobe Analytics (AppMeasurement) Start call

| Parameter | Value (sample) |
|---|---|
| `a.media.show` | Show Title |
| `a.media.season` | 6 |
| `a.media.episode` | Episode Title |
| `a.media.asset_id` | 123456 |
| `a.media.genre` | comedy |
| `a.media.first_air_date` | YYYY-07-04 |
| `a.media.rating` | TV-14 |
| `a.media.originator` | production house |
| `a.media.network` | network |
| `a.media.ad_load` | 1 |
| `a.media.mvpd` | mvpd |
| `a.media.authorized` | unlocked |
| `a.media.feed` | no feed |
| `a.media.stream_format` | 0 |

### Custom metadata in Adobe Analytics (AppMeasurement) Start call

| Parameter | Value (sample) |
|---|---|
| `custom.metadataA` | value |
| `custom.metadataB` | value |

### Media Analytics (heartbeats) Start call

| Parameter | Value (sample) |
|---|---|
| `s:event:type` | start |
| `l:event:playhead` | 0 |
| `l:event:duration` | 4 |
| `s:asset:name` | Episode Title |
| `s:asset:video_id` | 123456 |
| `l:asset:length` | 120 |
| `s:stream:type` | vod |
| `s:asset:type` | main |
| `s:meta:custom.[value]` | Custom metadata fields |
| `s:meta:a.media.[value]` | Standard metadata fields |

Notes:

* Additional context data variables should be present and contain metadata. See metadata details below.
* Playhead position for linear streams on video start should be set to the seconds elapsed since the start of the current show, not 0.

### Standard metadata in Media Analytics (heartbeats) Start call

| Parameter | Value (sample) |
|---|---|
| `s:meta:a.media.show` | Show |
| `s:meta:a.media.season` | 6 |
| `s:meta:a.media.episode` | Episode Title |
| `s:meta:a.media.asset_id` | 123456 |
| `s:meta:a.media.genre` | comedy |
| `s:meta:a.media.first_air_date` | 2018-07-04 |
| `s:meta:a.media.rating` | TV-14 |
| `s:meta:a.media.originator` | production house |
| `s:meta:a.media.network` | network |
| `s:meta:a.media.ad_load` | 1 |
| `s:meta:a.media.mvpd` | mvpd |
| `s:meta:a.media.authorized` | unlocked |
| `s:meta:a.media.feed` | no feed |
| `s:meta:a.media.stream_format` | 0 |

### Custom metadata in Media Analytics (heartbeats) Start call

| Parameter | Value (sample) |
|---|---|
| `s:meta:custom.metadata` | value |
| `s:meta:custom.metadata` | value |

### Media Analytics (heartbeats) Adobe Analytics Start call

| Parameter | Value (sample) |
|---|---|
| `s:event:type` | aa_start |
| `l:event:playhead` | 0 |
| `l:event:duration` | 4 |
| `s:asset:name` | Episode Title |
| `s:asset:video_id` | 123456 |
| `l:asset:length` | 120 |
| `s:stream:type` | vod |
| `s:asset:type` | main |

Notes:

* This call indicates that the Media SDK has requested that an Adobe Analytics `pev2=ms_s` call be sent to the Adobe Analytics (AppMeasurement) server.
* This call does not contain custom metadata.

## View ad playback

### Adobe Analytics (AppMeasurement) Ad Start call

| Parameter | Value (sample) |
|---|---|
| `pev2` | msa_s |
| `a.media.name` | 123456 |
| `a.media.ad.name` | 9378 |
| `a.media.ad.friendlyName` | Video_VPAID_DFA |
| `a.media.ad.podFriendlyName` | preroll |
| `a.media.ad.length` | 15 |
| `a.media.ad.playerName` | HTML5 |
| `a.media.ad.pod` | c27aaf3ff8224bb6b9ebfe1b2e79073d_1 |
| `a.media.ad.podPosition` | 1 |
| `a.media.ad.podSecond` | 0.0 |
| `a.media.ad.view` | True |
| `custom.[value]` | Metadata fields |
| `a.media.[value]` | Standard metadata fields |

Notes:

* Additional context data variables should be present and contain metadata. See metadata details below.
* Ad length may be set to -1 if not available on ad start.

### Standard metadata in Adobe Analytics (AppMeasurement) Ad Start call

| Parameter | Value (sample) |
|---|---|
| `a.media.show` | Show Title |
| `a.media.season` | 6 |
| `a.media.episode` | Episode Title |
| `a.media.asset_id` | 123456 |
| `a.media.genre` | comedy |
| `a.media.first_air_date` | 2016-07-04 |
| `a.media.rating` | TV-14 |
| `a.media.originator` | production house |
| `a.media.network` | network |
| `a.media.ad_load` | 1 |
| `a.media.mvpd` | mvpd |
| `a.media.authorized` | unlocked |
| `a.media.feed` | no feed |
| `a.media.stream_format` | 0 |

### Custom metadata in Adobe Analytics (AppMeasurement) Ad Start call

| Parameter | Value (sample) |
|---|---|
| `custom.metadata` | value |
| `custom.metadata` | value |

### Media Analytics (heartbeats) Ad Start call

| Parameter | Value (sample) |
|---|---|
| `s:event:type` | start |
| `l:event:playhead` | 0 |
| `l:event:duration` | 4 |
| `s:asset:ad_id` | 9378 |
| `l:asset:length` | 120 |
| `s:stream:type` | vod |
| `s:asset:type` | ad |
| `s:meta:custom.[value]` | Custom metadata fields |
| `s:meta:a.media.[value]` | Standard metadata fields |

Notes:

* Additional context data variables should be present and contain metadata. See metadata details below.
* Ad length may be set to -1 if not available on ad start.

### Standard metadata in Media Analytics (heartbeats) Ad Start call

| Parameter | Value (sample) |
|---|---|
| `s:meta:a.media.show` | Show |
| `s:meta:a.media.season` | 6 |
| `s:meta:a.media.episode` | Episode Title |
| `s:meta:a.media.asset_id` | 123456 |
| `s:meta:a.media.genre` | comedy |
| `s:meta:a.media.first_air_date` | YYYY-07-04 |
| `s:meta:a.media.rating` | TV-14 |
| `s:meta:a.media.originator` | production house |
| `s:meta:a.media.network` | network |
| `s:meta:a.media.ad_load` | 1 |
| `s:meta:a.media.mvpd` | mvpd |
| `s:meta:a.media.authorized` | unlocked |
| `s:meta:a.media.feed` | no feed |
| `s:meta:a.media.stream_format` | 0 |

### Custom metadata in Media Analytics (heartbeats) Ad Start call

| Parameter | Value (sample) |
|---|---|
| `s:meta:custom.metadata` | value |
| `s:meta:custom.metadata` | value |

### Media Analytics (heartbeats) Adobe Analytics Ad Start call

| Parameter | Value (sample) |
|---|---|
| `s:event:type` | aa_ad_start |
| `l:event:playhead` | 0 |
| `l:event:duration` | 0 |
| `s:asset:ad_id` | 9378 |
| `l:asset:ad_length` | 15 |
| `s:stream:type` | vod |
| `s:asset:type` | ad |

### Media Analytics (heartbeats) Ad Play call

| Parameter | Value (sample) |
|---|---|
| `s:event:type` | play |
| `l:event:playhead` | 15 |
| `l:event:duration` | 0 |
| `s:asset:ad_id` | 9378 |
| `l:asset:ad_length` | 15 |
| `s:stream:type` | vod |
| `s:asset:type` | ad |

### Media Analytics (heartbeats) Ad Pause call

| Parameter | Value (sample) |
|---|---|
| `s:event:type` | pause |
| `l:event:playhead` | 15 |
| `l:event:duration` | 0 |
| `s:asset:ad_id` | 9378 |
| `l:asset:ad_length` | 15 |
| `s:stream:type` | vod |
| `s:asset:type` | ad |

### Media Analytics (heartbeats) Adobe Analytics Ad Complete call

| Parameter | Value (sample) |
|---|---|
| `s:event:type` | complete |
| `l:event:playhead` | 15 |
| `l:event:duration` | 0 |
| `s:asset:ad_id` | 9378 |
| `l:asset:ad_length` | 15 |
| `s:stream:type` | vod |
| `s:asset:type` | ad |

## Play main content

### Media Analytics (heartbeats) Play call

| Parameter | Value (sample) |
|---|---|
| `s:event:type` | play |
| `l:event:playhead` | 29 |
| `l:event:duration` | 10189 |
| `s:asset:name` | Episode Title |
| `s:asset:video_id` | 123456 |
| `l:asset:length` | 120 |
| `s:stream:type` | vod |
| `s:asset:type` | main |

Notes:

* The playhead position should increment by 10 seconds with every play call.
* The `l:event:duration` value represents the number of milliseconds since the last tracking call and should be roughly the same value on each 10 second call.

## Pause main content

### Media Analytics (heartbeats) Pause call

| Parameter | Value (sample) |
|---|---|
| `s:event:type` | pause |
| `l:event:playhead` | 29 |
| `l:event:duration` | 10189 |
| `s:asset:name` | Episode Title |
| `s:asset:video_id` | 123456 |
| `l:asset:length` | 120 |
| `s:stream:type` | vod |
| `s:asset:type` | main |
