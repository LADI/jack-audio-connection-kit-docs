# Running PulseAudio on top of JACK

If you intend to use consumer applications like Flash or media players in your JACK setup, one possible solution is to use PulseAudio as an intermediate layer for all these non-JACKified programs. Basically, it boils down to the following:

  1. Redirect all ALSA output to PulseAudio
  1. Redirect PulseAudio to JACK

## A: with JACK2 and jackdbus

### Install Cadence

remove QJackQtl if you use that and install Cadence from the KXStudio repositories. From within Cadence you can enable the ALSA and PulseAudio bridges with the GUI.
Done.

## B: with JACK1

**Warning 1**: If you follow this setup there is no control with QJackCtl. Any settings there will have no effect. You will have to make changes to the configuration with jack_control. jack_control has no man page, try jack_control --help. 

 **Warning 2**: After the computer's wake from suspend you will have to restart PulseAudio and possibly the application using it. (If it is a browser plugin using the audio (flash), reloading the page should suffice. If it is html5 audio you will have to restart the browser) However, be aware that doing so might freeze programs connecting to Jack directly.

`pulseaudio -k`

**Warning 3**: It has been said about this method that it is "[the hardest way to do it](http://lists.linuxaudio.org/pipermail/linux-audio-user/2014-August/098427.html) [PulseAudio on JACK]". It probably needs a rewrite. Proceed with care.

### Redirecting ALSA to PulseAudio

Use the following settings in /etc/asound.conf (or $HOME/.asoundrc)

	
	pcm.pulse {
	    type pulse
	}
	
	ctl.pulse {
	    type pulse
	}
	
	pcm.!default {
	    type pulse
	}
	ctl.!default {
	    type pulse
	}
	


### Redirecting PulseAudio to JACK
On Debian-like systems, be sure to install *pulseaudio-module-jack*. (TODO: Add information about Fedora, OpenSuSE and the lot)

Then, use the following configuration if you intend to run jackd all the time:

*~/.pulse/default.pa*:

	
	load-module module-native-protocol-unix
	load-module module-jack-sink channels=2
	load-module module-jack-source channels=2
	load-module module-null-sink
	load-module module-stream-restore
	load-module module-rescue-streams
	load-module module-always-sink
	load-module module-suspend-on-idle
	set-default-sink jack_out
	set-default-source jack_in
	

The following isn't strictly necessary but might be useful. *~/.pulse/daemon.conf*:
	
	default-sample-format = float32le
	default-sample-rate = 48000
	realtime-scheduling = yes
	exit-idle-time = -1
	

If you're using qjackctl as a launcher for jackd.

in settings options add a post startup script like this:

	
	pactl load-module module-jack-sink channels=2; pactl load-module module-jack-source channels=2; pacmd set-default-sink jack_out
