# FAQ: what are the differences between JACK 1 and JACK2

**The short answer:** Jack 1 and Jack 2 are equivalent implementations of the same protocol. Jack 2 was originally written to add multi-processor support and to rewrite the internals using C++.  Programs compiled against Jack 1 will work with Jack 2 without recompile (and vice versa).  Jack 2 was originally planned to replace Jack 1, but this is no longer so and they are considered equivalent implementations.  JACK is a C API, and so the C++ rewrite in Jack 2 only matters to the people who program/maintain jack.

**So, why would I choose one or the other?**  For most folks, it doesn't matter.  For others, there is typically one or two features that cause them to prefer one or the other.  See below for a detailed listing of the difference.

**Why isn't Jack 2 going to replace Jack 1?  What happened?**  Simply put, the Jack 1 developers couldn't bring themselves to migrate to Jack 2.  There is no animosity between the two developer groups (they do indeed work together nicely).  Many of the Jack 1 developers disagree with the implementation details and class hierarchy of Jack 2, to the point where they would prefer working on the Jack 1 code.  Note that this rejection is *not* because it was done in C++ (as the Jack 1 devs are accomplished C++ devs).

**Are we going to have two JACKs forever?**  No.  However nobody forsees a sane way to choose between Jack 1 and Jack 2.  They're both very good and have loyal followings.  However, when the time comes to work on Jack 3 (after Jack 1 is deemed "complete") nobody expects this situation to happen again. As of 2018, both versions are maintained by falkTX. His current plan is to implement all Jack 1 features into Jack 2 and then slowly phase out Jack 1 into "bugfix-only" mode (see the JACK2 v1.9.12 release notes for more detailed information).

**So what about the version numbers ?** Jack 1 _approaches_ version 1.0. At the time of writing it is at version 0.121.3. Jack 2 approaches version 2.0 - at the time of writing it is at version 1.9.10. run `jackd --version` to find out the version that you have installed.


# Detailed Differences

Here is a feature listing/comparison of the different jack implementations.  Note that this list does not include tschack, Torben Hohn's experimental branch of Jack 1.

| **Feature**                                                |  **Jack 1**  | **Jack 2** |
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
| Supports OS X                                              |  Yes         |  Yes(4)    |
| Supports Windows                                           |  No          |  Yes(5)    |
| Supports Solaris/OpenSolaris                               |  Yes?        |  Yes       |
| Has builtin support for >1 soundcard on Linux              |  Yes         |  No        |
| Supports metadata API                                      |  Yes         |  Yes       |
| Has builtin integration with Linux MIDI                    |  Yes         |  No        |
| Has builtin integration with CoreMIDI (OS X)               |  No          |  Yes       |

(1) Jack2 can run in 2 different modes : "asynchronous" when the server does not wait for graph end for a given cycle but just write the outputs computed at the previous cycle. In this case an extra period of latency is added.  Jack 2 can also be run in "synchronous" mode, when the server waits for the graph activation end in a given cycle, in which case it works like Jack 1.  
(2) DBUS support helps in integrating with [PulseAudio](http://www.pulseaudio.org/) and [LADI](http://ladish.org/wiki/ladi).  
(3) It's not that netjack2 is exclusively for Jack2, it's just that nobody has ported the protocol to Jack1.  
(4) Integrated with [JackOSX](http://www.jackosx.com/) which allows CoreAudio clients become Jack clients (thanks to the JackRouter CoreAudio/Jack virtual audio device)  
(5) ASIO clients can become Jack clients (thanks to the JackRouter ASIO/Jack bridge)  
(6) DBUS and PulseAudio interaction support for jack1 is available through patch. Currently distributed as a modified jack1 [tarball](http://nedko.arnaudov.name/soft/jack/dbus/).  
(7) Bugfix-only mode. Don't expect new features (you can still make a pull request if you really want though).

Other differences that don't lend themselves to feature tables:

* Jack 2 is more forgiving of misbehaving audio apps, whereas Jack 1 is not very tolerant (zombifying the client).  (This behavior can be configured for both, but this is the default behavior.)
* Both will lock its memory to keep the kernel from moving it to the swap space on the hard disk (virtual memory).  This helps avoid critical delays (xruns).  Jack 1 locks all critical data (shared memory), process data, and executable code.  Jack 2 locks all critical data (shared memory) on systems that support it (Linux, OS X and Windows).
* Jack 1 uses less RAM than Jack 2.
* SMP support is not always as valuable as you would think.  If your applications are chained INPUT --> A --> B --> C --> OUTPUT, then it will _not_ be able to utilize multiple processors.  However, if you applications are independently generating audio to the OUTPUT, that is when "parallel" sub-graph exist in the global graph, then they can be.
* Jack 2 allocates 2 threads on the client side, one for real-time computation (the one that calls the "Process" callback) and one to handle all non real-time notifications (server context changes). Jack 1 only use 1 thread for real-time computation and notification handling.
* Jack 2 uses a large, fixed sized buffer for MIDI; Jack 1 defaults to a smaller size but allows the size to be set at run time.
