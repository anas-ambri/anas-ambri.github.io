---
layout: post
title: "Exoplayer on Android"
comments: true
categories: blog
published: false
disqus: y
tags: Android Media
---


##Supported formats

<img src="/images/supported_media.PNG" />

##Play music

<img src="/images/exoplayer_play_music.PNG" />

##Layout for exoplayer

<img src="/images/exoplayer_ui.PNG" />

##Non adaptive streaming

<img src="/images/non_adaptive_streaming_models.PNG" />

- If using a custom container format, need to provide a custom `ExtractorSampleSource`
- If using a custom network protocol, need to provide a custom `DefaultUriDataSource`

##Adaptive streaming

<img src="/images/adaptive_streaming_models.PNG" />

`AdaptiveEvaluator` determines the quality to start with (based on network). If don't like the custom low-quality-first, override it.

##Provided templates

- `ExtractorRendererBuilder` (for non-adaptive playback)
- `xxxxRendererBuilder` (xxxx being Dash, HLS, or SmoothStreaming) for the order types

##Relationship with lifecycle
In N+, guarantee of stop being called right away (not guaranteed before, possibly 5 sec delay)
<img src="/images/media_lifecycle.png" />

##Audio Focus
Last request gets it. Hold on to it until `stop`
The request takes a listener, that needs to handle callbacks when audio focus is/might be lost

- AUDIOFOCUS_LOSS: stop playback
- AUDIOFOCUS_LOSS_TRANSIENT: pause, will get GAIN later
- AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK: Lower volume, keep playing, or pause. Depends on use case (Test using 'Ok Google')

##ACTION_AUDIO_BECOMING_NOISY
Happens when the user switches from headphones to speaker. Best thing to do is to pause. Can't add receiver in manifest

<img src="/images/action_audio_becoming_noisy.png" />

##Media notification

- setColor is needed. The color used must not be too strong because it will cover the whole notification
- The actions' order depends on the order in which they are added (pause should be highest, thus first)
- Important to set MediaSession

<img src="/images/media_notification.png" />

##Media notification with service

Separating media playback service into its own process, so that memory footprint is small

<img src="/images/media_lifecycle_with_service.png" />
