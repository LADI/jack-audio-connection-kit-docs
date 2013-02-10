This document should act as a link between the JACK design documentation and the source code. It is not supposed to be helpful if you do not look at the source code in parallel.

# Shared memory

JACK2 uses the JACK1 source code for shared memory management. This code is encapsulated in an object oriented shared memory model in JACK2. The mother class for a shared memory management is `JackShmMemAble`. The classes `JackClientControl` and `JackShmMem` derive from it. `JackShmMemAble` is the base class for objects, which can be put in shared memory but do not have to be shared necessarily. `JackClientControl` is an example for such a class (the only one at the moment): Internal clients simply hold a normal instance of `JackClientControl`, while for external clients an instance of `JackShmMem` is created, as we see later in this chapter.

In `JackShmMem` the `new` and `delete` operators are overloaded, so that if you create a new object of a class inheriting from `JackShmMem`, not the normal dynamic object initialization routines are issued, but `jack_shmalloc` and `jack_attach_shm` (`common/shm.c`). It appeared that writing the `fInfo` field during the `new` operator failed with some compilers (the fields were erased at a later instant in the constructor's scope). For this reason, the overloaded `new` operator sets the info structure of the recently created shared memory segment to the global variable `gInfo` and gets it "written back" to `fInfo` during the function Init()} (common/JackShmMem.cpp}).

For client side use, there are the classes `JackShmReadWritePtr`, `JackShmReadWritePtr1` and `JackShmReadPtr`, which are to be constructed with an index and the server name as parameters. They represent a shared memory object on client side. One can get the address of the shared memory segment the object represents by its function `GetShmAddress()` or by applying the dereferencing operator `*` on it. If one wants to access a member of the shared memory segments structure, he can also use the `->` operator directly. One can even assign an integer to an object of one of these classes. This will set the integer as the new index of the object by calling the `Init()` function again.

# JACK2 thread handling

The base class for thread representation in JACK2 is located in the namespace `jack.detail` and can be found in `common/JackThread.h`. The constructor of `JackThread` asks among others for the parameter `JackRunnableInterface * runnable`.

`JackRunnableInterface` is an abstract root class for objects which can be handed over to a new thread defining the actions to be be performed by the thread. It holds only two member functions: `Init()` and `Execute()`. While `Init()` is supposed to be called only once by the thread handler, the function `Execute()` should be called over and over again in an endless server loop. Such a thread handler, which should be set as the init point of the new thread (normally by `Start()`) can look like this:
	
	void* JackPosixThread::ThreadHandler(void* arg)
	{
	    JackPosixThread* obj = (JackPosixThread*)arg;
	    JackRunnableInterface* runnable = obj->fRunnable;
	    ...
	    obj->fStatus = kIniting;
	    
	    if (!runnable->Init()) {
	        jack_error("Thread init fails: thread quits");
	        return 0;
	    }
	    
	    obj->fStatus = kRunning;
	    
	    bool res = true;
	    while (obj->fStatus == kRunning && res) {
	        res = runnable->Execute();
	    }
	    ...
	}
	
In addition to `Start()`, `Kill()`, `Stop()` and `AcquireRealTime()`, a thread must also implement the static methods `StartImp(pthread_t * thread, ...)`, `StopImp(pthread_t thread)`, `KillImp(pthread_t thread)` to be called "out of the cold". The method `StartSync()` starts the thread, but in contrast to `Start()` it sleeps until the thread is started correctly.

The specific system depended thread implementations like `JackWinThread` or `JackPosixThread` inherit from `JackThread`. Please note that `JackMachThread` is not derived from `JackThread` directly but from `JackPosixThread`. Which one of these implementations is plugged into the interface is set up in the header `$(platform)/JackPlatformPlug_os.h` via `typedef` `<implementation>` `JackThread`. This is possible because the mother class `JackThread` resides in the subnamespace `detail`.

# Lock-free shared memory access

To implement lock-free shared memory access JACK2 defines the class template
	
	      template <class T> class JackAtomicState
	        (common/JackAtomicState.h),
	
which offers lock-free access to an instance of the class `T`. For instance, the `JackGraphManager` is derived from `JackAtomicState<JackConnectionManager>`.

`JackAtomicState` has a two-element array of class `T`. One element holds the current state of the object, the other the next state. The next state can be written freely by non real-time threads, while still the real-time thread can read the status of the current state. If a non real-time thread wants to perform write access on the next state, it has to surround this quasi critical section with `WriteNextStateStart()` and `WriteNextStateStop()` commands (member functions of `JackAtomicState`). At the end of the cycle, the real-time thread tries to atomically switch the current and next state using CAS (Compare and Swap). It succeeds if no non real-time thread is in such a critical section, it fails otherwise. If it fails, one has to wait for the next cycle.

An important element of `JackAtomicState` is its member `fCounter` of the type `AtomicCounter`. The structure consists of a union called `info`. One representation of `info` is a `UInt32` called `fLongVal`, the other is a structure of two `UInt16`, called `fShortVal1` and `fShortVal2`
	
	      =================================
	      |            fLongVal           |
	      =================================
	      |   fShortVal1  |  fShortVal2   |
	      =================================
	      ^^^^^^^^^^^^^^^^ ^^^^^^^^^^^^^^^^
	           CurIndex        NextIndex
	      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
	                   Counter
	
	      CurArrayIndex(e)  = CurIndex(e)       mod 2
	      NextArrayIndex(e) = (CurIndex(e) + 1) mod 2
	
The header `JackAtomicState.h` introduces five macros
* `Counter(e)`,
* `CurIndex(e)`,
* `NextIndex(e)`,
* `CurArrayIndex(e)`,
* `NextArrayIndex(e)`
to make accessing `AtomicCounter` easier.
The semantics of `AtomicCounter` can be described as follows: If `fShortVal1` is equal to `fShortVal2` (and therefore `CurIndex == NextIndex`), this means that either no update to the next state has happened yet or that a non real-time thread is currently writing to the next state.

The only thing `TrySwitchState()` does is atomically setting `CurIndex(fCounter)` to `NextIndex(fCounter)` using CAS principles. If `CurIndex` and `NextIndex` already were equal, nothing has changed. Otherwise `CurIndex` has a new value now (that one of `NextIndex`) and `CurIndex` is equal to `NextIndex`, indicating that no change to `NextIndex` has happened yet. `WriteNextStateStart()` atomically sets `NextIndex(fCounter)` to `CurIndex(fCounter)`, causing `TrySwitchState()` to keep the current state. Additionally, it copies
	
	      fState[CurArrayIndex(fCounter)]
	
to
	
	      fState[NextArrayIndex(fCounter)] 
	
afterwards, if `NextIndex(fCounter)` and `CurIndex(fCounter)` already were equal before. `WriteNextStateStop()` increments `NextIndex(fCounter)` atomically, which indicates the next state has been changed but no thread is updating the next state at the moment so far.

# API

The headers for the client API can be found in `common/jack`. They are taken from the C implementation and have remained almost unmodified. The JACK2 API implementation is located in `common/JackAPI.cpp`. JACK2 is designated to be compiled with the `-fvisibility=hidden` GCC option and equips all the API functions with the C++ visibility attribute
	
	      (__attribute__((visibility("default"))))
	
But the export should only be done in the library itself, because the clients might not want to export the JACK1 API functions. Therefore, `common/JackAPI.cpp` does not include the API headers, but declares the API again with the visibility attribute attached.

Some of these API calls `jack_client_open`, `jack_client_open_aux`, `jack_client_close`, `jack_get_client_pid`) are different for external and internal clients. These functions are implemented in `common/JackLibAPI.cpp` and `common/JackServerAPI.cpp`, respectively. One might wonder why an internal client wants to call `jack_client_open`. An example for such a situation is a connected internal client, which wants to create another internal client.

