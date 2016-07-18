# Introduction
So ? You got started with jack, you can run it and play with your favorite JACK clients, and now you'd like to go a step further: sending audio data to other JACK clients over a network ! Let's see what we've got in store...

## ...NetJack1 !
NetJack is a functionality of jack which enables you to send and receive JACK data (audio and/or midi) on an IP network. It is a simple toolset, that gives you quite a lot of power. The purpose of this walkthough is to give you the necessary basics to use NetJack1.

It now even supports lossy compression, so it's possible to use it over internet.

## Basic terminology used in this document
 **JACK:**
 If you're stuck there, I suggest you read http://jackaudio.org first :-)

 **NetJack:**
 It is the generic name used for the functionality covered in this document.

 **Master JACK (aka MJ):**
 The JACK server running with a regular ALSA backend. You can connect multiple slave JACKs to it.

 **Slave JACK (aka SJ):**
 A JACK server running with the net backend. It can only be connected to one master JACK at a time.

# Getting started
NetJack is part of jack1 since version 0.116.0. If you need access to the audiocard on the slave machine,
i recommend using the svn, because these tools got significantly better. Also some serious bugs have been fixed.

Another alternative is use the port of netjack1 to jack2, which is in jack2 since 1.9.5.
 

## Who is who in my NetJack setup ?
TODO: We need a decent graph there...

## Deciding what will be sent over the wire
Just as a regular JACK ports, the ports you will use to send you data over the network will not be inactive when they are silent. there is *always* data to send even if it's silence. That is a basic rule you'll have to keep in mind to get the best out of NetJack. Thus take a minute and think about what needs to go through the network and what doesn't. If data needs only to flow in one way, you won't need both playback and capture ports.

A simple example: You've got 2 machines. One is hosting your decent sound card connected to your shiny audio gear, and let's say the other one is a war machine capable of processing crazy stuff without any dropouts. Imagine you want to play one of your huge ardour session on your war machine while routing the output to the machine with the good sound card. To minimize network traffic you'll only need to setup 2 audio ports going from the slave JACK (the war machine) to the master JACK (the one with the sound card).

Note: ok, you could just plug the audio interface in the war machine. Would that be fun ? No...

## All set ? Let's play !
For your first experiments, you might want to try using a direct link between the 2 computers

1. Start JACK on the master machine as you usually do (using a JACK controler like qjackctl, pyjackctl, or on the command line)
2. Start JACK with the net backend on the slave machine  
`jackd -R -d net`  
or for jack2  
`jackd -R -d netone`  
For jack1: Remember not to connect any clients to the slave JACK until we've connected both machines ! This limitation is gone with jack2, however, some clients might crash, when jack is changing
the buffer size.
3. Launch 'jack_netsource' on the master JACK to connect your slave:  
`jack_netsource -H <slave jack hostname>`

You're set ! If you've used NetJack's defaults, you should have 2 playback ports, 2 capture ports, 1 midi in and 1 midi out running between the master and the slave. Go ahead, launch a few clients and give your NetJack its first drive.

You can run qjackctl or any other patchbay on the slave to connect things, when all is running.


# All Options

Here are the options of the jack_netsource program:

	
	usage: jack_netsource [options]
	
	  -h this help text
	  -H <slave host> - Host name of the slave JACK
	  -o <num channels> - Number of audio playback channels
	  -i <num channels> - Number of audio capture channels
	  -O <num channels> - Number of midi playback channels
	  -I <num channels> - Number of midi capture channels
	  -n <periods> - Network latency in JACK periods
	  -p <port> - UDP port that the slave is listening on
	  -r <reply port> - UDP port that we are listening on
	  -B <bind port> - reply port, for use in NAT environments
	  -b <bitdepth> - Set transport to use 16bit or 8bit
	  -c <kbits> - Use CELT encoding with <kbits> kbits per channel
	  -m <mtu> - Assume this mtu for the link
	  -R <N> - Redundancy: send out packets N times.
	  -e - skip host-to-network endianness conversion
	  -N <jack name> - Reports a different name to jack
	  -s <server name> - The name of the local jack server
	
	
	

And the complete list of options for the netjack driver backend obtained by _jackd -d net --help_
Note that for the jack2 port it's currently _jackd -d netone --help_

