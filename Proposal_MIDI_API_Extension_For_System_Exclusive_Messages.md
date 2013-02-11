# JACK MIDI SYSEX messages -- Problem Description 

JACK MIDI, in its current state, is very good at handling lots of small MIDI messages, and sending/delivering them in sync with audio.  This is absolutely fantastic for the greater majority of JACK clients, which don't have to communicate with external hardware, and most likely have no need for system exclusive messages.  However, JACK MIDI cannot currently handle arbitrarily large system exclusive messages for a couple reasons:

  1. Support for system exclusive messages (and MIDI in general, in some cases) in drivers that provide an interface to physical MIDI ports is poor.  This problem is currently being addressed in JACK 2 (I don't know the state of MIDI drivers in JACK 1).
  2. The MIDI buffers that clients/drivers use to send/receive messages have a fixed size.  Any messages that can't fit in a MIDI buffer are discarded.  This is the problem I'd like to address with this proposal.

One way to solve this problem is to create a separate space in shared memory for any arbitrarily sized MIDI message.  Basically, a space is allocated (hereafter referred to as 'blob') for a MIDI message in shared memory, which is then populated by a client, and copied to the buffer.  When a MIDI message containing a blob is copied to the port buffer, the *address* of the MIDI blob is copied, and not the data itself.  Since the data is in shared memory, it can be read by clients that receive the MIDI message without making extremely large port buffers.

Of course, since there's lots of data to be read, there also needs to be a way for the blobs to be read over several cycles, or possibly in separate threads.  Since the data is in shared memory, there's also a need to make sure that clients that receive messages containing MIDI blobs can't easily modify the data in the blob.  As if that weren't enough, we also need to make sure that there's a way for the client that 'owns' the blob to know when other clients are done accessing the blob object.

__Notes__:

On the mailing list, Paul asked:

  is JACK MIDI an appropriate API for this sort of thing?

Good question! Here's a list of examples of application types and use cases for which using the JACK MIDI API to do such a thing would be practical:

## MIDI sequencers

  A MIDI sequencer plays MIDI sequences, which may include system exclusive messages.  An SMF (Standard MIDI File) can potentially contain *very* long system exclusive messages, though it's probably more common for SMF's to contain shorter system exclusive messages for doing simple things like configuring MIDI-capable hardware before sending realtime sequences to the device.  A notable exception to this rule are SMFs that are meant to be played over the MIDI port of a piece of hardware that are used to upgrade firmware, though this isn't exactly a realtime MIDI issue.

  There are several MIDI sequencers that use the JACK MIDI API.  Examples include:

  *  [dino](http://dino.nongnu.org/)
  *  [Epichord](http://evanr.infinitymotel.net/epichord/)
  *  [Jacker](http://bitbucket.org/paniq/jacker/wiki/Home)
  *  [Non-Sequencer](http://non-sequencer.tuxfamily.org/)
  
  (Edit: Not all of the above applications can load SMF files.  SMF is an example of a common format that can potentially contain system exclusive messages, but that doesn't stop other sequencers that don't support SMF files from playing back system exclusive messages, and doesn't stop other formats from support system exclusive messages.)


## MIDI monitors

  A MIDI monitor is used to take a close look at MIDI messages being sent to a MIDI port.  MIDI-capable devices often send system exclusive messages to communicate with software or other hardware, but there isn't always documentation on the format or purpose of these system exclusive messages.  A MIDI monitor can capture and save these messages, giving the developer a tool for figuring out the format and purpose of such messages.

  There is at least one MIDI monitor application that uses JACK MIDI:

*  [gmidimonitor](http://home.gna.org/gmidimonitor/)

## Software emulations of MIDI-capable hardware

  System exclusive messages are received from and sent to MIDI hardware.  Software emulations of such hardware, in addition to responding to other, more common MIDI messages, may very well want to respond to the system exclusive messages that the hardware it's trying to emulate responds to.

  There is at least one very-well known program that uses JACK MIDI and emulates several hardware synthesizers:

  *  [Bristol](http://bristol.sourceforge.net/index.html)

  I don't know if Bristol responds directly to system exclusive messages sent over JACK MIDI, but, according to the Bristol site, "... the GUI is really just a master keyboard for the engine and drives it with MIDI SYSEX messages over TCP sessions".


##  MIDI routers

  Applications that route MIDI data to/from MIDI-capable plugins loaded in the application itself ([LV2](http://lv2plug.in/), [VST](http://en.wikipedia.org/wiki/Virtual_Studio_Technology), [DSSI](http://dssi.sourceforge.net/) to a lesser extent)

  Applications that load plugins that can receive and send MIDI messages don't know what sorts of system exclusive messages any plugin will be capable of sending/receiving.  However, such plugins should be capable of receive/transmitting such messages successfully.

  There are lots of applications that are capable of loading plugins that receive MIDI messages.  I'm not so sure how many enable the sending of messages as well.

  XXX: I don't have time to form such a list right now.  If someone would like to take care of this, then that would be awesome.

# First Proposed Solution

The first solution involves the introduction of two new opaque data types; `jack_midi_blob_t` and `jack_midi_blobstream_t`:

	
	    typedef struct _jack_midi_blob jack_midi_blob_t;
	    typedef struct _jack_midi_blobstream jack_midi_blobstream_t;
	

New functionality for creating, accessing, and destroying blobs would
be necessary:

	
	    // Not RT-safe.
	
	    jack_midi_blob_t *
	    jack_midi_blob_create(jack_client_t *client, size_t size);
	
	    // Not RT-safe.
	
	    void
	    jack_midi_blob_free(jack_midi_blob_t *blob);
	
	    // RT-safe.
	
	    size_t
	    jack_midi_blob_size(const jack_midi_blob_t *blob);
	

A 'blobstream' object is used to read and write data from/to 'blob' objects.  The 'blobstream' API resembles a file API for the sake of familiarity.  A blobstream object, when it opens a blob using `jack_midi_blobstream_open_read`, will add a reference to the blob it uses, and will subtract a reference when `jack_midi_blobstream_close` is called.  All calls using 'blobstream' objects are RT-safe, and return 0 to indicate success, or a non-zero error code:

	
	    int
	    jack_midi_blobstream_open_read(jack_midi_blobstream_t *stream,
	                                   const jack_midi_blob_t *blob);
	
	    int
	    jack_midi_blobstream_open_write(jack_midi_blobstream_t *stream,
	                                    jack_midi_blob_t *blob);
	
	    int
	    jack_midi_blobstream_read(jack_midi_blobstream_t *stream,
	                              jack_midi_data_t *dest, size_t count);
	
	    int
	    jack_midi_blobstream_seek(jack_midi_blobstream_t *stream, size_t pos);
	
	    int
	    jack_midi_blobstream_tell(jack_midi_blobstream_t *stream, size_t *pos);
	
	    // 'jack_midi_blobstream_write' is not valid for 'blobstream' objects opened
	    // using 'jack_midi_blobstream_open_read'.
	
	    int
	    jack_midi_blobstream_write(jack_midi_blobstream_t *stream,
	                               const jack_midi_data_t *src, size_t count);
	
	    int
	    jack_midi_blobstream_close(jack_midi_blobstream_t *stream);
	

To enqueue blobs in MIDI port buffers, the definition for `jack_midi_event_t` will have to be altered, and MIDI port buffers will have to grow a method to enqueue MIDI blobs:

	
	    typedef struct _jack_midi_event {
	        jack_nframes_t time;
	        // 'size' is set to 0 when the event contains a blob
	        size_t size;
	        union {
	            jack_midi_data_t *buffer;
	            const jack_midi_blob_t *blob;
	        };
	    } jack_midi_event_t;
	
	    int
	    jack_midi_event_write_blob(void *port_buffer, jack_nframes_t time,
	                               const jack_midi_blob_t *blob);
	

Finally, JACK needs a way to communicate to JACK clients that their blobs are no longer being accessed by streams.  One way to do this is with callbacks:

	
	    typedef void (*JackMIDIBlobCallback)(jack_midi_blob_t *blob, void *arg);
	
	    int
	    jack_set_midi_blob_callback(jack_client_t *client,
	                                JackMIDIBlobCallback midi_blob_callback,
	                                void *arg);
	

__Pros__:

  * JACK clients don't have direct access to a blob's shared memory due to the opaque type.

__Cons__:

  * Setting the 'size' member of 'jack_midi_event_t' objects to 0 to indicate the presence of a blob is a bit of weirdness.
  * Blobstreams create an extra layer of complexity.

__Notes__:

On the mailing list, Torben noted:

>  i am not sure if we should do this implicit memory management.  maybe allocating a blob, and then putting the blob id into the midi event would be smarter.
>
>  we still need to manage references to blobs.  and we need an RT safe memory management in shm. (which probably needs to be threadsafe too.... tricky :)

# Second Proposed Solution

The second solution assumes a blob is just a `jack_midi_data_t *` created in shared memory.  This solution assumes that the jack_midi_event_t struct would not need to be modified in order for the 'buffer' member to reference shared memory.  My reading on shared memory indicates that this is okay, but I can't be totally sure because I don't have a thorough understanding of the internals of JACK.

To start, we'd keep functionality for creating and destroying MIDI blobs, only a MIDI blob is now a `jack_midi_data_t *` that points to a segment of shared memory:

	
	    jack_midi_data_t *
	    jack_midi_blob_create(jack_client_t *client, size_t size);
	
	    void
	    jack_midi_blob_free(jack_midi_data_t *blob);
	

MIDI buffers need to be taught how to enqueue blobs.  The 'size' argument specifies the amount of the blob that contains an actual message:

	
	    int
	    jack_midi_event_write_blob(void *port_buffer, jack_nframes_t time,
	                               const jack_midi_data_t *blob, size_t size);
	

Dealing with references to blobs is a little different than in the first solution.  The JACK server would need a way to map shared 'jack_midi_data_t *' objects to reference counts.

We need to know if a particular `jack_midi_data_t *` is a blob:

	
	    int
	    jack_midi_is_blob(const jack_midi_data_t *data);
	

(Perhaps this should take a `port_buffer` argument too.  Not sure.)

A client can acquire and release references to blobs if the client needs to access the blob for more than one cycle, or in a separate thread:

	
	    int
	    jack_midi_blob_acquire(jack_client_t *client,
	                           const jack_midi_data_t *data);
	
	    int
	    jack_midi_blob_release(jack_client_t *client,
	                           const jack_midi_data_t *data);
	

(Maybe the client arguments aren't necessary.)

Finally, we have callbacks to let clients know when a blob has no more clients using the blob:

	
	    typedef void (*JackMIDIBlobCallback)(jack_midi_data_t *blob, void *arg);
	
	    int
	    jack_set_midi_blob_callback(jack_client_t *client,
	                                JackMIDIBlobCallback midi_blob_callback,
	                                void *arg);
	

__Pros__:

  * Most JACK clients won't need to change anything they're doing right now.  The added functionality doesn't introduce any kludges for clients (i.e. size == 0).

__Cons__:

  * JACK clients have access to shared memory segments associated with MIDI buffers created by other clients.  Perhaps changing the 'jack_midi_event_t' 'buffer' member to be of type 'const jack_midi_data_t *' would be helpful, but that may break some clients.

__Notes__:

This is the solution I prefer.  It's cleaner and more elegant for clients.  However, I don't know what sorts of challenges this presents for JACK 1 and JACK 2.
