Welcome to the **JACK Audio Connection Kit** wiki! 

Notice: this wiki is undergoing renovation, not all content is currently up-to-date. Drop into IRC for the latest.

**jackd** is the JACK audio server daemon that allows JACK client programs to process and route audio and MIDI data in a synchronous, sample-accurate and low\-latency manner. Originally written for the GNU/Linux operating system, it also runs on various Unix platforms, Mac OS X and MS Windows.

JACK can connect a number of different client applications to an audio device and also to each other. Most clients are external, running in their own processes as normal applications. JACK also supports internal clients, which run within the jackd process using a loadable "plugin" interface.

JACK differs from other audio servers in being designed from the ground up for professional audio work. It focuses on two key areas: synchronous execution of all clients, and low latency operation.

# User Resources
* [[Differences between jack1 and jack2]]
* [[FAQ and myths]]
* [[Troubleshooting]]
* [[Mailing lists]]

## Manuals
* [jackd2 Manpages](https://manpages.debian.org/buster/jackd2)

## Guides and walk-throughs
* [Wikipedia: JACK Audio Connection Kit](https://en.wikipedia.org/wiki/JACK_Audio_Connection_Kit) - general overview
* [Demystifying JACK – A Beginners Guide to Getting Started with JACK](https://libremusicproduction.com/articles/demystifying-jack-%E2%80%93-beginners-guide-getting-started-jack) - Libre Music Production
* [CCRMA JACK Guide](https://ccrma.stanford.edu/docs/common/JACK.html)
* [Arch wiki: JACK Audio Connection Kit](https://wiki.archlinux.org/index.php/JACK_Audio_Connection_Kit) - Installation, Configuration, Troubleshooting
* [Past, Present and Future of the JACK Audio Connection Kit](https://media.ccc.de/v/sonoj2019-1902-jack-past-present-future) - video, talk by falkTX at Sonoj 2019
* [Sound Engineers Guide to Jackd](http://www.orford.org/assets/jack-idiots_guide.txt) (jack-idiots_guide.txt)
* [jack_control walkthrough](https://github.com/jackaudio/jackaudio.github.com/wiki/WalkThrough_User_jack_control) - Python tool to control JACK2 server
* [On JACK client threads](wiki/WalkThrough_User_ClientThreads)

## Configuration
* [List of JACK Frame & Period settings ideal for USB interface](https://linuxmusicians.com/viewtopic.php?f=47&t=10707) - (Frames/Sample Rate) * Period = Theoretical (or Math-derived) Latency
* [Latency Measurement Results](https://wiki.linuxaudio.org/wiki/jack_latency_results)
* [Balancing Performance and Reliability in Jack](https://www.penguinproducer.com/Blog/2011/10/balancing-performance-and-reliability-in-jack/)
* [List of JACK Frame \[or Buffer\] & Period settings ideal for USB interface](https://wiki.linuxaudio.org/wiki/list_of_jack_frame_period_settings_ideal_for_usb_interface)
* [[Cgroups]] - for a realtime kernel configuration
* [JACK and Headphones](http://tedfelix.com/linux/jack-headphones.html) - duplicate outputs at ALSA level

## Multiple soundcards
* [ALSA in/out](wiki/WalkThrough_User_AlsaInOut) - use more than one soundcard with jackd
  * https://github.com/IARI/alsa_jack_gui - a qt-based gui to manage alsa_in and alsa_out daemons
* [Zita-ajbridge](http://kokkinizita.linuxaudio.org/linuxaudio/zita-ajbridge-doc/quickguide.html)

## PulseAudio
* [JACK and Pulseaudio](wiki/WalkThrough_User_PulseOnJack)
* [PulseAudio and Jack](http://0pointer.de/blog/projects/when-pa-and-when-not.html)
* [How to replace Pulseaudio with Jack, Jack and PulseAudio Together as Friend](https://forum.manjaro.org/t/how-to-replace-pulseaudio-with-jack-jack-and-pulseaudio-together-as-friend/2086) - Manjaro Linux Forum

# JACK Transport/Timebase
* [[JACK Transport support]] and [[JACK Transport limitations]]

```
jack_transport> ?
  activate	  Call jack_activate().
  exit		  Exit transport program.
  deactivate  Call jack_deactivate().
  help		  Display help text [<command>].
  locate	  Locate to frame <position>.
  master	  Become timebase master [<conditionally>].
  play		  Start transport rolling.
  quit		  Synonym for 'exit'.
  release	  Release timebase.
  stop		  Stop transport.
  tempo       Set beat tempo <beats_per_min>.
  timeout	  Set sync timeout in <seconds>.
  ?  		  Synonym for `help'.

echo play |jack_transport
  # pass a command to execute. tempo change doesn't work via this method.
```

* [transporter.py](https://github.com/SpotlightKid/jack-audio-tools/blob/master/transport/transporter.py) / [timebase_master.py](https://github.com/SpotlightKid/jack-audio-tools/blob/master/transport/timebase_master.py) - query and manipulate JACK transport state and provide timebase information using jackclient-python.
* [jacktransportloop](http://danmbox.github.io/jack-file/) - forces the Jack transport to loop between two time-points.
* https://github.com/ycollet/qtmiditrans - translates midi events into jack transport
* [jack-trans2midi](https://sourceforge.net/projects/jack-trans2midi/) - converts jack transport into midi clock messages.
* [JackCtlMMC/QJackMMC](http://jackctlmmc.sourceforge.net/) - allow MIDI Machine Control (MMC) to drive JACK transport.
* https://github.com/rncbc/jack_link - a JACK transport timebase prototype bridge to [Ableton Link](https://www.ableton.com/en/link/).

# OSC
* https://github.com/ventosus/jack_osc - Open Sound Control (OSC) via Jack

# Networked
* [NetJACK 1](wiki/WalkThrough_User_NetJack) - connect JACK 1 servers via network
* [NetJACK 2](wiki/WalkThrough_User_NetJack2) - connect JACK 2 servers via network

# Install and packaging
* [[Installation filesystem layout]]
* [[Suggested packaging approach]]
* [JACK D-Bus packaging](wiki/JACK-DBus-packaging)

# Developer resources
* [JACK API reference (doxygen)](https://jackaudio.github.io/api)
* [[jack2 source code guide]]
* [[jack1 source code guide]]
* [[JACK API differences]] - changes between jack1 and jack2
* [[Repository layout]]
* [[WalkThrough_Dev_LatencyBufferProcess]]
* [[WalkThrough_Dev_SimpleAudioClient]]
* [[WalkThrough_Dev_SimpleMidiClient]]
* [[WalkThrough_Dev_Tools]]
* [[WalkThrough_Dev_Cpp]]
* [[WalkThrough_Dev_AlsaIoTools]]
* [[WalkThrough_Dev_NetOne]]
* https://github.com/jackaudio/example-clients

## Bindings
* https://github.com/spatialaudio/jackclient-python - JACK Audio Connection Kit (JACK) Client for Python snake 
* https://github.com/xthexder/go-jack -  Go bindings for Jack Audio Connection Kit

# Metadata
* [JACK Metadata](https://jackaudio.org/metadata)
  * [pretty-name](https://jackaudio.org/metadata/pretty-name)
  * [signal-type](https://jackaudio.org/metadata/signal-type)
  * [order](https://jackaudio.org/metadata/order)
  * etc.
* [jack2/JackMetadata.cpp at develop · jackaudio/jack2](https://github.com/jackaudio/jack2/blob/develop/common/JackMetadata.cpp#L30)
* https://github.com/drobilla/jackey - a simple define-only header that serves as an ad-hoc standard for Jack Metadata property keys.

`jack_property` - list, set and delete any metadata associated with the ports and clients of a JACK server.

* [jack-property-listener.py](https://gist.github.com/SpotlightKid/dc70dac006337c99bf65bcdc98e43ddc) -  Listen to and print JACK client/port metadata changes.
* https://github.com/drobilla/jackey - a simple define-only header that serves as an ad-hoc standard for Jack Metadata property keys. 

# JACK tools
* https://github.com/jackaudio/tools

JACK2 package commands:
```
jack_alias
jack_bufsize
jack_control
jack_cpu
jack_cpu_load
jack_disconnect
jack_evmon
jack_freewheel
jack_iodelay
jack_latent_client
jack_load
jack_metro
jack_midi_dump
jack_midi_latency_test
jack_midiseq
jack_midisine
jack_monitor_client
jack_multiple_metro
jack_net_master
jack_net_slave
jack_netsource
jack_rec
jack_samplerate
jack_server_control
jack_session_notify
jack_showtime
jack_simple_client
jack_simple_session_client
jack_test
jack_thru
jack_transport
jack_unload
jack_wait
jack_zombie
```

* https://github.com/SpotlightKid/jack-audio-tools - collection of utilities and tools for the JACK audio ecosystem
* https://github.com/Gimmeapill/xruncounter - measure jack xruns and evaluate the overall performance of a system
* https://github.com/7890/jack_tools - playground for JACK tools and experiments
* [Jacktools - Realtime Audio Processors as Python Classes](https://media.ccc.de/v/lac2018-46-jacktools_realtime_audio_processors_as_python_classes) - video [[*](http://kokkinizita.linuxaudio.org/linuxaudio/downloads/index.html)]
* https://github.com/x42/jack-stdio
* https://github.com/fps/jack2_split - facilitates parallelism in serial jack graphs by introducing latency, only useful for jack2/jackdmp

## Control
* [QjackCtl](https://qjackctl.sourceforge.io) - a simple Qt application to control the JACK sound server [[*](https://github.com/rncbc/qjackctl)]
  * https://github.com/brummer10/pajackconnect - script to be invoked via QjackCtl to make JACK with with PulseAudio [[*](https://linuxmusicians.com/viewtopic.php?f=19&t=13234)]
  * https://github.com/SpotlightKid/jack-select - systray application to quickly change the JACK configuration from QjackCtl presets via DBus
  * https://github.com/kmatheussen/qjackctl_macos - script to build on macos
* [Cadence](https://kx.studio/Applications:Cadence) - a set of tools useful for audio production [[*](https://github.com/falkTX/cadence)]
* https://github.com/be1/jackie - a small jack daemon controller
* https://github.com/progwolff/jackman - Collection of scripts that help managing multiple audio interfaces with Jack.
  * https://github.com/progwolff/jackman_kcm - KDE

## Mixer
* https://github.com/jack-mixer/jack_mixer - a GTK+ JACK audio mixer app with look similar to its hardwarecounterpart.  It has lot of useful features, apart from being able to mixmultiple JACK audio streams.

## WINE
* https://github.com/wineasio/wineasio - provides an ASIO to JACK driver for WINE

## Haiku
* https://github.com/Barrett17/libjackcompat - experimental media_kit compatibility layer

## Testing
* https://github.com/falkTX/jack_interposer - Library to check at run-time whether an application makes non-RT-safe calls inside JACK's process() callback 

# Proposed features
* [[Proposal for MIDI API extension for System Exclusive Messages]]
* [[Proposal for JesseChappellLooping]]

# Session Management
Both LASH/LADISH and JACK Session are deprecated; the use of [NSM](https://linuxaudio.github.io/new-session-manager) is recommended.