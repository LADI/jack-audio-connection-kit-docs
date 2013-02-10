This document should act as a link between the JACK design documentation and the source code. It is not supposed to be helpful if you do not look at the source code in parallel.

# Directory structure
The JACK source code tree contains the following subdirectories:
	
        * `jack`: API headers
* `jackd`: Code of the main JACK server
* `libjack`: Client library --- implementation of the API headers
	
The remaining subdirectories `tools`, `drivers`, `doc`, `man`, `config` and `example-clients` should be self-explanatory.

# Shared memory

The shared memory interface of JACK is defined by `jack/shm.h`. It is structured as follows: There is one central registry, which is a virtually contiguous chunk of shared memory and consists of one `jack_shm_header_t` followed by `MAX_SHM_ID` `jack_shm_registry_t` entries. These entries act as pointers to all of the shared memory elements of the JACK system containing the content data. The header holds an array of all the registered servers (`jack_shm_server_t`) including their names and their PIDs, the information which kind of shared memory is being used (POSIX or System V), the size of the whole registry, the size of one entry and the size of this header. One `jack_shm_registry_t` entry holds its index (If you have the pointer `jack_shm_header_t * p` to the header, you add 1 to it and then cast `p` to `jack_shm_registry_t` `*`, you then can directly navigate through the registry entries by `p[index]`), the PID of the allocator which created the shared segment, the size of the shared memory element it refers to and the ID of the shared memory element the registry element is representing. This ID is API specific: On System V it is an integer value, on POSIX it is a string, which starts with a slash. For now, we assume to have the POSIX interface.

There is another shared memory structure, which is valid only for a given address space, `jack_shm_info_t`. It has two members: an index into the registry and a pointer the shared memory segment is attached at in that address space. `jack_shmalloc()` takes such a `jack_shm_info_t` structure as a parameter together with the desired size, and it fills this structure. After the shared memory segment was created by `jack_shmalloc()`---in the case of POSIX, it is named `"/JACK-%d", d = registry->id`---it can be mapped to the local address space by `jack_attach_shm()`, which enters a valid value into `attached_at` of the `jack_shm_info_t` element.

The registry itself can be locked by a call to `jack_shm_lock_registry` which decrements the registry's semaphore; it can be unlocked by `jack_shm_unlock_registry`. These locks only refer to the information stored in the registry, but not to the other shared memory elements themselves.

# Data structures

The main data structure of the JACK engine is called `jack_engine_t` (`jack/internal.h`) alias `struct _jack_engine` (`jack/engine.h`). Its singleton instance is located in the address space of the JACK daemon process (`jackd`). It has an element `control` typed as `jack_control_t` (`jack/internal.h`). This is the part of the engine structure, which is located in shared memory (no functions, only data); its corresponding `jack_shm_info_t` member is called `control_shm`. In `jack_engine_t` the six function pointers `set_buffer_size`, `set_sample_rate`, `run_cycle`, `delay`, `transport_cycle_start` and `driver_exit` are located, which are set to `jack_set_sample_rate` (`jackd/transengine.c`), `jack_driver_buffer_size` (`jackd/engine.c`), `jack_run_cycle` (`jackd/engine.c`), `jack_engine_delay` (`jackd/engine.c`), `jack_engine_driver_exit` (`jackd/engine.c`) and `jack_transport_cycle_start` (`jackd/engine.c`) by `jack_engine_new`. Especially `jack_run_cycle` is important, as this is the _entry point_ for the driver to activate a new cycle.

In `engine->clients` the engine keeps a `JSList` (`jack/jlist.h`), a singly linked list of `jack_client_internal_t` structures (`jack/internal.h`). The type `jack_client_internal_t` is not the structure for internal clients, but the information about every client the engine keeps in its local memory. It holds the server side request file descriptor and the event file descriptor of the private communication channels with the respective client.

There is also a _truefeeds'' and a ''sortfeeds_ list. What they mean, is explained in a comment in `engine.c`: 

  Each client has a _sortfeeds_ list of clients indicating which clients it should be considered as feeding for the purposes of sorting the graph. This list differs from the clients it *actually* feeds in the following way:
  1.   Connections from a client to itself are disregarded

  2.   Connections to a driver client are disregarded

  3.   If a connection from A to B is a feedback connection (ie there was already a path from B to A when the connection was made) then instead of B appearing on A's sortfeeds list, A will appear on B's sortfeeds list.

  If client A is on client B's sortfeeds list, client A must come after client B in the execution order. The above 3 rules ensure that the sortfeeds relation is always acyclic so that all ordering constraints can actually be met. 

  Each client also has a _truefeeds_ list which is the same as sortfeeds except that feedback connections appear normally instead of reversed. This is used to detect whether the graph has become acyclic. 

