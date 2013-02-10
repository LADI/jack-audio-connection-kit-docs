

# Introduction

the programs alsa_in and alsa_out enable you to use another soundcard with jackd.
Note that on a NetJack Slave the soundcard is effectively your Network Card.


# Use Cases

* using 2 or more soundcards not synced via wordclock.
* using the soundcard of the NetJack slave
* using jackd -d dummy and dynamically add/remove soundcards

The tools handle the resampling and synchronisation required. However this makes clear
that the result is not as perfect as that of the real soundcard which is clocking the
whole jack graph.

So you should not record through these tools, and never use it as a monitor for mastering or such.

Also these tools impose some additional latency. (I was able to reduce that latency into the 20 frames
range by tweaking the parameters, but see the notes below on usb devices)


# Quick Start

just start 

	
	alsa_out -d hw:1
	

connect using your favourite patchbay. I guess you can hear the resampling now, but you are using the default values,
which were set to make it output something in almost any case.

In most cases you can set the catch factor to 10 000 or 100 000 even to obtain much better sound quality.


	
	alsa_out -d hw:1 -f 30000
	

and now the input of the card

	
	alsa_in -d hw:1 -f 30000
	


# Tuning Latency

Now lets try to reduce the latency because the defaults are 1024 frames which is a lot.
the options -l and -m are the latency in frames the program should approach
And the max diff is the number of frames which the equvalent to an xrun occurs.

max diff MUST be smaller than the latency.

	
	alsa_out -d hw:1 -f 30000 -l 128 -m 64
	

This should work nicely for example.


# Complete option List

	-j <jack name>::
	  reports a different name to jack
	-d <alsa_device>::
	-c <channels>::
	-p <period_size>::
	-n <num_period>::
	-r <sample_rate>::
	-m <max_diff>::
	-t <target_delay>:: 
	-f <catch_factor>::

Please note that the period size and numperiods dont directly realate to
the latency of alsa_in and alsa_out. 
The tools use a different interface to alsa then jackd.

# Some more info

So... you came this far. And now it gets complicated.
The tools measure the position of the hardware audio pointer each time they get called during
the processing of the jack graph. Then it tries to determine the clock drift and resampling factor
from this measurement.

For usb cards the granularity of this pointer is not 1 but the size of the data packet sent.
This makes the whole calculation less accurate. So you cant set the latency and max diff as
low as you would like.

To obtain better results you must use a usb card as jacks native card and use alsa_out with the
pci or internal card. 

Using jackd -d dummy has another effect: the clock of the dummy has a lot of jitter.
So the measurement times also have more jitter and you need to use more latency.

There is some potential for improvement of this situation, so perhaps this problem can be solved.


# Report results

Please report working settings and latencies you have obtained so we can get more insight and make
this stuff better. And spread the word. The Mailing Lists are full of posts saying, that its not
possible to use jackd with 2 unsynced soundcards. And NetJack exists for over 2 years now.
It was not really changed, it only got integrated into jacks source tree.

