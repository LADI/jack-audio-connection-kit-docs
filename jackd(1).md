# Name

jackd - JACK Audio Connection Kit sound server

# Synopsys

	
	  jackd [options] -d backend [backend-parameters]
	  jackd --help
	

# Description
*jackd* invokes the JACK audio server daemon. The jackstart command provides JACK's built-in support for Linux 2.4.x kernels with the realtime capabilities patch. Use jackd for Linux 2.6.x and all other systems. All parameters are the same.

JACK is a low-latency audio server. Originally written for the GNU/Linux operating system, it also supports Mac OS X and various Unix platforms. JACK can connect a number of different client applications to an audio device and also to each other. Most clients are external, running in their own processes as normal applications. JACK also supports internal clients, which run within the jackd process using a loadable "plugin" interface.

JACK differs from other audio servers in being designed from the ground up for professional audio work. It focuses on two key areas: synchronous execution of all clients, and low latency operation.

For the latest JACK information, please consult the web site, http://trac.jackaudio.org .

# Options

-d, --driver backend [backend-parameters ]

    Select the audio interface backend. The current list of supported backends is: alsa, coreaudio, dummy, freebob, oss and portaudio. They are not all available on all platforms. All backend-parameters are optional. 
-h, --help

    Print a brief usage message describing the main jackd options. These do not include backend-parameters, which are listed using the --help option for each specific backend. Examples below show how to list them. 
-m, --no-mlock
    Do not attempt to lock memory, even if --realtime. 
-n, --name server-name
    Name this jackd instance server-name. If unspecified, this name comes from the $JACK_DEFAULT_SERVER environment variable. It will be "default" if that is not defined. 
-p, --port-max n
    Set the maximum number of ports the JACK server can manage. The default value is 256. 
-R, --realtime

    Use realtime scheduling. This is needed for reliable low-latency performance. On most systems, it requires jackd to run with special scheduler and memory allocation privileges, which may be obtained in several ways. The simplest, and least-secure method is to run jackd with root privileges. This means that all JACK clients must also run as root. With a Linux 2.6 kernel, ordinary users can run jackd and its clients using options of the realtime LSM. Linux 2.4 kernels need "POSIX draft capabilities" enabled (see the <linux/capability.h> include file). Using that method, ordinary users who invoke the daemon using jackstart, can later launch JACK clients without running them as root. See http://jackaudio.org/faq for more information. 
-P, --realtime-priority int
    When running --realtime, set the scheduler priority to int. 
--silent
    Silence any output during operation. 
-T, --temporary
    Exit once all clients have closed their connections. 
-t, --timeout int

    Set client timeout limit in milliseconds. The default is 500 msec. In realtime mode the client timeout must be smaller than the watchdog timeout (5000 msec). 
-Z, --nozombies

    Prevent JACK from ever kicking out clients because they were too slow. This cancels the effect any specified timeout value, but JACK and its clients are still subject to the supervision of the watchdog thread or its equivalent. 
-u, --unlock

    Unlock libraries GTK+, QT, FLTK, Wine. 
-v, --verbose
    Give verbose output. 
-c, --clocksource ( c(ycle) | h(pet) | s(ystem) )
    Select a specific wall clock (Cycle Counter, HPET timer, System timer). 
-V, --version
    Print the current JACK version number and exit.

## Alsa Backend Options

-C, --capture [ name ]
    Provide only capture ports, unless combined with -D or -P. Parameterally set capture device name. 
-d, --device name

    The ALSA pcm device name to use. If none is specified, JACK will use "hw:0", the first hardware card defined in /etc/modules.conf. 
-z, --dither [rectangular,triangular,shaped,none]
    Set dithering mode. If none or unspecified, dithering is off. Only the first letter of the mode name is required. 
-D, --duplex
    Provide both capture and playback ports. Defaults to on unless only one of -P or -C is specified. 
-h, --help Print a brief usage message describing only the
    alsa backend parameters. 
-M, --hwmeter

    Enable hardware metering for devices that support it. Otherwise, use software metering. 
