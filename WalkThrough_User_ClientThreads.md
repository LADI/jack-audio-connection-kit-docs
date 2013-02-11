JACK1 was using  a one thread model: real-time code and notifications (graph reorder event, xrun event...) are treated in a unique thread. Indeed the server stops audio processing while notifications are handled on the client side. This has some advantages like a much simpler model for synchronization, but also some problematic consequences: since notifications are handled in a thread with real-time behaviour, a non real-time safe notification may disturb the whole machine. 

In JACK2, the client graph in the server is handled in a lock-free manner. The connection state for instance can be changed without having to stop the audio stream anymore, thus having the good effect of avoiding any audio click while accessing the graph. Because the server audio thread is not interrupted anymore, most of server notifications will typically be delivered while the client audio thread is also running. A two threads model for client has been defined: 

*  a real-time thread dedicated to the audio process 
*  a standard thread for notifications 

This two threads model will possibly have some consequences for existing JACK applications: they may have to be adapted to allow a notification to be called while the audio thread is running.