The _sortfeeds'' and ''truefeeds_ lists are set during every port (dis-)connection.

The pointer to the shared memory part of the client `jack_client_control_t` is, again, called `control`; its `jack_shm_info_t` structure is called `control_shm`. In `jack_client_control_t` all the callbacks are put, notably the `JackProcessCallback` and the `JackSyncCallback`. There can also be found some information about the client ID, the client name, the clients state (`jack_client_state_t`, `jack/internal.h`), whether the client is the timebase master and whether it is a _slow sync client_ or not.

On client side each client has its own representation of itself, called `struct _jack_client` (`libjack/local.h`) alias `jack_client_t` (`jack/types.h`). It holds the request file descriptor and the event file descriptor (`event_fd` is `#defined` as `pollfd[EVENT_POLL_INDEX].fd` in `libjack/client.c`.) It has a pointer to the shared memory engine structure (`jack_control_t`) called `engine` and a pointer to its own shared memory part `jack_client_control_t`, called `control`. The associated `jack_shm_info_t` member of the shared memory structure is called `engine_shm` and `control_shm`, respectively.

# jackd and drivers, part 1

The main function can be found in `jackd/jackd.c`. After the standard GNU `getopt_long` parameter demarshalling---at least a driver has to be given (`seen_driver`), we start with loading the descriptions of all the available drivers into a the global variable `static JSList * drivers`:
	
	      drivers = jack_drivers_load ();
	
The function `jack_drivers_load` crawls the JACK drivers directory, which is given in the environment variable `JACK_DRIVER_DIR`. Whenever a driver is found, it calls `jack_drivers_get_descriptor`, which extracts the driver description from the shared object file. All drivers in JACK are specific shared object files defined by `jack_driver_t` or `jack_driver_nt_t` in `jack/driver.h`. Therefore, `jack_drivers_get_descriptor` dynamically loads the driver file via `dlopen` and calls `driver_get_descriptor()`, which every driver has to implement. We end up with a `JSList` of the descriptors of all the drivers found. The driver description type `jack_driver_desc_t` can be found in `jack/driver_interface.h`.

# IPC

There are two bidirectional channels between each external client and the server: An event socket and a request socket. On the request socket, the client can ask the server to do some work for him, for example register a port, connect a port, activate the client and so on. Via the event socket the server can notify its clients about occurring events, such as a (dis-)connected port, a buffer size or sample rate change or a graph reordering. These channels need to be bidirectional (and therefore are implemented as sockets) because the other side needs a feedback channel to answer, whether the request or the event delivery, respectively, was successful.

The _entry sockets'' on server side are created during `jack_engine_new()` (`jackd/engine.c`) by calling `make_sockets()` (`jackd/engine.c`): First the ''master server socket'' is created and its file descriptor saved in `engine->fds[0]`. Then, the socket is bound to its registration path `"%s/jack_0"` (where the string variable `s` is filled by `jack_server_dir()`) and the socket is advised to listen to connection events. This socket is used to establish the request socket between a client and the server. As we see later: Now a client can create a socket and connect it to the path `"%s/jack_0"`. When then server accepts the connection on the master socket, it receives another file descriptor: A privat request channel between the client and the server with one file descriptor on each side is established. After that, the same procedure is repeated for the ''client/server event ack socket_, which is bound to the path `"%s/jack_ack_%d"`.

