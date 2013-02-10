	
	#LegendBox
	#!legend: Work in progress
*This page is under construction...*
	


----

----

# Introduction

----

Netjack2 is a Jack2 version of Netjack, a powerful tool allowing to send audio and midi data to a distant computer just as if it were a simple jack client.
The current version is still in development and doesn't already include all the features proposed by netjack.
Indeed, the original concept has been completely redesigned to better fit to the Jack2 architecture, but also in order to provide additional capabilities, and ultimately a greater robustness.
The purpose of this walkthrough is to give you the necessary basics to a simple use of Netjack2.

If your browser made you land on this page, this is no coincidence, Jack2 is probably what you're looking for...If you don't know what it is about, why don't you start with [http://jackaudio.org]. You'll find there all the information you need. Else, read the following to know everything about Netjack2...

----

# Summary

----

1. Concepts[[BR]]
2. Requirements[[BR]]
3. Installation[[BR]]
4. How to set up the master ?[[BR]]
5. How to set up the slave ?[[BR]]
6. Why can't I use my soundcard on a slave ?[[BR]]
7. Why do I miss packets ?[[BR]]
8. What about transport ?[[BR]]
9. What's next ?[[BR]]


----

## 1. Concepts

----

  a. Main concept 

Imagine that you have two computers (Linux, OSX, Windows or Solaris), fully operational for audio. You run your favorite synths, sampler and other stuff on the first one, a fast processor and a lot of memory inside; and you start your favorite audio/midi sequencer on the other computer, which has large disks and great audio hardware. You may thing, how could I plug those two systems together without any audio wiring or d<->a conversion ?

Then you probably think, ho, I made my best to get the cleanest local network of the neighborhood, I set up firewalls, proxys, etc., and for now, I have no data to send on it...
Don't worry, you didn't think twice for nothing...Netjack and now Netjack2 allow you to connect two 'jackified' computers as you are used to. On the first one, you see the second like a simple jack client, with its audio and midi ports you can connect to any other jack port as you always did.

 But how does it works ?

Netjack2 is based on a Master/Slave communication.
One master can handle several slaves. In fact, a separate software element manages all the slaves. This element, called *Net Manager*, is the only thing you have to launch to activate Netjack2.
On the slave side, you just have to launch a classical jack server. The only difference is this server will be launched with the 'net' backend. It means the driver isn't an audio hardware driver, like _alsa'', ''coreaudio_ and others ; but it is a driver which directly deals with the network instead of an audio card.

Here is a picture of the global architecture of Netjack2. You can see on the left the Net Manager, which has one master for each slave. Thus you can interface every available jack port of the slave just in connecting it in the master's jack client.

  [[netjack2_jack.png]]

----

Once launched, the slave will be automatically seen and 'captured' by the manager running on the master. You don't have to know by advance any IP address or network hostname, everything is done in a fully transparent way.
Of course we assume that your network is properly configured and working.
The things are made easier because it uses multicast network transmissions, allowing a computer to tell to anybody who wants to hear it 'hey, I'm available, use me, I like it...' or 'hey, I'm going away, bye'. That's what the Net Manager does : simply listen to a multicast stream, and when a slave appears and send something, the manager knows what to do, create a new master for the new slave, or delete an existent master when a slave is gone.

In this picture, you can see what happens on the network during the different initialization and exiting processes. You can also notice the multicast communication channel, through which the _available'' and ''exiting_ messages travel.

  [[netjack2_init.png]]

----

  b. Synchronous vs. asynchronous

Netjack2 is based upon Jackmp, so the server can run backends with two running modes : synchronous or asynchronous.
In Sync mode, the data are written to the driver's output ports just after the graph execution. That means the computed data are available directly at the end of the current cycle. Thus in Netjack2, data are just sent back to the Master with no additional latency.
If you just make a loop on the slave' side (simply connect the driver's output to input), you will get your audio stream back to the master in the same cycle.
The compensation to that is the master have to wait for those streams in its own process.
In Sync mode, the master is blocked in its process until the slave send data.

Here is a schematic showing that :

  [[netjack2_sync.png]]

In Async mode, the graph execution is done after the driver's 'read' and 'write'. That means the driver 'read' data (the Net Driver receive data from the network), then 'write' the previous cycle computed data (the Net Driver send data over the network) and then launch the graph execution.
During this time, the master just send data, and immediately after receive the previously computed data from the slave. No need to wait here for data anymore.
In Async mode, the master isn't blocked in its process, so it's faster, but the received data from the slave unfortunately have a one cycle additional latency.