Most options are autodetected, you normally dont need to specify any of them.
	
	        -i, --audio-ins         Number of capture channels (defaults to 2) (default: 2)
	        -o, --audio-outs        Number of playback channels (defaults to 2) (default: 2)
	        -I, --midi-ins  Number of midi capture channels (defaults to 1) (default: 1)
	        -O, --midi-outs         Number of midi playback channels (defaults to 1) (default: 1)
	        -r, --rate      Sample rate (default: 48000)
	        -p, --period    Frames per period (default: 1024)
	        -n, --num-periods       Network latency setting in no. of periods (default: 5)
	        -l, --listen-port       The socket port we are listening on for sync packets (default: 3000)
	        -f, --factor    Factor for sample rate reduction (deprecated) (default: 1)
	        -u, --upstream-factor   Factor for sample rate reduction on the upstream (deprecated) (default: 0)
	        -c, --celt      sets celt encoding and kbits value one channel is encoded at (default: 0)
	        -b, --bit-depth         Sample bit-depth (0 for float, 8 for 8bit and 16 for 16bit) (default: 0)
	        -t, --transport-sync    Whether to slave the transport to the master transport (default: 1)
	        -a, --autoconf  Whether to use Autoconfig, or just start. (default: 1)
	        -R, --redundancy        Send packets N times (default: 1)
	        -e, --native-endian     Don't convert samples to network byte order. (default: 0)
	        -J, --jitterval         attempted jitterbuffer microseconds on master (default: 0)
	        -D, --always-deadline   Always wait until deadline (default: 0)
	

The defaults are to wait for a Packet from jack_netsource, and then configure with the necessary parameters.
(One exceptions is the -c option, which is not autoconfigured, and has to be set on both ends.)

For jack1:

You cant connect clients while jackd is waiting for the autoconf packet.
This can be overcome, by passing -a0 and setting up in the right way beforehand.
Jackd will start immediately in freerunning mode, similar to jackd -d dummy. It will
sync to the clocksource once it's there.

jack2 will just not have system port until a master connects. The system ports will also
diaappear after some time running without a master.


# Bandwidth requirements

The bandwidth used for every channel transmitted via netjack is dependend on the sample rate,period size and
number of channels. A good rule of thumb is to assume 2Mbit/s per channel for 48kHz.

So with 100 Mbit/s you could theoretically transmit 50 channels. If you just link 2 computer with an xover cable,
you will have a full duplex connection. So there will be no packet collisions. In that case the theoretical maximum
would be 50 channels in both directions.

In Order to max out a link you need to run the netsource with -l3 because the audio data needs one period to travel one way,
so 2 periods for travel, and 1 for processing on the slave.

A wireless connection for example, will show collisions. Collisions are not detected on wireless Networks. So the
packets will be lost. Also note that the transfer speed of a wireless card almost never reaches the advertised values.

In my experiences it's only possible to fill half of the links bandwidth. After that the netjack connections will start to exhibit
network xruns. Setting -l 2 when invoking jack_netsource leviates this problem. But at the cost of 2 periods of roundtrip latency. (Note that the default is -l5, but for normal Channel counts on ethernet -l1 is just fine)

For an internet connection you will need a much higher latency setting. In case you configered a too low latency, netsource will print
out some latency values. Look for the lowest one you see, and add 1 or 2... but just adding 10 to be on the safe side is fine too.

# CELT

jack-0.116.1 brings support for the celt codec.
You need at least celt-0.5.0 which you can obtain from www.celt-codec.org.
But i _recommend_ the git version, because 0.5.0 contains a bug resulting in
denormals when both ends are not connected.
The celt bitstreams, are still changing. So you need to make sure, that both
ends are using the same version.

It's a low-latency lossy codec. So if you want to use netjack over internet, this is now possible.

use a period size between 128 and 512. then give -c <kbit_per_channel> to both jackd -d net (or jackd -d netone) and jack_netsource.


alsa_in and alsa_out also got better.


# The soundcard on the slave

jackd -d net means: jackd should use the network interface as the soundcard and not alsa. So you will not see the soundcard of the slave in
your jackd patchbay.
If you want to access the soundcard of the slave machine: Thats what `alsa_in` and `alsa_out` are for, see [WalkThrough_User_AlsaInOut].