-H, --hwmon

    Enable hardware monitoring of capture ports. This is a method for obtaining "zero latency" monitoring of audio input. It requires support in hardware and from the underlying ALSA device driver.

    When enabled, requests to monitor capture ports will be satisfied by creating a direct signal path between audio interface input and output connectors, with no processing by the host computer at all. This offers the lowest possible latency for the monitored signal.

    Presently (March 2003), only the RME Hammerfall series and cards based on the ICE1712 chipset (M-Audio Delta series, Terratec, and others) support --hwmon. In the future, some consumer cards may also be supported by modifying their mixer settings.

    Without --hwmon, port monitoring requires JACK to read audio into system memory, then copy it back out to the hardware again, imposing the basic JACK system latency determined by the --period and --nperiods parameters. 
-i, --inchannels int

    Number of capture channels. Default is maximum supported by hardware. 
-n, --nperiods int

    Specify the number of periods of playback latency. In seconds, this corresponds to --nperiods times --period divided by --rate. The default is 2, the minimum allowable. For most devices, there is no need for any other value with the --realtime option. Without realtime privileges or with boards providing unreliable interrupts (like ymfpci), a larger value may yield fewer xruns. This can also help if the system is not tuned for reliable realtime scheduling.

    For most ALSA devices, the hardware buffer has exactly --period times --nperiods frames. Some devices demand a larger buffer. If so, JACK will use the smallest possible buffer containing at least --nperiods, but the playback latency does not increase.

    For USB audio devices it is recommended to use -n 3. Firewire devices supported by FFADO (formerly Freebob) are configured with -n 3 by default. 
-o, --outchannels int

    Number of playback channels. Default is maximum supported by hardware. 
-P, --playback [ name ]
    Provide only playback ports, unless combined with -D or -C. Optionally set playback device name. 
-p, --period int

    Specify the number of frames between JACK process() calls. This value must be a power of 2, and the default is 1024. If you need low latency, set -p as low as you can go without seeing xruns. A larger period size yields higher latency, but makes xruns less likely. The JACK capture latency in seconds is --period divided by --rate. 
-r, --rate int
    Specify the sample rate. The default is 48000. 
-S, --shorts

    Try to configure card for 16-bit samples first, only trying 32-bits if unsuccessful. Default is to prefer 32-bit samples. 
-s, --softmode

    Ignore xruns reported by the ALSA driver. This makes JACK less likely to disconnect unresponsive ports when running without --realtime.

## Coreaudio Backend Parameters

-c --channel
    Maximum number of channels (default: 2) 
-i --channelin
    Maximum number of input channels (default: 2) 
-o --channelout
    Maximum number of output channels (default: 2) 
-C --capture
    Whether or not to capture (default: true) 
-P --playback
    Whether or not to playback (default: true) 
-D --duplex
    Capture and playback (default: true) 
-r --rate
    Sample rate (default: 44100) 
-p --period
    Frames per period (default: 128). Must be a power of 2. 
-n --name
    Driver name (default: none) 
-I --id
    Audio Device ID (default: 0)

## Dummy Backend Parameters

-C, --capture int
    Specify number of capture ports. The default value is 2. 
-P, --playback int
    Specify number of playback ports. The default value is 2. 
-r, --rate int
    Specify sample rate. The default value is 48000. 
-p, --period int
    Specify the number of frames between JACK process() calls. This value must be a power of 2, and the default is 1024. If you need low latency, set -p as low as you can go without seeing xruns. A larger period size yields higher latency, but makes xruns less likely. The JACK capture latency in seconds is --period divided by --rate. 
-w, --wait int
    Specify number of usecs to wait between engine processes. The default value is 21333.

## Oss Backend Parameters

-r, --rate int
    Specify the sample rate. The default is 48000. 
-p, --period int
    Specify the number of frames between JACK process() calls. This value must be a power of 2, and the default is 1024. If you need low latency, set -p as low as you can go without seeing xruns. A larger period size yields higher latency, but makes xruns less likely. The JACK capture latency in seconds is --period divided by --rate. 
-n, --nperiods int
    Specify the number of periods in the hardware buffer. The default is 2. The period size (-p) times --nperiods times four is the JACK buffer size in bytes. The JACK output latency in seconds is --nperiods times --period divided by --rate. 
-w, --wordlength int
    Specify the sample size in bits. The default is 16. 
-i, --inchannels int
    Specify how many channels to capture (default: 2) 
-o, --outchannels int
    Specify number of playback channels (default: 2) 
-C, --capture device_file
    Specify input device for capture (default: /dev/dsp) 
-P, --playback device_file
    Specify output device for playback (default: /dev/dsp) 
-b, --ignorehwbuf boolean
    Specify, whether to ignore hardware period size (default: false)

## Portaudio Backend Parameters