# Client representation

The client model of JACK2 contains four important classes:
* `JackDriverClientInterface`,
* `JackExternalClient`,
* `JackInternalClient` and
* `JackLibClient`.

All client classes are based on the `JackClientInterface` (`common/JackClientInterface.h`). Each external client is represented by a `JackLibClient` object on client side and a corresponding `JackExternalClient` in the server address space. Internal clients only need one representation, `JackInternalClient`, as the server address space is the context they run in. The `JackDriverClientInterface` will be covered in the drivers section.

`JackClient`---also derived from `JackClientInterface`---is an intermediate base class of `JackLibClient` and `JackInternalClient`. It encapsulates all the code shared between them. In addition, `JackClient` inherits from `JackRunnableInterface`. Its `Execute()` function is the clients real-time Process()} loop.

The client data, which is shared between the server and external clients, is put in a variable of the type `struct` `JackClientControl`. The shared memory segment is initialized during the Open()} function of `JackExternalClient`: First, the `new` operator of `JackShmMem` is called with the size of the `struct` `JackClientControl` as parameter. The resulting object is casted to its base class `JackShmMemAble`; the pointer to it is saved temporarily in an variable of the type `JackShmMemAble` `*`, named `shared_mem`. With the _placement new operator_, a new `JackClientControl` object is created. But instead of allocating new memory on the heap, this operator
	
	      new(shared_mem)
	
