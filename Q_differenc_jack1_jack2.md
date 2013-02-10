_The short answer:_ Jack 1 and Jack 2 are equivalent implementations of the same protocol.  Jack 2 was originally written to add multi-processor support and to rewrite the internals using C++.  Programs compiled against Jack 1 will work with Jack 2 without recompile (and vice versa).  Jack 2 was originally planned to replace Jack 1, but this is no longer so and they are considered equivalent implementations.  JACK is a C API, and so the C++ rewrite in Jack 2 only matters to the people who program/maintain jack.

_So, why would I choose one or the other?_  For most folks, it doesn't matter.  For others, there is typically one or two features that cause them to prefer one or the other.  See below for a detailed listing of the difference.

_Why isn't Jack 2 going to replace Jack 1?  What happened?_  Simply put, the Jack 1 developers couldn't bring themselves to migrate to Jack 2.  There is no animosity between the two developer groups (they do indeed work together nicely).  Many of the Jack 1 developers disagree with the implementation details and class hierarchy of Jack 2, to the point where they would prefer working on the Jack 1 code.  Note that this rejection is *not* because it was done in C++ (as the Jack 1 devs are accomplished C++ devs).

_Are we going to have two JACKs forever?_  No.  However nobody forsees a sane way to choose between Jack 1 and Jack 2.  They're both very good and have loyal followings.  However, when the time comes to work on Jack 3 (after Jack 1 is deemed "complete") nobody expects this situation to happen again.

# Detailed Differences

Here is a feature listing/comparison of the different jack implementations.  Note that this list does not include tschack, Torben Hohn's experimental branch of Jack 1.

| *Feature_'                                                 | '''Jack 1''' | '_Jack 2*  |
|------------------------------------------------------------|:------------:|:----------:|
| Implements the JACK C API                                  |  Yes         |  Yes       |
| Supports multiple processors (SMP)                         |  No          |  Yes(1)    |
| Allows apps to connect/disconnect without disrupting audio |  No          |  Yes       |
| Has (optional) DBUS support(2)                             |  No(6)       |  Yes       |
| Can be used with [[WalkThrough_Dev_NetOne netjack1]]       |  Yes         |  Yes       |
| Can be used with [[WalkThrough_User_NetJack2 netjack2]]    |  No          |  Yes       |
| Is under active development                                |  Yes         |  Yes       |
| Supports Linux/*nix/POSIX OS's                             |  Yes         |  Yes       |
| Supports OS X                                              |  Yes         |  Yes(4)    |
| Supports Windows                                           |  No          |  Yes(5)    |
| Supports Solaris/OpenSolaris                               |  Yes?        |  Yes       |

(1) Jack2 can run in 2 different modes : "asynchronous" when the server does not wait for graph end for a given cycle but just write the outputs computed at the previous cycle. In this case an extra period of latency is added.  Jack 2 can also be run in "synchronous" mode, when the server waits for the graph activation end in a given cycle, in which case it works like Jack 1.[[BR]]
(2) DBUS support helps in integrating with [PulseAudio](http://www.pulseaudio.org/) and [LADI](http://ladish.org/wiki/ladi) .[[BR]]
(3) It's not that netjack2 is exclusively for Jack2, it's just that nobody has ported the protocol to Jack1.[[BR]]
(4) Integrated with [JackOSX](http://www.jackosx.com/) which allows CoreAudio clients become Jack clients (thanks to the JackRouter CoreAudio/Jack virtual audio device)[[BR]]
(5) ASIO clients can become Jack clients (thanks to the JackRouter ASIO/Jack bridge)[[BR]]
(6) DBUS support for jack1 is available through patch. Currently distributed as a modified jack1 [tarball](http://nedko.arnaudov.name/soft/jack/dbus/).

Other differences that don't lend themselves to feature tables:

* Jack 2 is more forgiving of misbehaving audio apps, whereas Jack 1 is not very tolerant (zombifying the client).  (This behavior can be configured for both, but this is the default behavior.)
* Both will lock its memory to keep the kernel from moving it to the swap space on the hard disk (virtual memory).  This helps avoid critical delays (xruns).  Jack 1 locks all critical data (shared memory), process data, and executable code.  Jack 2 locks all critical data (shared memory) on systems that support it (Linux, OS X and Windows).
* Jack 1 uses less RAM than Jack 2.
* SMP support is not always as valuable as you would think.  If your applications are chained INPUT --> A --> B --> C --> OUTPUT, then it will _not_ be able to utilize multiple processors.  However, if you applications are independently generating audio to the OUTPUT, that is when "parallel" sub-graph exist in the global graph, then they can be.
* Jack 2 allocates 2 threads on the client side, one for real-time computation (the one that calls the "Process" callback) and one to handle all non real-time notifications (server context changes). Jack 1 only use 1 thread for real-time computation and notification handling.
* Jack 2 has a master/slave back-end model.  This design allows you to, for instance,  load MIDI backends along with a master backend which can be audio or network: like "jackd -X coremidi -d coreaudio"  jackd -X coremidi -d net"  on OSX, or "jackd -X winmme -d portaudio"  , "jackd -X winmme -d net" on Windows.
