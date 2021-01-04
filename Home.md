Welcome to the **JACK Audio Connection Kit** wiki! 

Notice: this wiki is undergoing renovation, not all content is currently up-to-date. Drop into [IRC](https://kiwiirc.com/nextclient/#irc://irc.freenode.net/#jack) for the latest.

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
* [ArchWiki: JACK Audio Connection Kit](https://wiki.archlinux.org/index.php/JACK_Audio_Connection_Kit) - Installation, Configuration, Troubleshooting
* [Past, Present and Future of the JACK Audio Connection Kit](https://media.ccc.de/v/sonoj2019-1902-jack-past-present-future) - video, talk by falkTX at Sonoj 2019
* [Sound Engineers Guide to Jackd](http://www.orford.org/assets/jack-idiots_guide.txt) (jack-idiots_guide.txt)

## Configuration
* [List of JACK Frame & Period settings ideal for USB interface](https://linuxmusicians.com/viewtopic.php?f=47&t=10707) - (Frames/Sample Rate) * Period = Theoretical (or Math-derived) Latency
* [Latency Measurement Results](https://wiki.linuxaudio.org/wiki/jack_latency_results)
* [Balancing Performance and Reliability in Jack](https://www.penguinproducer.com/Blog/2011/10/balancing-performance-and-reliability-in-jack/)
* [List of JACK Frame \[or Buffer\] & Period settings ideal for USB interface](https://wiki.linuxaudio.org/wiki/list_of_jack_frame_period_settings_ideal_for_usb_interface)
* [[Cgroups]] - for a realtime kernel configuration
* [JACK and Headphones](http://tedfelix.com/linux/jack-headphones.html) - duplicate outputs at ALSA level

# Control
* [jack_control walkthrough](https://github.com/jackaudio/jackaudio.github.com/wiki/WalkThrough_User_jack_control) - Python tool to control JACK2 server
* [QjackCtl](https://qjackctl.sourceforge.io) - a simple Qt application to control the JACK sound server [[*](https://github.com/rncbc/qjackctl)]
  * https://github.com/brummer10/pajackconnect - script to be invoked via QjackCtl to make JACK with with PulseAudio [[*](https://linuxmusicians.com/viewtopic.php?f=19&t=13234)]
  * https://github.com/SpotlightKid/jack-select - systray application to quickly change the JACK configuration from QjackCtl presets via DBus
  * https://github.com/kmatheussen/qjackctl_macos - script to build on macos
  * [QjackCtl and the Patchbay](https://www.rncbc.org/drupal/node/76) - A tutorial for the QjackCtl "Patchbay" tool
* [Cadence](https://kx.studio/Applications:Cadence) - a set of tools useful for audio production [[*](https://github.com/falkTX/cadence)]
* https://github.com/ovenwerks/studio-controls - helper for setting up a system for audio work with multiple device GUI
* https://github.com/be1/jackie - a small JACK daemon controller
* https://github.com/progwolff/jackman - collection of scripts that help managing multiple audio interfaces with Jack
  * https://github.com/progwolff/jackman_kcm - KDE
* https://github.com/madskjeldgaard/rofi-jack -  keyboard centric JACK audio management using the rofi app launcher
* https://github.com/redtide/jacksettings -  JACK settings using a jackd based systemd service
* https://github.com/febef/jackwui - web user interface for Linux Jack Audio Server, WIP

## Multiple soundcards
* [jack_load audioadapter](https://linuxmusicians.com/viewtopic.php?p=95025#p95025)
* [ALSA in/out](wiki/WalkThrough_User_AlsaInOut) - use more than one soundcard with jackd
  * https://github.com/IARI/alsa_jack_gui - a qt-based gui to manage alsa_in and alsa_out daemons
* [Zita-ajbridge](http://kokkinizita.linuxaudio.org/linuxaudio/zita-ajbridge-doc/quickguide.html)

## Session management
Both LASH/LADISH and JACK Session are deprecated, and the use of [NSM](https://linuxaudio.github.io/new-session-manager) and an NSM GUI like [Agordejo](https://laborejo.org/agordejo/) is recommended.

NSM clients usually come with `jackpatch`, which automatically saves connections but isn't the most robust of software.

Other suggestions can be found at: https://wiki.linuxaudio.org/wiki/session_management

## Connection management
* https://github.com/SpotlightKid/jack-matchmaker - auto-connect new JACK ports matching the patterns given on the command line
* https://github.com/kripton/jack_autoconnect -  application that reacts on port registrations by clients by creating connections
* https://github.com/gentoo-root/jacklistener - daemon that monitors ALSA jacks plugging/unplugging events and sends signals with these events to D-Bus
* https://github.com/gentoo-root/jacknotifier - daemon that monitors signals from JACK Listener Daemon and shows desktop notifications
* https://github.com/gentoo-root/jackeventcmd - daemon that runs custom commands when headphones are plugged/unplugged
* [jack-plumbing](http://rohandrape.net/?t=rju&e=md/jack-plumbing.md) - implicitly bounded regex rules with sub-expression pattern supportto manage connections as clients register ports with JACK.
* https://github.com/resinbeard/autocable - eats your text files and routes JACK connections for you
  * [Beginning a GNU/Linux/JACK headless performance system](https://llllllll.co/t/beginning-a-gnu-linux-jack-headless-performance-system/1119)
* [ALSA Patch Bay](http://pkl.net/~node/software/alsa-patch-bay) - graphical patch bay for ALSA and JACK that can use FLTK or GTKmm

### Terminal
* [njconnect](https://sourceforge.net/projects/njconnect)

### Graph
* [Patchage](http://drobilla.net/software/patchage) - a modular patch bay for audio and MIDI systems based on Jack and Alsa.
* https://github.com/falkTX/Catia -  Simple JACK Patchbay with A2J integration and JACK Transport controls 
* https://github.com/OpenMusicKontrollers/patchmatrix - a JACK patchbay in flow matrix style

## Connection saving
* [aj-snapshot](http://aj-snapshot.sourceforge.net) - small program that can be used to make snapshots of the connections made between JACK and/or ALSA clients
* https://github.com/dedelala/patchy - store and recall jack audio port connections 
* https://github.com/jacktrip/jmess-jack - save an XML file with all the current connections
* [jack_snapshot](https://sourceforge.net/projects/heaven/files/Audio%20Applications/Jack%20Related/jack_snapshot/) 

## PulseAudio integration
* [JACK and Pulseaudio](wiki/WalkThrough_User_PulseOnJack)
* [PulseAudio and Jack](http://0pointer.de/blog/projects/when-pa-and-when-not.html)
* [How to replace Pulseaudio with Jack, Jack and PulseAudio Together as Friend](https://forum.manjaro.org/t/how-to-replace-pulseaudio-with-jack-jack-and-pulseaudio-together-as-friend/2086) - Manjaro Linux Forum

## Tools
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

* [rd jackd utilities](http://rohandrape.net/?t=rju)
  * jack-data: jack audio data onto osc
  * jack-dl: load dsp algorithms from shared libraries
  * jack-level: jack cli level meter
  * jack-lxvst: jack cli host for linux vst instruments
  * jack-osc: jack <-> open sound control daemon
  * jack-play: resampling soundfile playback
  * jack-plumbing: plumbing daemon
  * jack-record: soundfile recording
  * jack-scope: plain X oscilloscope
  * jack-transport: minimalist ncurses jack transport
  * jack-udp: jack over udp client
* https://github.com/SpotlightKid/jack-audio-tools - collection of utilities and tools for the JACK audio ecosystem
* https://github.com/Gimmeapill/xruncounter - measure jack xruns and evaluate the overall performance of a system
* https://github.com/madskjeldgaard/jack_xruntotal -  display total number of xruns from JACK in a status bar friendly way
* https://github.com/7890/jack_tools - playground for JACK tools and experiments
* [Jacktools - Realtime Audio Processors as Python Classes](https://media.ccc.de/v/lac2018-46-jacktools_realtime_audio_processors_as_python_classes) - video [[*](http://kokkinizita.linuxaudio.org/linuxaudio/downloads/index.html)]
* https://github.com/x42/jack-stdio
* https://github.com/fps/jack2_split - facilitates parallelism in serial JACK graphs by introducing latency, only useful for jack2/jackdmp
* https://github.com/x42/robtk/blob/master/jackwrap.c - x42 jack wrapper / minimal LV2 host
* https://github.com/ponderworthy/MultiJACK - fully operational demo of a framework to increase available audio DSP power available to JACK within a single multicore motherboard, using multiple JACK processes in concert, connected via IP transport. 

# JACK Transport / JACK Timebase
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

echo play | jack_transport
  # pass a command to execute. tempo change doesn't work via this method.
```

# JACK Clients
* [JACK-AUDIO-CONNECTION-KIT: Creating & manipulating clients](https://jackaudio.org/api/group__ClientFunctions.html) - API
* [On JACK client threads](wiki/WalkThrough_User_ClientThreads)
* https://github.com/jackaudio/example-clients
  * https://github.com/jackaudio/example-clients/blob/master/inprocess.c - internal client, runs as part of jackd
* https://github.com/resinbeard/jacksandbox - simple JACK client for learning and testing audio code
zig-jack-examples
* https://github.com/noisesmith/zig-jack-examples - ports of the example JACK audio code to zig

## Playback
* [jplay2](https://gareus.org/oss/jplay2/start) - a command-line audio player that plays a single file, gluing JACK, libsamplerate, liblo (OSC control)
* https://github.com/eeeeeta/mfl-gramophone -  A simple Rust application to play audio, using JACK, when it receives commands via OSC. 

## Recording
* https://github.com/kmatheussen/jack_capture - program for recording soundfiles with JACK
  * [jrec2](http://gareus.org/oss/jrec/jrec2) - patch to jack_capture that implements silence detection and splitting of output files
  * [RecJack](http://mein-neues-blog.de/2015/02/07/mein-neues-blog-deb-repository/#recjack) - a front end for jack_capture
* https://github.com/HoerTech-gGmbH/jack_playrec - provides an interface for synchronous recording/playback via JACK
* [qjackrcd](https://orouits.github.io/qjackrcd/) - stereo recorder with automatic silence pause, file splitting and background file post-processing
* https://github.com/UoC-Radio/audio-coffin -  A simple audio recorder/logger on top of Jack, libsndfile and libsoxr 
* https://github.com/swh/timemachine - JACK application that can retrospectively record audio

## Editing / DAW
* [Audacity](https://www.audacityteam.org/) - multi-track recording and editing
* [Ardour](https://ardour.org/) - the digital audio workstation
* [Zrythm](https://www.zrythm.org) - highly automated and intuitive digital audio workstation 
* [GNUsound](https://www.gnu.org/software/gnusound/) - multitrack sound editor for GNOME 1 and 2
* [Marlin](http://marlin.sourceforge.net) - sample editor for Gnome 2, uses GStreamer
* [ReZound](http://rezound.sourceforge.net/) - graphical audio file editor
* [Snd](https://ccrma.stanford.edu/software/snd/) - a sound editor

## Mixer
* https://github.com/MaurizioB/jack_switch - simple insert switcher for jack clients
* https://github.com/jack-mixer/jack_mixer - a GTK+ JACK audio mixer app with look similar to its hardwarecounterpart.  It has lot of useful features, apart from being able to mixmultiple JACK audio streams.
* [Non Mixer](http://non.tuxfamily.org/wiki/index.php?page=Non%20Mixer) - a powerful, reliable and fast modular Digital Audio Mixer ([LV2 fork](https://github.com/falktx/non))
* https://github.com/kampfschlaefer/jackmix - Matrix-Mixer for the Jack-Audio-connection-Kit 
* https://github.com/jacob3141/mx2482 -  JACK Mixer written with Qt, based on QJackAudio
* [Jackmaster](http://69b.org/cms/software/jackmaster) - "Master Console" for the jack-audio-connection-kit
* [JackMiniMix](https://www.aelius.com/njh/jackminimix/) - simple mixer with an OSC based control interface
* https://github.com/knoellle/audio-multiplexer - interlace two streams of speech while maintaining comprehension. 

## Host / FX
* [Carla](https://kx.studio/Applications:Carla) - fully-featured modular audio plugin host
* [Ingen](https://gitlab.com/drobilla/ingen/) - realtime modular audio system based on LV2
* [JACK Rack](http://jack-rack.sourceforge.net) - an effects "rack" for the JACK

## Samples
* https://github.com/resinbeard/candor - enables a monome 256 user to capture and playback audio from up to 8 channels simultaneously, managing a maximum of 48 remembered wav soundfiles without a computer display.
* https://github.com/jmage619/jmage-sampler
* [Swami](http://www.swamiproject.org/) - for editing, managing and playing musical instruments for MIDI music composition

## Looping
* https://github.com/monocasual/giada - minimalistic and hardcore music production tool

## Synthesis
* https://github.com/Ho-Ro/connie -  Electronic organ (Vox Continental like) with JACK interface

## Metronome
* https://github.com/jmage619/jmetro - dumb JACK based metronome with a Qt based UI.
* https://github.com/TGrif/jack-metronome - gtkmm, sndfile
* https://gitlab.com/tmatth/polygnome - A polyrhythmic metronome

## Timeline
* [gjacktransport](http://gjacktransport.sourceforge.net/) - app for controlling the JACK transport mechanism
* [transporter.py](https://github.com/SpotlightKid/jack-audio-tools/blob/master/transport/transporter.py) / [timebase_master.py](https://github.com/SpotlightKid/jack-audio-tools/blob/master/transport/timebase_master.py) - query and manipulate JACK transport state and provide timebase information using jackclient-python.
* [jacktransportloop](http://danmbox.github.io/jack-file/) - forces the Jack transport to loop between two time-points.
* https://github.com/ycollet/qtmiditrans - translates midi events into jack transport
* [jack-trans2midi](https://sourceforge.net/projects/jack-trans2midi/) - converts jack transport into midi clock messages.
* [JackCtlMMC/QJackMMC](http://jackctlmmc.sourceforge.net/) - allow MIDI Machine Control (MMC) to drive JACK transport.
* https://github.com/rncbc/jack_link - a JACK transport timebase prototype bridge to [Ableton Link](https://www.ableton.com/en/link/).

## DJing
* https://mixxx.org - Free and open source DJ software for Windows, macOS, and Linux

## Tuners
* https://github.com/brummer10/XTuner - simple tuner for JACK including NSM support
* https://github.com/jessecrossen/jackstrobe - simple strobe tuner using JACK and Qt 5

## Effects
* https://github.com/Fishmael/jack-plugins - simple JACK audio plugins written in Java

## Visualisation
* [JACK Meter](https://www.aelius.com/njh/jackmeter/) - a basic console based DPM (Digital Peak Meter) for JACK
* https://github.com/madskjeldgaard/jack_meter_simple - fork of JACK Meter for status bar commands
* https://github.com/gethiox/jack-peak-meter - terminal-based peak-meter for JACK audio system writen in Go 
* https://github.com/xkr47/vu-meter -  Audio VU meter for JACK with any number of channels written in Rust
* [jack_oscrolloscope](http://das.nasophon.de/jack_oscrolloscope/) - a simple waveform viewer for JACK
* https://github.com/alxdb/jack-analyser -  an osciloscope for the JACK Audio API 
* [JACK Meterbridge](http://plugin.org.uk/meterbridge/) - supports a number of different types of meter, rendered using the SDL library and user-editable pixmaps
* [Japa](http://kokkinizita.linuxaudio.org/linuxaudio/japa-pict.html)
* http://sed.free.fr/spectrojack - little spectrogram/audiogram/sonogram/whatever that, using GTK2 and FFTW3. 
* http://spectrum3d.sourceforge.net - displays a 3D audio spectrogram in real time or not, made using Gstreamer, SDL (or Gtkglext), OpenGl, GTK2/3 and uTouch-Geis free libraries
* [JACK Live Spectrum](http://consciousness.mywebcommunity.org/programming/jack-live-spectrum/) - a small C program for Linux that displays a flowing frequency spectrum of live audio
* https://github.com/jcarrano/rtfi - Resonator Time-Frequency Image
* https://github.com/simoncrowe/stylegan2-jack-visualizer - Visualise the spectral density of a JACK client's audio output in real time using StyleGAN 2

## Beamforming
* https://github.com/balkce/beamform - ROS package that carries out simple 1D beamforming strategies, using JACK as input/output audio server.

## Radio
* https://github.com/recri/keyer - iambic/ascii morse code keyer
* https://github.com/UoC-Radio/JMPXRDS - FM MPX signal generator with RDS support on top of Jack Audio Connection Kit 

# CV
* [JACK/LV2 CV - LinuxMusicians](https://linuxmusicians.com/viewtopic.php?f=1&t=20701)

# MIDI
* [JACK-AUDIO-CONNECTION-KIT API: Reading and writing MIDI data](https://jackaudio.org/api/group__MIDIAPI.html)
* [JACK-AUDIO-CONNECTION-KIT API: midiport.h File Reference](https://jackaudio.org/api/midiport_8h.html)
* https://github.com/harryhaaren/JACK-MIDI-Examples
* https://github.com/jerash/jpmidi - command line midi player synced with JACK transport
* [jack-smf-player / jack-smf-recorder](https://sourceforge.net/projects/jack-smf-utils/) - play and record MIDI streams from/to Standard MIDI Files, uses libsmf as a backend.
* https://github.com/x42/jack-midi-cmd - commandline tool to generate JACK MIDI events
* https://github.com/paulguy/crustymidi - JACK MIDI event scripting triggerable on events
* https://github.com/noedigcode/oldnotes - replaces velocities of specified notes with that ofthe previously received non-specified notes
* https://github.com/jmage619/velociraptor -  A Jack midi velocity compressor with Qt based UI.
* https://github.com/x42/jack_midi_clock - utility to send MIDI Beat Clock pulses that follow the JACK Transport 
* https://github.com/chronopoulos/jack-midi-timestamp - simple timing log for JACK MIDI events 
* https://github.com/xitiomet/jack-rtp-midi-bridge - bridge JACK MIDI and RTP MIDI
* https://github.com/falkTX/nooice - joysticks as JACK-MIDI devices
* https://github.com/moddevices/mod-midi-merger - small JACK-internal client to merge MIDI events from several inputs to one output
* https://github.com/falkTX/JackAss - a VST plugin that provides JACK-MIDI support for VST hosts

## Arpeggiation
* https://sourceforge.net/projects/arpage - MIDI Arpeggiator w/ JACK Tempo Sync
* https://github.com/emuse/qmidiarp - an arpeggiator, sequencer and MIDI LFO for ALSA and JACK.

## Keyboard
* https://github.com/brummer10/Mamba - virtual MIDI Keyboard and MIDI file player/recorder/looper for Jack Audio Connection Kit
* [jack-keyboard](http://jack-keyboard.sourceforge.net/) - virtual keyboard for JACK MIDI
* https://github.com/metachronica/audio-midihaskey - virtual MIDI keyboard for microtonal music
* https://github.com/noedigcode/keytarboard - translates keyboard key presses into JACK MIDI notes
* https://github.com/ovenwerks/midikb - keyboard to Jack MIDI program
* https://github.com/Krasjet/kb -  Customizable virtual MIDI keyboard daemon for JACK. 

## Sequencing
* https://github.com/chronopoulos/libsequoia - an API for generative musical sequencing
* https://github.com/rdybka/vht - MIDI tracker

## Looping
* https://github.com/ahlstromcj/sequencer64 - major reboot of Seq24, with new features
* https://github.com/ahlstromcj/seq66 - lean and mean version of Sequencer64 using modern C++ and Qt 5.

## Hardware configuration
* https://github.com/kmtaylor/gi_editor -  A software editor for the Roland JunoGi synthesizer 

## Bridge
* https://github.com/linuxaudio/a2jmidid - ALSA sequencer to JACK MIDI bridging (for jack2)
* https://github.com/free-creations/a_j_midi - Connect ALSA with JACK MIDI
* https://github.com/williamweston/jamrouter - Near-Sample-Accurate JACK <-> Raw MIDI router
* https://github.com/hselasky/jack_umidi - Raw MIDI to JACK daemon 

# OSC
* https://github.com/ventosus/jack_osc - Open Sound Control (OSC) via Jack
* [jack-data](http://rohandrape.net/?t=rju&e=md/jack-data.md)- JACK audio data onto OSC
* https://github.com/x42/jackmidi2osc - tool to generate OSC triggered by JACK MIDI events.
* https://github.com/ssj71/OSC2MIDI - configurable OSC to jack MIDI (and back) bridge
* https://github.com/piegamesde/jackmidiosc -  Midi to OSC bridge using JACK audio, written in Rust
* https://github.com/voidseg/jack-volume -  JACK client for controlling the volume of multiple audio channels via OSC. 

# Networked
* [NetJACK 1](wiki/WalkThrough_User_NetJack) - connect JACK 1 servers via network
* [NetJACK 2](wiki/WalkThrough_User_NetJack2) - connect JACK 2 servers via network
* [Zita-njbridge](https://kokkinizita.linuxaudio.org/linuxaudio/) - transmit full quality multichannel audio over a local IP network, with adaptive resampling by the receiver(s).
  * https://github.com/rhetr/ipaudio -  use jackd, zita-njbridge and systemd for network ip audio
  * https://github.com/nettings/medianet - Raspbian overlay to create a JACK-based embedded system for audio processing and streaming
  * https://github.com/gisogrimm/ov-client - headless clients to share/receive spatial realtime audio using JACK, zita-njbridge and TASCAR
* [JackTrip](https://ccrma.stanford.edu/software/jacktrip)
  * https://github.com/jcacerec/jacktrip
  * https://github.com/noahbailey/jacktrip-docker - Container for JackTrip network audio server
  * https://github.com/noiseorchestra/jacktrip_pypatcher - Python scripts to autopatch a JackTrip hubserver 
  * https://github.com/noiseorchestra/autonomous-noise-unit - Python scripts for running JackTrip on an RPi with OLED screen and rotary switch interface.
  * https://github.com/alastairhmoore/test-jacktrip-wsl -  Test audio generation on wsl played back using host windows 

## Websockets
* https://github.com/lucdoebereiner/klangraum - sum incoming websocket audio streams and send to JACK
* https://github.com/lucdoebereiner/KlangSendWS - node.js websocket server that broadcasts audio (vorbis encoded) from JACK (via ffmpeg) to its clients

# Radio
* https://github.com/mildsunrise/jackpifm - program that runs on your Raspberry Pi, reads live audio from JACK and broadcasts it over FM through the GPIO pin 7 (GPIO #4).

# Hardware
* https://github.com/nettings/gpioctl - Control the ALSA mixer or generate JACK MIDI or OSC messages from rotary encoders and switches connected to GPIOs. 

# Installation and packaging
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
* https://github.com/jackaudio/headers
* [x42/weakjack](https://github.com/x42/weakjack) - small library that abstracts the JACK Application Binary Interface.

## Bindings
* https://github.com/spatialaudio/jackclient-python - JACK Audio Connection Kit (JACK) Client for Python snake
* [PyJack](https://sourceforge.net/projects/py-jack/) - JACK client module for Python
* https://github.com/fphammerle/jacker - Python bindings for the JACK Audio Connection Kit C API
* https://github.com/falkTX/Cadence/blob/master/src/jacklib.py
* https://github.com/jaudiolibs/jnajack - Java bindings to JACK Audio Connection Kit [[*](http://lac.linuxaudio.org/2007/presentations/lac07_slides_gulden1.pdf)]
* https://github.com/free-creations/jackAudio4Java - thin JNI wrapper around the JACK Audio Connection Kit
* https://github.com/umjammer/jjack - Jack-to-Java Audio Driver and Bridge API
* https://github.com/residuum/JackSharp - C# bindings for jackd
* https://github.com/xthexder/go-jack - Go bindings for Jack Audio Connection Kit
* https://github.com/stetre/luajack - requires Lua (>=5.3) and JACK (API >= v0.124.1)
* https://github.com/unclechu/node-jack-connector - Bindings JACK-Audio-Connection-Kit for Node.JS 
* https://github.com/markjfisher/jack-egg - CHICKEN Scheme bindings to Jack Audio 
* https://github.com/dpzmick/rust-easyjack - Some hopefully easy to use Rust bindings for the JACK API, WIP.

## Metadata
* [JACK Metadata](https://jackaudio.org/metadata)
  * [order](https://jackaudio.org/metadata/order)
  * port-group - for stereo pair/ambisonics/etc
  * [pretty-name](https://jackaudio.org/metadata/pretty-name)
  * [signal-type](https://jackaudio.org/metadata/signal-type)
  * [event-types](https://jackaudio.org/metadata/event-types)
  * [connected](https://jackaudio.org/metadata/connected)
  * [hardware](https://jackaudio.org/metadata/hardware)
  * [icon-large](https://jackaudio.org/metadata/icon-large)
  * [icon-small](https://jackaudio.org/metadata/icon-large)
  * [icon-name](https://jackaudio.org/metadata/icon-large)
* [jack2/JackMetadata.cpp at develop · jackaudio/jack2](https://github.com/jackaudio/jack2/blob/develop/common/JackMetadata.cpp#L30)
* https://github.com/drobilla/jackey - simple define-only header that serves as an ad-hoc standard for Jack Metadata property keys

`jack_property` - list, set and delete any metadata associated with the ports and clients of a JACK server

* [jack-property-listener.py](https://gist.github.com/SpotlightKid/dc70dac006337c99bf65bcdc98e43ddc) - listen to and print JACK client/port metadata changes

# Frameworks
* https://github.com/DISTRHO/DPF - designed to make development of new plugins an easy and enjoyable task
* https://github.com/PieterPenninckx/rsynth - Rust API abstraction for API's for audio plugins and applications

# Testing
* https://wiki.linuxaudio.org/apps/all/jack_delay - command line JACK app you can use to measure the latency of your sound card
* https://github.com/falkTX/jack-midi-timing - Sender and receiver tool to test the timing accuracy and jitter of MIDI devices
* http://devel.tlrmx.org/audio/bitmeter - a diagnosis tool for JACK audio software on Linux
* https://github.com/falkTX/jack_interposer - Library to check at run-time whether an application makes non-RT-safe calls inside the process() callback 
* [Zita MU1](http://kokkinizita.linuxaudio.org/linuxaudio/zita-mu1-doc/quickguide.html) - used to organise stereo monitoring
* [SilentJack](https://www.aelius.com/njh/silentjack) - silence/dead air detector for the Jack Audio Connection Kit
* [Rotter](https://www.aelius.com/njh/rotter/) - a Recording of Transmission / Audio Logger for JACK
* https://github.com/jackaudio/tools
* https://github.com/wmedrano/j2sdl2 -  Output to SDL2 Audio From JACK with Rust

# Video
* [XJADEO - The X Jack Video Monitor](http://xjadeo.sourceforge.net/) - a software video player that displays a video-clip in sync with an external time source (MTC, LTC, JACK-transport). Xjadeo is useful in soundtrack composition, video monitoring or any task that requires to synchronizing movie frames with external events.


# WINE
* https://github.com/wineasio/wineasio - provides an ASIO to JACK driver for WINE

# Haiku
* https://github.com/Barrett17/libjackcompat - experimental media_kit compatibility layer

# Proposed features
* [[Proposal for MIDI API extension for System Exclusive Messages]]
* [[Proposal for JesseChappellLooping]]

# See also
* https://github.com/lv2/lv2/wiki