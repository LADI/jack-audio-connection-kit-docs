FAQ
===

See http://jackaudio.org/faq for now.


JACK and JACK-Related Myths Debunked
====================================

**The only way to reliably use jackd requires getting rid of pulseaudio entirely.**
certainly not. see [[http://jackaudio.org/pulseaudio_and_jack]]

**I need to use <code>schedtool</code> or <code>chrt</code> to set up realtime scheduling of each audio application.**
No you do not.
There are countless examples e.g.
[archwiki](https://wiki.archlinux.org/index.php?title=JACK_Audio_Connection_Kit&oldid=243256#A_Shell-Based_Example_Setup)
which demonstrate elaborate scripts to raise the priority of audio-processing applications. In fact they all achieve the opposite.
The fundamental idea behind jack is to only elevate the audio-processing part (jack-callback) of each application and never the application itself. The GUI and user-interface threads of an application should run at normal priority. If the application itself is prioritized it can itself interfere with the audio-processing.


**I need a realtime kernel to use realtime scheduling.**
No. Realtime scheduling is available on all modern Linux systems (>2.6.0) no matter what kernel they use. Processes can be prioritized on any version of Linux, the realtime preemtion kernel only adds reliability. It may be required if you want to run jack at very low latencies. A normal kernel may meet the scheduling requirement most of the time, but a realtime-kernel will try to meet it all the time.
Some feature of the preemt_rt kernel can also help to achieve lower latencies with certain hardware configurations.


**I must disable CPU frequency scaling to get rid of x-runs**
That depends. CPU frequency scaling happens orders of magnitude faster than any audio-processing. There are however two caveats: the CPU frequency governor may not react in time, and on multi-core systems the governor may take overall CPU load into account instead of DSP load. These issues have been addressed by [[http://gareus.org/oss/jackfreqd/]]. NTL if you are not concerned with fan-noise it is a safe bet to disable CPU freq scaling. (this is usually done by calling <code>sudo cpufreq-set -g performance</code> or <code>sudo cpufreq-selector -g performance</code>). You should however disable bus-frequency scaling and C1E halt states and EIST in the BIOS if your BIOS offers those options.


**Is it possible to make LADSPA,LV2,etc plugins run in a separate process, so that any crashes would not bring Ardour to its knees?**
It is possible, but it does not scale. A context-switch would be needed for the process-callback of every plugin.
If you want to run 2 VST plugins like this, it is fine. If you want EQ and compression for every track in an N-track session, its like one JACK client per plugin, and adds notable CPU load.

**Kernel clock frequency affects midi performance/jitter.**
There is no canonical answer to that. The kernel clock freq (<code>dev.rtc.max-user-freq</code>, <code>dev.hpet.max-user-freq</code>) is irrelevant for JACK, JACK-midi and JACK's MIDI-backends including a2jmidid.
However, applications that directly use ALSA-MIDI or ALSA-sequencer may be affected by this. There are many related factors. It depends which timer is used by the soundcard's driver (snd_timer, snd_hrtimer) and how the applications schedules ALSA-MIDI data processing.