stores the new object in the place given by the `shared_mem` variable. The pointer to the new object is stored in the `fClientControl` member of `JackExternalClient`.

To access the shared data, `JackLibClient` also holds an attribute called `fClientControl`, but with the type
	
	      JackShmReadWritePtr1<JackClientControl>
	
The `Open()` function of `JackLibClient` initializes this value. The class `JackInternalClient` holds `fClientControl` as a simple member variable of the type `JackClientControl`.

# Drivers

For the C++ reimplementation, a new driver interface was designed: This class `JackDriverInterface` can be found in `common/JackDriver.h`. Its important methods are `Open()`, `Attach()`, `Detach()`, `Read()`, `Write()`, `Start()`, `Stop()` and `Process()`.

Together with `JackClientInterface` `JackDriverInterface` builds the base for the class `JackDriverClientInterface` (`common/JackDriver.h`) all drivers are derived from. `JackDriverClientInterface` does not add any members, but it is only used to join the two interfaces to a single one.

The function `Read()` / `Write()` is supposed to transport the data from / to the sound card to / from the driver clients output / input ports. The method `Attach()` / `Detach()` (de)allocates the capture and playback ports for the driver. `Start()` and `Stop()` should be self-explanatory. `Process()` is the function to call `Read()` and `Write()` and to activate the engine. It should be set as the real drivers callback routine (or should be called in the function set as a callback). Since some of these functions are the same for most of the drivers, this code is put in the class `JackAudioDriver` a driver should inherit from.

JACK works with callback based sound APIs like CoreAudio, which run a thread on their own, as well as with non-threaded APIs like ALSA. The common thread code for non-threaded mode driver APIs is located in `JackThreadedDriver`. This class is derived from `JackDriverClientInterface` and `JackRunnableInterface` and is constructed with the "real" non-threaded driver object as its only argument. `JackThreadedDriver` inherits from `JackRunnableInterface`, because it is also handed over to its member `fThread` as the task to be accomplished. On the driver part of `JackThreadedDriver`, most methods are directly redirected to the same method of the non-threaded driver (named `fDriver`); except for `Start()`, which first calls `fDriver->Start()` and then starts the drivers thread. The `Init()` function of `JackThreadedDriver` sets up the thread and driver parameters, the member `Execute()` is even simpler:
	
	      bool JackThreadedDriver::Execute()
	            { return (Process() == 0); }
	

# IPC

In the JACK2 architecture, the IPC implementation is distributed amongst four different classes:
* `JackServerChannel`, 
* `JackClientChannel`,
* `JackServerNotifyChannel` and
* `JackNotifyChannel`.

All of these classes are system dependent. The specific implementation is plugged in with the `$(platform)/JackPlatformPlug_os.h` header. The semantics of these classes are exemplified with the linux implementation using sockets. 

The only instance of a `JackServerChannel`, `fChannel`, is held by the `JackServer` class. `JackServerChannel` implements the request channel endpoints on server side. The class `JackSocketServerChannel` both holds the entry point request socket `fRequestListenSocket` as well as a map from all open file descriptors to their socket representations. A `JackServerChannel` class is expected to start its own thread during `Open()`. Therefore, it makes sense for a `JackServerChannel` to inherit from `JackRunnableInterface`.

In contrast to `JackServerChannel`, a `JackClientChannel` implementation both has to take care of the notifications, which come from the server, and provide an interface for sending requests to the server. The expected `JackClientChannel` interface is defined explicitly in `JackClientChannelInterface` (`common/JackChannel.h`). These functions are called indirectly by the user via the client API (see the API section). 

If a non real-time thread of the server wants to send a notification to clients, it will either use the `JackEngine::NotifyClient()` or `JackEngine::NotifyClients()`. The difference: The former informs only one single client, the latter iterates over all available clients. Both will call `client->ClientNotify()`. If the client has the type `JackInternalClient`, the function is called directly. If the client has the type `JackExternalClient`, its `ClientNotify()` function forwards the call to its member `fChannel` with the type `JackNotifyChannel`, which in turn has to forward the call via IPC to its corresponding `JackLibClient`. When the `JackClientChannel` of the `JackLibClient` receives the event, it will finally call `ClientNotify()` of the `JackLibClient`.

If the real-time thread of the server, the driver, wants to send a notification to clients, it has to use the function Notify()} of the member `fChannel` in the engine with the type `JackSeverNotifyChannel`. This function may fail and must be non-blocking, as it is used by the real-time thread. The implementation for Linux, `JackSocketServerNotifyChannel`, sends the notification as a request to the non real-time server thread, which delivers the notification to the clients / the client.