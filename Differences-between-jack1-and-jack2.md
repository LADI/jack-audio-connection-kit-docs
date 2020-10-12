# FAQ: what are the differences between JACK1 and JACK2

**The short answer:** JACK1 and JACK2 are equivalent implementations of the same protocol. JACK2 was originally written to add multi-processor support and to rewrite the internals using C++.  Programs compiled against JACK1 will work with JACK2 without recompile (and vice versa with a few exceptions).  JACK2 was originally planned to replace JACK1, but there is a convoluted story behind it... As of right now, JACK1 is in bugfix-only maintenance mode while JACK2 is in active development.

**Which one should I choose?**  Use JACK2, as that is one being actively worked on and maintained. If you're feeling brave, try PipeWire.

**Are we going to have two JACKs forever?**  No.  falkTX now maintains both, and as declared JACK2 to be the one to go for.  The current plan is to implement all JACK1 features into JACK2 and then slowly phase out JACK1 into "bugfix-only" mode (see the JACK2 v1.9.12 release notes for more detailed information).  This feature-parity is pretty much complete now, with a few exceptions that will be finalized soon.

**So what about the version numbers ?** JACK1 _approaches_ version 1.0. At the time of writing it is at version 0.121.3. JACK2 approaches version 2.0 - at the time of writing it is at version 1.9.15. run `jackd --version` to find out the version that you have installed.


# Detailed Differences

Here is a feature listing/comparison of the different JACK implementations.  Note that this list does not include tschack, Torben Hohn's experimental branch of JACK1.

| **Feature**                                                |  **JACK1**  | **JACK2** |
|------------------------------------------------------------|:------------:|:----------:|
| Implements the JACK C API                                  |  Yes         |  Yes       |
| Supports multiple processors (SMP)                         |  No          |  Yes(1)    |
| Allows apps to connect/disconnect without disrupting audio |  No          |  Yes       |
| Interacts with PulseAudio on Linux to share soundcard      |  No(6)       |  Yes       |
| Has (optional) DBUS support(2)                             |  No(6)       |  Yes       |
| Can be used with [[WalkThrough_Dev_NetOne]]                |  Yes         |  Yes       |
| Can be used with [[WalkThrough_User_NetJack2]]             |  No          |  Yes       |
| Is under active development                                |  Yes(7)      |  Yes       |
| Supports Linux/*nix/POSIX OS's                             |  Yes         |  Yes       |
| Supports macOS                                             |  Yes         |  Yes(4)    |
| Supports Windows                                           |  No          |  Yes(5)    |
| Supports Solaris/OpenSolaris                               |  Yes?        |  Yes       |
| Has builtin support for >1 soundcard on Linux              |  Yes         |  No        |
| Supports metadata API                                      |  Yes         |  Yes       |
| Has builtin integration with Linux MIDI                    |  Yes         |  No        |
| Has builtin integration with CoreMIDI (macOS)              |  No          |  Yes       |

(1) JACK2 can run in 2 different modes : "asynchronous" when the server does not wait for graph end for a given cycle but just write the outputs computed at the previous cycle. In this case an extra period of latency is added.  JACK2 can also be run in "synchronous" mode, when the server waits for the graph activation end in a given cycle, in which case it works like JACK1.  
(2) DBUS support helps in integrating with [PulseAudio](http://www.pulseaudio.org/).  
(3) It's not that netjack2 is exclusively for JACK2, it's just that nobody has ported the protocol to JACK1.  
(4) Integrated with [JackOSX](http://www.jackosx.com/) which allows CoreAudio clients become JACK clients (thanks to the JackRouter CoreAudio/JACK virtual audio device)  
(5) ASIO clients can become JACK clients (thanks to the JackRouter ASIO/JACK bridge)  
(6) DBUS and PulseAudio interaction support for jack1 were available through patch. Currently lacking an interested person to port and maintain it.  
(7) Bugfix-only mode. Don't expect new features (you can still make a pull request if you really want though).

Other differences that don't lend themselves to feature tables:

* JACK2 is more forgiving of misbehaving audio apps, whereas JACK1 is not very tolerant (zombifying the client).  (This behavior can be configured for both, but this is the default behavior.)
* Both will lock its memory to keep the kernel from moving it to the swap space on the hard disk (virtual memory).  This helps avoid critical delays (xruns).  JACK1 locks all critical data (shared memory), process data, and executable code.  JACK2 locks all critical data (shared memory) on systems that support it (Linux, macOS and Windows).
* JACK1 uses less RAM than JACK2.
* SMP support is not always as valuable as you would think.  If your applications are chained INPUT --> A --> B --> C --> OUTPUT, then it will _not_ be able to utilize multiple processors.  However, if you applications are independently generating audio to the OUTPUT, that is when "parallel" sub-graph exist in the global graph, then they can be.
* JACK2 allocates 2 threads on the client side, one for real-time computation (the one that calls the "Process" callback) and one to handle all non real-time notifications (server context changes). JACK1 only use 1 thread for real-time computation and notification handling.
* JACK2 uses a large, fixed sized buffer for MIDI; JACK1 defaults to a smaller size but allows the size to be set at run time.

