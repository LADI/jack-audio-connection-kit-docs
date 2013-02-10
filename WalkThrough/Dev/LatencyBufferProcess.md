_Most information on this page is either plain wrong or dangerous half-truth. It is retained because it serves as a bad example i n Ross Bencina's excellent article http://www.rossbencina.com/code/real-time-audio-programming-101-time-waits-for-nothing._

This page discusses the theory of real time sound processing: latency, buffer sizes and process callback model.

This page is intended for you if:
* You use Jack but don't know how to parameter it
* You aim to develop an effect plugin and don't understand the process callback method model

The theory is not Jack specific and can be applied to other real time processing too (video for example).

[[PageOutline]]


# Callback model

When you process sound using Jack then your program will work in callback mode. That means you don't open a processing thread but your code will be called on Jack's own processing thread. Effects must do processing on the callback method provided by the effect:

int process(jack_nframes_t nframes, void *arg);

This way the processing latency will be the possible minimum.

# Latency
Small latency is important when using Jack to host a MIDI instrument or an audio effect of a guitar for example. If latency is too big, the sound will come out of the system decades after the musician has hit the string of the guitar.

# Buffer underrun
If the synchronous data flow can not be achieved because a processing step does not finish in time then the output buffer of line-out will become empty. So some silence or random samples will be sent to the D/A converter. Both are recognized as noise when amplified...

# Example of processing using the callback model

Processing audio data with a line in, two plugins and a line out happens this way:

1. A/D converter of your sound card has processed a new data packet on line in. The size of packet is determined by Jack configuration. You set it through command line or qjackctl.
1. Plugin one's process method is executed. After executing it will produce its output on its output buffer.
1. _as soon as_ plugin1 finished plugin2's process method is executed. After executing it will produce output on its output buffer.
1. The output buffer is passed to the line out D/A converter of your sound card. The D/A converter will consume samples in its own frequency one by one.

# Theory of latency and buffer
Jack system works in periods. In each period it processes a fixed size buffer full of data.

Let's say the length of buffer in samples is N ant the time of a buffer is T. Each period processes N samples and the periods start when the input buffer is filled by data after sampling input for T.

## Factors of latency
The latency of this system is built from the following factors:

* The packet length is a latency. If all plugins could pass the audio data in 0 time then the output could be played as soon as the input buffer is filled so the minimum theoretical latency would be the buffer's length (measured in time). This latency is always T.
* Plugin processing time is not zero. So playback of the buffer can not be started as soon as the input buffer is ready. This gives a second latency factor. This factor is a free decision. This is the output latency. If you select it to be too small the chance of underrun will be high.
* Some effects (such as fft based effects) analyse sound in blocks. They can not produce output (as a result of a sample) until the whole buffer is filled. So the time until the buffer is filled will cause a latency.

## Connection between processing time and buffer underrun

    * As an average processing of N samples must be finished in T time. If not the next package will arrive before the last is processed. Some processing modules use the same amount of processor ticks in each period, but others don't. I don't know whether Jack supports processing periods that take longer than T but the next periods are shorter in return. In theory an output latency and some spare place in input buffer (the one that the A/D converter fills) can tolerate this kind of processing time jitter.
      You have to find the ideal compromise between output latency and chance of buffer underrun.

# Real time kernel

* If you have installed a real time correctly the audio processing will be run in a real time thread. That means no lower priority threads will be scheduled to execute until processing of the samples is finished. Other tasks will run in the remaining time of the T period. It has drawbacks too:
   * if you add too much effects then your computer will be overloaded and you can not event send a kill command. So it hangs. Jack may detect overload and stop the process itself. I don't know whether this mechanism is built in or not.
   * A buggy effect with an indefinite loop can lock your system. A dual core processor may help...
   * It does not work fine enough. Especially on my box playback using some effects went fluid. Then I have started moving a window around using the mouse and buffer underruns happened. So the graphical subsystem of my box had higher priority than audio processing. This does not mean that the whole concept is bad but means that it is hard to configure and buggy drivers may wreck all your effort.

# Asynchronous data flow and ringbuffer

* Some processing is not clever to be done on a real-time thread. Some processing use real big buffer sizes, others wait for inpredictible devices:
   * exact frequency measurement of a guitar tuner uses huge block sizes
   * playback of audio from disc - ogg or mp3 sometimes wait for IO
   * recording of audio to disc
* An example recording module that saves samples in a file:
   * Create a ringbuffer (a buffer that is read and written on two seaparate threads. Both read and write locks the buffer so it the pointers of the buffer will be maintained consistent)
   * Size of the ringbuffer must be big enough not to overrun
   * create a thread that reads the ringbuffer and write all data found to disc. When it finds the ringbuffer is empty just sleep the process for a short period. When the thread wakes up the ringbuffer will contain some samples again :-). This thread will only block the real-time thread while it copies data from the ringbuffer. The priority of this porcess should be raised to real-time during the copy. I don't know whether Jack does it or not automatically.
   * The jack process callback just reads data from its input and write it to the ringbuffer. It has no other task.

