# Making the algorithm better

some technical Stuff:

	
	int process (jack_nframes_t nframes, void *arg) {
	
	// get number of frames in the audio fifo (between h/w ptr and snd_pcm_readi() 
	  
	    snd_pcm_delay( alsa_handle, &delay );
	
	
	//  now check if we are out of the max_diff allowed tolerance
	//  and just read or rewind the stream so we get into the tolerance field again
	
	    if( delay > (target_delay+max_diff) ) {
	        snd_pcm_readi( alsa_handle, tmp, delay-target_delay );
	        delay = target_delay;
	    }
	    if( delay < (target_delay-max_diff) ) {
	        snd_pcm_rewind( alsa_handle, target_delay - delay );
	        delay = target_delay;
	    }
	
	    /* ok... now we should have target_delay +- max_diff on the alsa side.
	     *
	     * calculate the number of frames, we want to get.
	     */
	
	// if jack_sample_rate equals our sample rate -> request_samples == nframes
	
	    double resamp_rate = nframes / alsa_samples.
	    double request_samples = nframes / resamp_rate;                        //== alsa_samples;
	
	// how much are we off of the latency we want to acchieve.
	
	    double offset = delay - target_delay;
	
	    double frlen = request_samples + offset;
	    double compute_factor = (double) nframes / frlen;
	
	// compute_factor is the resample factor necessary to make the offset 0 in next process cycle.
	// we now slowly adapt the current_resample_factor so that we dont have sudden samplerate changes.
	
	    double diff_value =  pow(current_resample_factor - compute_factor, 3) / (double) catch_factor;
	
	    current_resample_factor -= diff_value;
	
	
	


now assume some client X which only eats CPU every second process cycle.

connecting X -> alsa_out would result in quite big jitter of the call to snd_pcm_delay which just reads the number of
samples between the hardware data pointer and our write pointer.

the algorithm assumes, that snd_pcm_delay is called in a constant frequency without jitter.
(the jitter to the calls show up as part of the normal measurement jitter of the delay value.

i want to remove this jitter. (like moving the call to snd_pcm_delay right to the entry_point of the alsa_driver after
an audio interrupt occurs)

of course i want to do this by calculating the error of the measurement and leaving the function call where it is now.
so i basically need to know the time consumed in the process function of client X.


[pieterpalmers]
Some random thoughts on this sync stuff...

Suppose you have a method to figure out at what system time the process cycle was called:
	
	   pc_start = jack_get_process_cycle_start_time();
	

in the process call one could do:
	
	     time_pc_start = jack_get_process_cycle_start_time();
	
	     time_now = jack_get_time_now();
	     snd_pcm_delay( alsa_handle, &delay );
	     
	     // suppose we have a rate estimate from before (calculated_rate)
	     // we can say:
	     delay_at_process_start = delay - calculated_rate * (time_now - time_pc_start);
	

This will factor out most of the previous clients CPU time stuff. The main problem with this is that you have to make sure the jack_get_time_now and snd_pcm_delay calls are executed at the same time (atomically).

The calculated_rate value can be obtained by using a standard DLL on the (jack_time T, alsa_hw_ptr_at_time_T) tuples (cfr paper Fons).

Phase alignment can be achieved either by dropping/adding frames (skip based) or by using a different resample factor for a while to catch up (as it is done now). Once phase sync is obtained however there should not be any modification of the resample factor if you want "professional" results. A varying resample factor will cause vibrato style effects / phase modulation on the processed audio. 

This is why I would prefer a (simpler) skip based algorithm that only runs at startup. If the phase alignment gets out-of-bound during normal operation, there is something wrong in the resampling operation, hence it is an error condition and should not be masked. It might periodically be wrong due to jitter, but that should average out on a longer timescale.

# Autodetecting the values

The values we need to autodetect are target_latency, max_diff and the catch factor.
things we could measure to calculate the values:

standard deviation of the measured delay.
granularity of snd_pcm_delay() value. (which is quite high on usb cards)

