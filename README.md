**Table of Contents**

[TOCM]

[TOC]

## GStreamer Intro
- Development framework for creating applications like media players, video editors etc
- Written in C for portability, speed, easy binding
- Uses GObject for OO support
- Hosted on https://gitlab.freedesktop.org/gstreamer
- Bindings in Java, C++, Rust, Python & more

### Install, Build
- Using package Manager
 - https://gstreamer.freedesktop.org/documentation/installing/index.html
- From Sources (usually required for latest version)
  - https://gstreamer.freedesktop.org/documentation/frequently-asked-questions/git.html
  - https://developer.ridgerun.com/wiki/index.php?title=GStreamer_build_on_Ubuntu_16.04#GStreamer_module_build_instructions

## What is GStreamer ?
The GStreamer core function is to provide a framework for plugins, data flow and media type handling/negotiation. It also provides an API to write applications using the various plugins.
  
  
![](https://gstreamer.freedesktop.org/documentation/application-development/introduction/images/gstreamer-overview.png)

GStreamer provides
- an API for multimedia applications
- a plugin architecture
- a pipeline architecture
- a mechanism for media type handling/negotiation
- a mechanism for synchronization
- over 250 plug-ins providing more than 1000 elements
- a set of tools

GStreamer plug-ins could be classified into
- protocols handling
- sources: for audio and video (involves protocol plugins)
- formats: parsers, formaters, muxers, demuxers, metadata, subtitles
- codecs: coders and decoders
- filters: converters, mixers, effects, ...
- sinks: for audio and video (involves protocol plugins)

## GStreamer Components

#### Elements
#### Bins
#### Bus
#### Pads & Caps
#### Buffers & Events

#### Plugins - Good, Bad, Ugly ?
- Good
- Bad
- Ugly

## GStreamer CLI Tools

- gst-inspect-1.0
  - Discover available plugins
  - Discover a plugin's pads, capabilites, properties
  - Discover plugins compatible with another
  - ` gst-inspect-1.0 videotestsrc`
- gst-launch-1.0
  - Debug tool
  - Build and run a GStreamer pipeline from CLI
  - src -> sink `gst-launch-1.0 videotestsrc ! ximagesink`
  - video filesrc, demux, decode, play 
    - `gst-launch-1.0 filesrc location=/home/sean/Downloads/videoplayback.mp4 ! qtdemux ! decodebin ! autovideosink`
	
### Element Naming, Queue & T
- name demux element, select the video pad, decode, sink
    - `gst-launch-1.0 filesrc location=/home/sean/Downloads/videoplayback.mp4 ! qtdemux name=dmx dmx.video_0 ! decodebin ! autovideosink`
- name demux element, select the audio pad, decode, sink
    - `gst-launch-1.0 filesrc location=/home/sean/Downloads/videoplayback.mp4 ! qtdemux name=dmx dmx.audio_0 ! decodebin ! autoaudiosink`
- name demux element, queue the audio & video pads, decode both, sink
    - `gst-launch-1.0 filesrc location=/home/sean/Downloads/videoplayback.mp4 ! qtdemux name=dmx dmx.video_0 ! queue ! decodebin ! autovideosink  dmx.audio_0 ! queue ! decodebin ! autoaudiosink`
	- Addition of the queue element here adds a pipeline thread boundary & buffering. A thread will fill the buffer at the source side & another will empty it on the sink side.
- T element allows us to split a src into multiple sink pads
  - `gst-launch-1.0 filesrc location=/home/sean/Downloads/videoplayback.mp4 ! qtdemux name=dmx dmx.audio_0 ! tee name=t t. ! queue ! decodebin ! audioconvert ! goom ! autovideoconvert ! autovideosink  t. ! queue ! autoaudiosink`
  - Here after we demux the audio and video, we then T the audio pad. Then link the T'd audio pad to an audio visualisation plugin and send it to our video sink. Then link another T'd audio pad to our audio sink. Giving audio with a visualistion. 