Here is how things are doing in async mode :

  [[netjack2_async.png]]

----

  c. What about network latency

Sending data over a network implies an additional latency. Let's try to quantify it in this case...
We're talking here about the network bandwidth, which is the amount of data the network is able to carry in one time unity.
If we take one audio cycle as a time unity, we have a kind of network audio bandwidth, in bytes per cycle.
That means if we have a network audio bandwidth of 500kbytes per audio cycle, and we just have 500kbytes of data to send over this network, sending those 500kbytes will take all the time unity, so all the audio cycle duration. That means we have an additional network latency that can reach one audio cycle.

So starting from this consideration, we can imagine two cases :

- the Master doesn't want to wait for current data, considering the current cycle's data will arrive right in the next cycle, waiting for it will block the jack server on this process. So just don't try to receive current data, and just look to the network reception buffer if some data (normally from the last cycle) is available, then just receive it, this is a fast operation.
This is what constitutes the normal Netjack2 network mode : in a given cycle, the master just wait for previous cycle data.
- the Master want to wait for current data, because we want a zero latency system, and we know the network bandwidth will not be filled, so data can arrive quickly. So wait some time to 'return' data from the slave. The master will block the jack server, but normally, not too long because data are supposed to be received in less than one cycle.
This is the fast Netjack2 network mode : in a given cycle, the master just wait for the 'return' data.

The fast mode is a parameter of the slave, because this mode depends more of the slave behavior than the master's. Fast mode is quite dangerous because in this mode, nothing is guarantee, and if, on the master' side, you are running a lot of process, maybe you won't have the material time to wait for the return data. The fast mode directly implies a variable CPU load for the master, because while waiting, jack process is interrupted. It's a good way to run processes on a faster computer, with a fast network (gigabyte), but its use is at your own risks...


----

## 2. Requirements

----

The smallest architecture you can set needs two computers (Linux, Macosx or Windows) with network capabilities (100mbit/s is a good start).
Netjack2 needs one UDP port (default is 19000), and also multicast networking (default IP is 225.3.19.154), so first check your network equipment will not block multicast or filter UDP.
That is very important that your firewall don't block ICMP messages or UDP Port you're using. Blocking ICMP turn the system unable to react in case of network failure (physical disconnection or strong network errors). Netjack2 uses low level error detection in order to keep a realtime behavior. We can't use a whole system of connection error detection, based upon acknowledgments and other things, that's why we just detect a connection failure when the network operation (send or receive) fails. Blocking ICMP disable the network operations capabilities of detecting such errors. 

If you use Gigabyte with 'jumbo frames', you can set a larger MTU for the system. This is discussed in a few lines.

Wireless and Internet use are not supported because they can't be qualified as _realtime''. In Netjack, you can set a additional latency which allow you to prevent from larger transmission delays, due to the use of networks with some kind of ''random delivery_, like wireless or Internet networks. Netjack2 doesn't include this feature for now.
That's why we recommend the use of classical wired network.

But network isn't everything, and you also need to have a proper installation of Jack2 (current SVN integrating the latest developments of Netjack2 is recommended), with a working realtime system. If you can launch Jackd with realtime, you probably won't have any problem to set up a functional Netjack2 system.

So that's it, two computers running Jackd with realtime, on the same local network, let's install and try it...

----

## 3. Installation

----

Netjack2 is fully integrated in the Jack2 package. It is automatically installed with Jack2. You don't have anything to do to get it working.
You can get and install Jack2 from SVN with :
	
	  svn co http://subversion.jackaudio.org/jack/jack2/trunk
	
  On Linux, configure, build and install using :
	
	  ./waf configure
	  ./waf build
	  ./waf install (in root)
	
  On MacOSX, you need to build it using XCode to build it as a framework, or you can use Waf to make a classical 'Unix' installation. It is not included in the JackOSX package yet.

  On windows, you can build it using the Code::Blocks project. You can make the small installer using the 'jack.ci' script with the small freeware 'CreateInstallFree'. More info in the 'Readme' from the windows folder.

----

## 4. How to set up the master ?

----

Before setting everything up, just be sure of what you want to do. For now, the Master is the only computer able to capture or playback an audio stream from/to any piece of audio hardware (audio soundcard etc.).
So you have to keep in mind that actually, you can't make any transaction with any audio hardware into the slave computer (in Netjack, ALSA In and ALSA Out allow an audio hardware to register in the slave as a regular jack client, this in not implemented in Netjack2 yet).
So the main idea is to consider the slave as a source of processing resources. You can run a synth on it, send midi data and get generated audio stream back on the master; or you can send a multichannels audio stream, apply your favorite resource eater audio treatment and get the audio back on the master, again. Or you can just use the slave to play some audiofiles, or...or...we let you imagine what you want to do with it.

