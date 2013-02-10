The JACK transport and timebase mechanism is powerful, but has a couple of limitations which will be explained here.

These limitations make the JACK transport mechanism unusable for looping. The currently recommended workaround is not to make the timebase master aware of any looping at all: it is considered 'internal' to each client separately, which present themselves to JACK as if playing one big song. No transport relocations are used at all in this scenario.

This is not a particularly good way of doing things, as it might for example be hard to make sure the loops are of equal length in each client, but it is a solution that can be used right now, and does not depend on the relocation api which suffers from the limitations detailed below.

See also:
* http://lists.jackaudio.org/private.cgi/jack-devel-jackaudio.org/2010-January/004310.html
* http://jackaudio.org/files/docs/html/transport-design.html
* http://jackaudio.org/files/docs/html/group__TransportControl.html

# No real-time relocation/looping

Some clients cannot 'instantly' relocate in rt, as they may (for example) need to read information from disk. They can pause relocation.

If you'd want to sync 'looping' by repeatedly relocating the jack transport, you'll need a way to do this in a seamless way. Some ways to do that:
* give clients 'advance notification' of relocations, so they're ready when it happens. This notification might come in the form of a simple 'we're going to relocate to location X next time we hit location Y'-event.
* give clients access to an entire 'loop/relocation map' so they can anticipate relocations
* add a new 'looping' state and add fields with start- and end locations to [jack_position_t](http://jackaudio.org/files/docs/html/structjack__position__t.html) as per the [JesseChappellLooping looping proposal by Jesse Chappell].

A drawback to syncing looping at all: suppose you have a song where the drum machine loops a 2-bar pattern, an accompaniment generator loops a 16-bar accompaniment, and a melody synth plays a 32-bar melody. The drum machine and accompaniment generator will still have to do 'internal' looping. Perhaps the 'current workaround' of just letting time roll on isn't so bad. The only potential real problem with that approach seems to be the danger of drift (which might occur when clients disagree about the exact loop length, for example due to different rounding techniques).

See also:
* http://lists.jackaudio.org/private.cgi/jack-devel-jackaudio.org/2010-January/004328.html
* http://lists.jackaudio.org/htdig.cgi/jack-devel-jackaudio.org/2009-March/002363.html
* http://lists.jackaudio.org/htdig.cgi/jack-devel-jackaudio.org/2009-April/002567.html

# relocations are always performed at block boundaries

A relocate currently always happens between process() calls. 

Solution directions: inside process(), a client should be able to tell whether a relocation is due during this block, and if so, act accordingly.

# Tempo changes are underspecified

The current implementation might actually be sufficient to support tempo changes, but the API is not entirely clear on how this should work.

There are 2 ways of doing tempo changes: sharing a 'tempomap' among all synchronized applications, so that each can convert to frames from BBT/BPM information and vice-versa, or by putting the Timebase Master in charge of keeping the tempomap and keeping the clients up-to-date.

There is currently no recommended mechanism/format for the former. The latter approach might be suitable, but is not well-documented yet.

## 'frame' is required when relocating

Assuming we try and take the 'only the Timebase Master can relate BBT/BPM to frame numbers' approach.

Suppose, in the presence of tempo changes, you're playing measure 10 of a song. A timebase client wishes to move to measure 40. The client has no way of knowing any future tempo changes, so he can't determine the frame number corresponding with measure 40.

Yet, 'frame' is currently a required field in [jack_position_t](http://jackaudio.org/files/docs/html/structjack__position__t.html) - there is no flag to mark it invalid in [jack_position_bits_t](http://jackaudio.org/files/docs/html/transport_8h.html#a64608154318de05af9e763bfb5fb8529). 

Solution directions: adding a bit for 'valid' to [[jack_position_bits_t](http://jackaudio.org/files/docs/html/transport_8h.html#a64608154318de05af9e763bfb5fb8529)] would be backwards-incompatible - it might be better to choose a 'magic value' (such as '-1') to specify this field should be ignored. Masters who cannot reposition based on only BBT information could return EINVAL, in that case perhaps a client might try again with a best-effort frame guess included.

See also: http://lists.jackaudio.org/htdig.cgi/jack-devel-jackaudio.org/2009-March/002348.html

# Variable speed

Variable speed means speeding up/slowing down playback. The difference with 'tempo change' is that 'tempo change' will generally be assumed to compensate so that things like pitch remain constant, while for 'variable speed' this is not assumed. This would for example be used to support relocation by jogwheel. This is not expected to be supported.