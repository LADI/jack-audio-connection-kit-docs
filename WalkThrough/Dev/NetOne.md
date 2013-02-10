
# How is NetJack1 operating

The old netjack implementation was pretty straight forward.
jack_netsource sends out a period of audio data with a sequence number S. And then tries to read the data for S-latency.

The driver just reads the current period on the wire, triggers a process cycle and sends back a reply with the same sequence number. 

If the Audio data does not fit into the MTU, one period is split up into several packets, they all carry a netjack Packet header,
only the fragment field is different.

audio data is not interleaved. its just [channel1] [channel2] ... [channelN]
obviously the last packet is a bit smaller than the others.


In order to handle packet loss, netjack needs to be able to give up waiting for a packet, which will not come.
this is achieved by using poll with a timeout on the incoming socket fd.

simply put: if this timeout expires, and the packet did not come, netjack will trigger a process cycle, and the capture ports will still contain
old data. (with celt, celts packet loss concealment code will render something less disturbing) 

	
	    while(1) {
	        // this returns false when there is no complete packet in the cache.
	        if( packet_cache_get_next_available_framecnt( global_packcache, netj->expected_framecnt, &next_frame_avail) ) {
	            // now check if thats the frame we want.
	            if( next_frame_avail == netj->expected_framecnt ) {
	                we_have_the_expected_frame = 1;
	                // normally we break here. (but by still waiting for the deadline, we make can compensate a lot of the link jitter.
	                //                          this helps alsa_out to maintain stable opration)
	                // but i repeat. for LAN we break now, and trigger a process cycle.
	                if( !netj->always_deadline )
	                        break;
	            }
	        }
	        // no data on the socket, and the deadline has not passed yet.
	        // so we poll. when no data is received the timeout will hit.
	        // and we will break;
	        if( ! netjack_poll_deadline( netj->sockfd, netj->next_deadline ) ) {
	            break;
	        }
	        
	        // this reads out all available data from the socket, and put it into the reassembly cache.
	        packet_cache_drain_socket( global_packcache, netj->sockfd );
	    }
	


the tricky part is calibrating this timeout. so when the slave sends out a reply, it fills a field in the packet header, which contains the time from
time the packet was sent to the deadline. The netsource is then able to calculate when this packet would have been received, when it was sent at the deadline.
i called this value the deadline_goodness. 

this deadline goodness is sent back to the slave with every packet. the slave will check this value and adjust its deadline a bit to aproach a value which
seems to work fine. (its currently a function of the period_size and the latency parameter)



joerns version:

	
	struct _jacknet_packet_header
	{
	    // General AutoConf Data
	    jack_nframes_t capture_channels_audio;
	    jack_nframes_t playback_channels_audio;
	    jack_nframes_t capture_channels_midi;
	    jack_nframes_t playback_channels_midi;
	    jack_nframes_t period_size;
	    jack_nframes_t sample_rate;
	    jack_nframes_t wire_format;
	
	    // Transport Sync
	    jack_nframes_t sync_state;
	    jack_nframes_t transport_frame;
	    jack_nframes_t transport_state;
	
	    // Packet loss Detection, and latency reduction
	    jack_nframes_t framecnt;
	    jack_nframes_t latency;
	
	    jack_nframes_t reply_port;
	    jack_nframes_t mtu;
	    jack_nframes_t fragment_nr;
	};
	

necessary for every fragment:

	
	    jack_nframes_t framecnt;    // maybe reduce to 16bit. but we need to be careful, to fix all compare operations for wrap.
	    jack_nframes_t fragment_nr; // 16bit easy. 8bit possible. BUT: need to calc max chancount. with low mtu settings.
	

absolutely necessary values master->slave:
	
	    jack_nframes_t deadline_goodness;   // not sure about 16bit here.
	    jack_nframes_t transport_frame;     // 32bit. basta :)
	    jack_nframes_t transport_state;     // 3 bits.
	

absolutely necessary values slave->master:

	
	    jack_nframes_t sync_state;        // one bit.
	    jack_nframes_t time_to_deadline;  // 16bit might be critical.
	