So once you have chosen which one of your computer will be the master, let's set it up...

First launch a classical Jackd server. You can set it as you want, use the hardware you want to etc. The only thing you have to set rationally is the period size.

To understand why, let's have a look to how the data are sent over the network...

Imagine you want to send and receive for audio channels over a network. The MTU of the network gives you the maximum size in bytes that you can build your packets of. You remove a few dozen for a header, which identify the packet as a Netjack packet, and you get the maximum _audio packet size_.
So imagine we can't put 1400 bytes in one packet. We have four channels, at a sample rate of 48kHz, each audio sample takes four bytes.
We then have 16 bytes per audio frames. We can put 1400/16 frames in one packet. We take the _power of two_ directly under this value, to better fit to the audio world needs, what gives us 64 frames per packet.

If you take a period size of 2048, you will have to send and receive 32 packets per cycle. That means more packets sent together, so more opportunities to loose some of them.
So keep in mind this simple fact : the smaller period size you set, the less packets you have __per cycle__ (you still have the same amount of data to transmit), added, of course, to a smaller latency.

If you use ALSA and want to send midi from your hardware (keyboard, control surface etc.), you can add the -X raw or seq option to the ALSA driver.

Net Manager is an internal client that will run in the server process space. Once your jack server launched, just load the Net Manager with this :

	
	 jack_load netmanager
	

This will load the Net Manager into your jack server.
And that's all you have to do. You don't have to set any name or parameters.

If you need, for particulary reasons to set another Multicast address or UDP Port, you can use :
	
	  jack_load netmanager -i "-a xxx.xxx.xxx.xxx -p port"
	

Alternatively, internal clients can be configurated and loaded using the jack server DBUS service, when Jack2 has been compiled with DBUS support (see http://trac.jackaudio.org/wiki/WalkThrough/User/jack_control).

You have to be careful because NetJack doesn't check if the Multicast IP or the UDP Port is correct. So if you have some network error (typically 'Can't bind multicast address : Invalid Argument' or something else), first check if it's a valid multicast address and an authorized free port.

Use the following command to see all possible parameters :

	
	  jack_load netmanager -i "-h"
	

Now, let's add some slaves...

----

## 5. How to set up slaves ?

----

Slaves are simple Jack server running under the _net_ backend.

To launch it, just type :

	
	  jackd -R -d net
	

The -R option is required to ensure realtime operations. You can choose to use the Net Driver in Sync or Async mode (with '-S').
The Sync mode doesn't add extra-latency, but that means the Master's process have to wait for the Slave's data before to return.

You can add some specific options, here is a full list :

	
	Parameters for driver 'net' (all parameters are optional):
	        -a, --multicast_ip      Multicast Address (default: 225.3.19.154)
	        -p, --udp_net_port      UDP port (default: 19000)
	        -M, --mtu               MTU to the master (default: 1500)
	        -C, --input_ports       Number of audio input ports (default: 2)
	        -P, --output_ports      Number of audio output ports (default: 2)
	        -i, --midi_in_ports     Number of midi input ports (default: 0)
	        -o, --midi_out_ports    Number of midi output ports (default: 0)
	        -n, --client_name       Name of the jack client (default: 'hostname')
	        -t, --transport_sync    Sync transport with master's (default: 1)
	        -m, --mode      Slow, Normal or Fast mode. (default: slow)
	

If you specify a Multicast IP and a UDP Port on the master's side, you have to set the same options here, on the slave, with the -a and -p options.
The -m option set the MTU of the Master/Slave networking path. The best way to set the MTU would probably be by doing a discovery using the _path MTU discovery_ technique. The fact is this technique is quite controversial because it uses ICMP messages, which are sometimes ignored by network devices such as routers or firewalls (for security reasons). So for now, there is no such technique, and MTU is manually set. If you know the MTU of your network (could be 9000 with jumbo frames for example), you can change this value. It has a direct influence on the maximum packet size, and so on the number of packets. If you don't know what to set, keep the default value (classical value for 100mbit Ethernet networks). The fast mode allows the master to wait for the current cycle slave's return data. This means a zero latency transmission, but the compensation to that is the master will wait some time for this data.

You can have this options list while typing at any time :

	
	  jackd -d net --help
	

The default configuration is a simple stereo use, with no midi port.

----

## 6. Why can't I use my soundcard on a slave ?