The communication with the clients is handled in an extra server thread, implemented in the function `jack_server_thread()`, (`jackd/engine.c`) and started at the end of `jack_engine_new()} (	jackd/engine.c`) with `jack_client_create_thread()` (`libjack/thread.c`). The jack server thread implements an infinite server loop. Each cycle begins with setting up the `engine->pfd` array of `pollfd` structures containing the file descriptors and the events which should be polled for. The _master server socket'' `engine->fds[0]` is put to `engine->pfd[0].fd`, the ''client/server event ack server'' `engine->fds[1]` to `engine->pfd[1].fd`. The `fd` member of `engine->pfd[2]` holds `engine->cleanup_fifo[0]`: In addition to the main sockets, there is a so-called ''cleanup FIFO_,  which is actually a pipe. It is also set up during `jack_engine_new()` with `cleanup_fifo[1]` being the source and `cleanup_fifo[0]` the sink. This pipe is not used for communication, but only for synchronization and it gives the other threads of the server an opportunity to wake up the server thread (`jack_wake_server_thread`, `jackd/clientengine.c`) if some errors occurred during client checkup (`jack_check_clients`, `jackd/clientengine.c`). After these three elements, all the valid request file descriptors of the clients are added. Now we poll on `engine->pfd` and sleep, if an event or many events have occurred, the server thread gets waked up and handles the events.
	
	
	# Time and transport handling
	How the time and transport handling works, is explained in the [JACK Audio Connection Kit reference manual](http://jackaudio.org/files/docs/html/transport-design.html).
	
	# The engine and drivers, part 2
	
	The core engine of the jack daemon is situated in `jack_main` (`jackd/jackd.c`). First it creates a new engine structure via `{jack_engine_new` (`jackd/engine.c`). This function sets up all the engine data structures and the ports, creates the request and event master sockets and the cleanup FIFO. At the end it starts the `jack_server_thread` (`jackd/engine.c`), which handles all the requests delivered from the clients.
	
	Now `jack_main` calls `jack_engine_load_driver` (`jackd/engine.c`). The function loads the driver from disk via `jack_load_driver` (`jackd/engine.c`). Afterwards `jack_engine_load_driver` creates a driver client (for the sinks and sources of the driver): `jack_create_driver_client` (`jackd/clientengine.c`). It then calls `initialize` on the drivers info structure, which lets the driver return its _real driver structure_ `jack_driver_t`. `jack_use_driver` puts this driver structure into the engine (`engine->driver`) and attaches the engine to the driver. During attachment, the driver normally registers the ports for its driver client `driver->client` and activates the client.
	
	A driver can either be threaded or non-threaded: `jack_driver_t` or `jack_driver_nt_t`. Many drivers have their own driver data structure; in case of ALSA this is `alsa_driver_t`. A threaded drivers structure has to have all the elements of `jack_driver_t` at its beginning (a non-threaded driver them of `jack_driver_nt_t`) to allow casting it to `jack_driver_t` (or `jack_driver_nt_t`). The structure `jack_driver_t` in turn is a subset of `jack_driver_nt_t`, so you can cast any `jack_driver_nt_t` to `jack_driver_t`. To make programming easier, the elements of `jack_driver_t` and `jack_driver_nt_t` are not directly coded into the structures but via the macros `JACK_DRIVER_DECL` and `JACK_DRIVER_NT_DECL`, which also should be used for the own driver structures. The difference between threaded and non-threaded drivers is that threaded drivers have to bring up a thread, which executes the driver loop
	
	      while (...)
	        { driver->wait (); driver->engine->run_cycle () }
	
on their own.
Whereas a non-threaded driver normally calls `jack_driver_nt_init` (`libjack/driver.c`) during `driver_initialize`, which sets `jack_driver_nt_start` as the driver starting function `driver->start`. In `jack_driver_nt_thread` the driver loop
	
	      while (...)
	        { driver->nt_run_cycle() }
	
is implemented. Hence, the waiting has to be done inside of `nt_run_cycle()`.

In any case, the driver is responsible for initiating a new engine cycle, which is executed in the context of the driver. The function `engine->run_cycle` (set to `jack_run_cycle`, `jackd/engine.c`) is the _callback_ of the engine---or in other words: it is the entry point into the engine from the driver. The important lines of `jack_run_cycle` are
	
	      for (left = nframes; left >= b_size; left -= b_size) 
	         jack_run_one_cycle (engine, b_size, delayed_usecs)
	
The function `jack_run_one_cycle` essentially does the following: 
	
		1. driver->read (driver, nframes)
		2. jack_engine_process (engine, nframes)
		3. driver->write (driver, nframes)
	
The function `jack_engine_process` iterates over all the clients in the list `engine->clients` and, as long as the client is active and not dead, it executes `jack_process_internal` (`jackd/engine.c`) if the client is internal, and `jack_process_external` (`jackd/engine.c`) otherwise.

