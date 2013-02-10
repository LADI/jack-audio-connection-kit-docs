Having just implemented seamless looping in ardour, I find myself                                         
wanting a way to use this in conjunction with the jack                                                    
transport system.   This has been touched upon in the past, so                                            
here is my proposal for extension:                                                                        
                                                                                                          
Reintroduce two new items into jack_position_t's padding space:                                           
	
	    jack_nframes_t         loop_start;                                                                    
	    jack_nframes_t         loop_end;                                                                      
	
                                                                                                          
Introduce a new api call:                                                                                 
	
	    jack_transport_start_loop()                                                                           
	
                                                                                                          
which will do exactly the same thing as jack_transport_start                                              
except the state (after slow-sync procedure) will be JackTransportLooping                                 
(already defined).                                                                                        

                                                                                                          
jack_transport_reposition() will be used as-is to set/modify the                                          
loop boundaries, and proper slow-sync procedure is still used                                             
whenever loop bounds change while JackTransportLooping.                                                   
                                                                                                          

The key element of the JackTransportLooping state is that jack                                            
will wrap the frame position appropriately after the cycle containing                                     
or ending on the loop_end.   All clients who can handle the looping                                       
state will already be aware of the loop boundary from their last                                          
slow-sync at reposition/start and are expected to have already made                                       
preparations for the seamless wraparound.                                                                 
                                                                                                          

On the last cycle in a given loop, the starting frame will be correct,                                    
but due to the loop_end, the effective transport frame after the                                          
loop_end within the cycle will wrap to loop_start                                                         
-- all clients are expected to handle this intra-cycle mod-arithmetic                                     
internally.  Jack will apply the proper mod-math at the start                                             
of the next cycle.                                                                                        
                                                                                                          

The only issue i foresee is with clients who support the jack                                             
transport but do not know about the JackTransportLooping state.                                           
Depending on their implementation, their results could be unpredictable.                                  
If they are simple (fast-sync) clients, most likely they will just run                                    
past the loop_end in the last cycle, only to be corrected at the                                          
start of the next cycle.  If they are slow-sync, well... they                                             
could quite easily become confused, or they might ignore the                                              
looping altogether.....                                                                                   
                                                                                                          

Anyway, as you can see, the jack-side implementation of this is                                           
next to trivial.  Most of the onus is upon the clients to make                                            
jack-wide seamless looping a workable reality.      

State transitions are as follows.                                                                         
	                                                                                                          
*  JackTransportStopped                                                                                   
	  -> jack_transport_start_loop()                                                                          
	     ifloop bounds legal then normal sync                                                                 
	     operation follows.....                                                                               
	  * JackTransportStarting                                                                                 
	  * JackTransportLooping                                                                                  
	                                                                                                          
	--------------------------------------                                                                    
* JackTransportLooping                                                                                    
	  -> jack_transport_stop()                                                                                
	  *  JackTransportStopped                                                                                 
	                                                                                                          
	-------------------------------------                                                                     
* JackTransportLooping                                                                                    
	  -> jack_transport_reposition()                                                                          
	     loop bounds legal (start must be strictly less than end),                                            
	     normal sync operations follow:                                                                       
	     * JackTransportStarting                                                                              
	     * JackTransportLooping                                                                               
	                                                                                                          
	-------------------------------------                                                                     
* JackTransportLooping                                                                                    
	  -> jack_transport_reposition()                                                                          
	     but loop bounds are *illegal* (start must be strictly less than end).                                
	     * JackTransportStopped                                                                               
	                                                                                                          
	 [instead of stopping, we could also transition into rolling]                                             
	                                                                                                          
	--------------------------------------                                                                    
* JackTransportLooping                                                                                    
	  -> jack_transport_start()                                                                               
	     normal sync operations follow:                                                                       
	     * JackTransportStarting                                                                              
	     * JackTransportRolling                                                                               
	                                                                                                          
	--------------------------------------                                                                    
* JackTransportRolling                                                                                    
	  -> jack_transport_start_loop()                                                                          
	     if loop bounds are legal, normal sync operations follow:                                             
	     * JackTransportStarting                                                                              
	     * JackTransportLooping                         
	

This thanks to David's mail archives - even more discussion at http://lists.jackaudio.org/private.cgi/jack-devel-jackaudio.org/2010-January/004381.html                                                      