Welcome to the **JACK Audio Connection Kit** wiki! 

Notice: this wiki is undergoing renovation, not all content is currently up-to-date. Drop into IRC for the latest.

# User Resources
* [[jackd(1)]]
* [[jackdrc(5)]]
* [[Differences between jack1 and jack2]]
* [[FAQ and myths]]
* [[Troubleshooting]]
* [[Mailing lists]]

# Walkthroughs
* [Demystifying JACK – A Beginners Guide to Getting Started with JACK](https://libremusicproduction.com/articles/demystifying-jack-%E2%80%93-beginners-guide-getting-started-jack) - Libre Music Production
* [jack_control walkthrough](https://github.com/jackaudio/jackaudio.github.com/wiki/WalkThrough_User_jack_control) - Python tool to control JACK2 server
* [ALSA in/out](wiki/WalkThrough_User_AlsaInOut) - use more than one soundcard with jackd
* [NetJACK 1](wiki/WalkThrough_User_NetJack) - connect JACK 1 servers via network
* [NetJACK 2](wiki/WalkThrough_User_NetJack2) - connect JACK 2 servers via network
* [JACK and Pulseaudio](wiki/WalkThrough_User_PulseOnJack)
* [PulseAudio and Jack](http://0pointer.de/blog/projects/when-pa-and-when-not.html)

# About JACK
* [Past, Present and Future of the JACK Audio Connection Kit](https://media.ccc.de/v/sonoj2019-1902-jack-past-present-future) - Filipe Coelho, 2019
* [[JACK Transport support]] and [[JACK Transport limitations]]
* [On JACK client threads](wiki/WalkThrough_User_ClientThreads)
* [[Cgroups]] - for a realtime kernel configuration
* [http://www.orford.org/assets/jack-idiots_guide.txt Sound Engineers Guide to Jackd] (jack-idiots_guide.txt)

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

# JACK tools
* https://github.com/jackaudio/tools
* https://github.com/jackaudio/example-clients
* https://github.com/Gimmeapill/xruncounter - measure jack xruns and evaluate the overall performance of a system

## Control
* https://github.com/rncbc/qjackctl - a simple Qt application to control the JACK sound server
* https://github.com/falkTX/cadence - a set of tools useful for audio production

# Proposed features
* [[Proposal for MIDI API extension for System Exclusive Messages]]
* [[Proposal for JesseChappellLooping]]