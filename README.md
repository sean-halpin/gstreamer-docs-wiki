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
For the application programmer, elements are best visualized as black boxes. On the one end, you might put something in, the element does something with it and something else comes out at the other side.
##### Source Elements
- Generates data for the pipeline, maybe reading from disc, network, video or sound card.
	- ![](https://gstreamer.freedesktop.org/documentation/application-development/basics/images/src-element.png)
  
##### Filters, convertors, demuxers, muxers and codecs
- Filter like elements may have 1-n inputs & 1-n outputs
	- ![](https://gstreamer.freedesktop.org/documentation/application-development/basics/images/filter-element.png) ![](https://gstreamer.freedesktop.org/documentation/application-development/basics/images/filter-element-multi.png)

##### Sink elements
- Sink elements are the endpoint for a pipeline, these might write to disc, network or display or play some media/data liike video or sound 
	- ![](https://gstreamer.freedesktop.org/documentation/application-development/basics/images/sink-element.png)

##### Linking Elements

By linking a source element with zero or more filter-like elements and finally a sink element, you set up a media pipeline. Data will flow through the elements.

How would we construct the following pipeline as an app programmer  

- ![](https://gstreamer.freedesktop.org/documentation/application-development/basics/images/linked-elements.png)

1. Create a Pipeline
2. Create all elements
	- source
	- filter
	- sink
3. Add elements to the pipeline
4. Link elements in order using `gst_element_link` or `gst_element_link_many()`

Example ;
	#include <gst/gst.h>

	int
	main (int   argc,
	char *argv[])
	{
	GstElement *pipeline;
	GstElement *source, *filter, *sink;

	/* init */
	gst_init (&argc, &argv);

	/* create pipeline */
	pipeline = gst_pipeline_new ("my-pipeline");

	/* create elements */
	source = gst_element_factory_make ("fakesrc", "source");
	filter = gst_element_factory_make ("identity", "filter");
	sink = gst_element_factory_make ("fakesink", "sink");

	/* must add elements to pipeline before linking them */
	gst_bin_add_many (GST_BIN (pipeline), source, filter, sink, NULL);

	/* link */
	if (!gst_element_link_many (source, filter, sink, NULL)) {
	g_warning ("Failed to link elements!");
	}

	[..]

	}

##### Element States - Briefly
- GST_STATE_NULL
	- default element state
	- no resources are allocated in this state
- GST_STATE_READY
	- element has allocated all of its global resources
- GST_STATE_PAUSED
	- element has opened the stream, but is not actively processing it
- GST_STATE_PLAYING
	- clock is running & stream is processing

#### Bins
![](https://gstreamer.freedesktop.org/documentation/application-development/basics/images/bin-element.png)

- A bin is a container element. You can add elements to a bin. Since a bin is an element itself, a bin can be handled in the same way as any other element.
- Bins allow you to combine a group of linked elements into one logical element. You do not deal with the individual elements anymore but with just one element, the bin.
- Instantiating a bin is done the same way as an [element](https://gstreamer.freedesktop.org/documentation/application-development/basics/bins.html)

#### Bus

- Output  all bus messages on CLI by using -m option `gst-launch-1.0 -m` 
- A bus is a simple system that takes care of forwarding messages from the streaming threads to an application in its own thread context.
	- We can peek at the bus by adding a callback for each bus message using `gst_bus_add_watch()`
	- We can also peek at the bus by adding a callback for bus signals by message type using `gst_bus_add_signal_watch();` then `g_signal_connect()`
- [Code Examples](https://gstreamer.freedesktop.org/documentation/application-development/basics/bus.html)

#### Pads & Caps
- Pads are the element's interface to the outside world
- Data streams from one element's source pad to another element's sink pad
- Pad type is defined by two properties:
	- direction
		- `src`
			- this is a source seen from outside the element (outbound)
		- `sink`
			- this is a sink seen from outside the element (inbound)
	- availability
		- `always`
			- always available
		- `sometimes` (aka dynamic)
			- available sometimes, can dissapear any time
		- `request`
			- available upon application level request

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

  ### More Interesting CLI Pipelines
	- GStreamer Cheatsheet
	- http://wiki.oz9aec.net/index.php/Gstreamer_cheat_sheet
