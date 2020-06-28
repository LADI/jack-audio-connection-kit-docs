Welcome to the **JACK Audio Connection Kit** wiki! 

Notice: this wiki is undergoing renovation, not all content is currently up-to-date. Drop into IRC for the latest.

# User Resources
* [jackd2 Manpages](https://manpages.debian.org/buster/jackd2)
* [[Differences between jack1 and jack2]]
* [[FAQ and myths]]
* [[Troubleshooting]]
* [[Mailing lists]]
* https://en.wikipedia.org/wiki/JACK_Audio_Connection_Kit

# Guides and walk-throughs
* [Demystifying JACK – A Beginners Guide to Getting Started with JACK](https://libremusicproduction.com/articles/demystifying-jack-%E2%80%93-beginners-guide-getting-started-jack) - Libre Music Production
* https://wiki.archlinux.org/index.php/JACK_Audio_Connection_Kit
* [jack_control walkthrough](https://github.com/jackaudio/jackaudio.github.com/wiki/WalkThrough_User_jack_control) - Python tool to control JACK2 server
* [List of JACK Frame & Period settings ideal for USB interface](https://linuxmusicians.com/viewtopic.php?f=47&t=10707) - (Frames/Sample Rate) * Period = Theoretical (or Math-derived) Latency
* [Balancing Performance and Reliability in Jack](https://www.penguinproducer.com/Blog/2011/10/balancing-performance-and-reliability-in-jack/)
* [ALSA in/out](wiki/WalkThrough_User_AlsaInOut) - use more than one soundcard with jackd
  * https://github.com/IARI/alsa_jack_gui - a qt-based gui to manage alsa_in and alsa_out daemons
  * [Zita-ajbridge](http://kokkinizita.linuxaudio.org/linuxaudio/zita-ajbridge-doc/quickguide.html)
* [NetJACK 1](wiki/WalkThrough_User_NetJack) - connect JACK 1 servers via network
* [NetJACK 2](wiki/WalkThrough_User_NetJack2) - connect JACK 2 servers via network
* [JACK and Pulseaudio](wiki/WalkThrough_User_PulseOnJack)
* [PulseAudio and Jack](http://0pointer.de/blog/projects/when-pa-and-when-not.html)

# About JACK
* [Past, Present and Future of the JACK Audio Connection Kit](https://media.ccc.de/v/sonoj2019-1902-jack-past-present-future) - video, talk by Filipe Coelho at Sonoj 2019
* [[JACK Transport support]] and [[JACK Transport limitations]]
* [On JACK client threads](wiki/WalkThrough_User_ClientThreads)
* [[Cgroups]] - for a realtime kernel configuration
* [Sound Engineers Guide to Jackd](http://www.orford.org/assets/jack-idiots_guide.txt) (jack-idiots_guide.txt)

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

## Metadata
* [JACK Metadata](https://jackaudio.org/metadata)
  * [pretty-name](https://jackaudio.org/metadata/pretty-name)
  * [signal-type](https://jackaudio.org/metadata/signal-type)
  * [order](https://jackaudio.org/metadata/order)
  * etc.
* [jack2/JackMetadata.cpp at develop · jackaudio/jack2](https://github.com/jackaudio/jack2/blob/develop/common/JackMetadata.cpp#L30)
* [jack-property-listener.py](https://gist.github.com/SpotlightKid/dc70dac006337c99bf65bcdc98e43ddc) -  Listen to and print JACK client/port meta-data changes.
* https://github.com/drobilla/jackey - a simple define-only header that serves as an ad-hoc standard for Jack Metadata property keys. 

# JACK tools
* https://github.com/jackaudio/tools
* https://github.com/jackaudio/example-clients
* https://github.com/SpotlightKid/jack-audio-tools - collection of utilities and tools for the JACK audio ecosystem
* https://github.com/Gimmeapill/xruncounter - measure jack xruns and evaluate the overall performance of a system
* https://github.com/7890/jack_tools - playground for JACK tools and experiments
* [Jacktools - Realtime Audio Processors as Python Classes](https://media.ccc.de/v/lac2018-46-jacktools_realtime_audio_processors_as_python_classes) - video [[*](http://kokkinizita.linuxaudio.org/linuxaudio/downloads/index.html)]
* https://github.com/x42/jack-stdio

## Control
* [https://qjackctl.sourceforge.io QjackCtl] - a simple Qt application to control the JACK sound server [[*](https://github.com/rncbc/qjackctl)]
  * https://github.com/brummer10/pajackconnect - script to be invoked via QjackCtl to make JACK with with PulseAudio [[*](https://linuxmusicians.com/viewtopic.php?f=19&t=13234)]
  * https://github.com/SpotlightKid/jack-select - systray application to quickly change the JACK configuration from QjackCtl presets via DBus
  * https://github.com/kmatheussen/qjackctl_macos - script to build on macos
* [Cadence](https://kx.studio/Applications:Cadence) - a set of tools useful for audio production [[*](https://github.com/falkTX/cadence)]
* https://github.com/be1/jackie - a small jack daemon controller
* https://github.com/progwolff/jackman - Collection of scripts that help managing multiple audio interfaces with Jack.
  * https://github.com/progwolff/jackman_kcm - KDE

## WINE
* https://github.com/wineasio/wineasio - provides an ASIO to JACK driver for WINE

# Proposed features
* [[Proposal for MIDI API extension for System Exclusive Messages]]
* [[Proposal for JesseChappellLooping]]