-c --channel
    Maximum number of channels (default: all available hardware channels) 
-i --channelin
    Maximum number of input channels (default: all available hardware channels) 
-o --channelout
    Maximum number of output channels (default: all available hardware channels) 
-C --capture
    Whether or not to capture (default: true) 
-P --playback
    Whether or not to playback (default: true) 
-D --duplex
    Capture and playback (default: true) 
-r --rate
    Sample rate (default: 48000) 
-p --period
    Frames per period (default: 1024). Must be a power of 2. 
-n --name
    Driver name (default: none) 
-z --dither
    Dithering mode (default: none)

# Examples

Print usage message for the parameters specific to each backend.

	
	    jackd -d alsa --help
	    jackd -d coreaudio --help
	    jackd -d dummy --help
	    jackd -d freebob --help
	    jackd -d oss --help
	    jackd -d portaudio --help
	

Run the JACK daemon with realtime priority using the first ALSA hardware card defined in /etc/modules.conf.

	
	jackstart --realtime --driver=alsa
	
Run the JACK daemon with low latency giving verbose output, which can be helpful for trouble-shooting system latency problems. A reasonably well-tuned system with a good sound card and a low-latency kernel can handle these values reliably. Some can do better. If you get xrun messages, try a larger buffer. Tuning a system for low latency can be challenging. The JACK FAQ, http://jackit.sourceforge.net/docs/faq.php has some useful suggestions.

	
	jackstart -Rv -d alsa -p 128 -n 2 -r 44100
	

Run jackd with realtime priority using the "sblive" ALSA device defined in ~/.asoundrc. Apply shaped dithering to playback audio.

	
	jackd -R -d alsa -d sblive --dither=shaped
	

Run jackd with no special privileges using the second ALSA hardware card defined in /etc/modules.conf. Any xruns reported by the ALSA backend will be ignored. The larger buffer helps reduce data loss. Rectangular dithering will be used for playback.

	
	jackd -d alsa -d hw:1 -p2048 -n3 --softmode -zr
	

Run jackd in full-duplex mode using the ALSA hw:0,0 device for playback and the hw:0,2 device for capture.

	
	jackd -d alsa -P hw:0,0 -C hw:0,2
	

Run jackd in playback-only mode using the ALSA hw:0,0 device.

	
	jackd -d alsa -P hw:0,0
	

# Environment
JACK is evolving a mechanism for automatically starting the server when needed. Any client started without a running JACK server will attempt to start one itself using the command line found in the first line of $HOME/.jackdrc if it exists, or /etc/jackdrc if it does not. If neither file exists, a built-in default command will be used, including the -T flag, which causes the server to shut down when all clients have exited.

As a transition, this only happens when $JACK_START_SERVER is defined in the environment of the calling process. In the future this will become normal behavior. In either case, defining $JACK_NO_START_SERVER disables this feature.

To change where JACK looks for the backend drivers, set $JACK_DRIVER_DIR.

$JACK_DEFAULT_SERVER specifies the default server name. If not defined, the string "default" is used. If set in their respective environments, this affects jackd unless its --name parameter is set, and all JACK clients unless they pass an explicit name to jack_client_open().

# See Also

* [wiki:jackdrc(5)]
* http://www.jackaudio.org The official JACK website with news, docs and a list of JACK clients.
* http://lists.sourceforge.net/mailman/listinfo/jackit-devel The JACK developers' mailing list. Subscribe, to take part in development of JACK or JACK clients. User questions are also welcome, there is no user-specific mailing list.
* http://www.jackosx.com/ Tools specific to the Mac OS X version of JACK.
* http://www.alsa-project.org The Advanced Linux Sound Architecture.

# Bugs

Please report bugs to http://trac.jackaudio.org/report

# Authors
Architect and original implementor: Paul Davis

Original design Group: Paul Davis, David Olofson, Kai Vehmanen, Benno Sennoner, Richard Guenther, and other members of the Linux Audio Developers group.

Programming: Paul Davis, Jack O'Quin, Taybin Rutkin, Stephane Letz, Fernando Pablo Lopez-Lezcano, Steve Harris, Jeremy Hall, Andy Wingo, Kai Vehmanen, Melanie Thielker, Jussi Laako, Tilman Linneweh, Johnny Petrantoni.

Manpage written by Stefan Schwandter, Jack O'Quin and Alexandre Prokoudine. Wikified by Arnout Engelen