----

At this point, you probably think  Ok, that's great, but why don't use the soundcard on the slave ?

There is a reason for it, something in connection with synchronization.

You probably know that if you take two computers, and make them run an audio software, or a Jack server at a given sample rate, they are obviously not running exactly at the same sample rate. That is because they don't have exactly the same _master clock_.
This is the greatest inconvenience that all the digital audio world has ever fought.

In the Netjack system, no problem of synchronization because the slave isn't running an audio hardware. It's simply led by a network stream. the incoming stream delivers 64 frames, for example, so the slave have to deal with those 64 frames, then send back 64 frames. There is no other time consideration than the master's cycle last.

But if you want to make these 64 frames goes to an audio hardware, you will have to resample because the master's cycle duration will not fit to the new arbitrary slave's.
Master and slave have no other way of syncing each other (except for hardware which include _wordclock_ or some other kind of wired sync).

Netjack2 includes a system allowing to resample the network stream to send it on a piece of audio hardware. This is done using an in server client called _audioadapter''. After the slave has been started using the ''net_ backend, load the audioadapter client using jack_load:

	 
	  jack_load audioadapter
	

By default this client will open the same number of input/output ports the net driver has opened and will use the sampling rate the net backend is currently using. Resampling is done to compensate for small clock drift that exist between the master audio card clock and the slave audio card clock. Thus the slave audio hardware will stay in sync with the network stream. 

To hear what is received from the network on the slave machine soundcard, just connect _system'' (that is the net driver) and ''audioadapter_ jack ports, using jack_connect (or any more sophisticated connection tool):

	 
	jack_connect system:capture_1 audioadapter:playback_1
	jack_connect system:capture_2 audioadapter:playback_2
	

Obviously the same kind of connection between the _system'' and ''audioadapter_ ports have to be done to send the soundcard input audio streams on the network.  You can use the following:

	 
	  jack_load audioadapter -i "-h"
	

to print possible parameters for the audioadapter component (will be printed on stdout) and then gives parameters when loading the audioadapter, like:

	 
	  jack_load audioadapter -i "-P4 -C4"
	

to load the audioadapter with 4 input/output JACK ports.

Alternatively, internal clients can be configurated and loaded using the jack server DBUS service, when Jack2 has been compiled with DBUS support (see http://trac.jackaudio.org/wiki/WalkThrough/User/jack_control).

----

## 7. Can I use master synchronized on it's audio card and slave also synchronized on it's audio card ?

----

This setup can also be done. But as explained previously, this will cause some synchronization issue.  In this setup you'll need to use another component called _netadapter_. It is an in-server client that will connect to the master machine, and does resampling as needed to adapt with the sample rate and buffer size used on the master machine.

After the slave has been started using the standard _audio_ backend (ALSA, CoreAudio, OSS...), load the netadapter client using jack_load:

	 
	  jack_load netadapter
	

To hear what is received from the network on the slave machine soundcard, just connect _netadapter'' and ''system_  jack ports, using jack_connect (or any more sophisticated connection tool):

	 
	jack_connect netadapter:capture_1 system:playback_1 
	jack_connect netadapter:capture_2 system:playback_2 
	

You can use the following:

	 
	  jack_load netadapter -i "-h"
	

to print possible parameters for the netadapter component (will be printed on stdout) and then gives parameters when loading the netadapter, like:

	 
	  jack_load netadapter -i "-P4 -C4"
	

to load the netadapter with 4 input/output JACK ports.
----

## 8. Why do I miss packets ?

----

Loosing a packet in a network is something completely normal. That's why TCP has been imagined, that's also the reason to be of a big part of network protocols.

But here, we do something quite unusual since we want to keep a true _realtime_ transmission.
We want to send our data to a distant computer, but we want to do it in a very short timing window, classically the period size duration.
We also want to get the data back just during the next cycle, not after.

That's why Netjack use UDP. UDP allow to just send, and receive data as it comes. No acknowledgment or verification is needed because it would take some time we haven't.

So in order to maintain the realtime aspect, we can't avoid packet loss. The better way to minimize it is to keep the cleanest network infrastructure, prefer switch to hub etc.

----

## 9. What about transport ?

Netjack includes the 'transport of transport'. You can start a sequencer on the slave, and by this way synchronize everyone in the network.

----

## 10. What's next ?

The next Netjack2 developments are :
* second version allowing _interslave_ communication (imagine a big network in which any netjack'ified computer see every others, and where you can connect any jack port to any other without going through the master...)
* any great idea from the interested user you are...

----
