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

<img src="/images/media_lifecycle.png" />
