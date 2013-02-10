This pages summarizes "changes" in JACK client API between version 1 and 2

"not public" means that symbol is available in library but not declared in public header

# Symbols present in jack1 but missing in jack2
* `B _jack_get_microseconds                `- not public
* `D jack_builtin_NULL_functions           `- not public
* `D jack_builtin_audio_functions          `- not public
* `D jack_builtin_midi_functions           `- not public
* `D jack_builtin_port_types               `- not public
* `D jack_tmpdir                           `- not public
* `T cleanup_mlock                         `- not public
* `T default_jack_error_callback           `- not public
* `T default_jack_info_callback            `- not public
* `T jack_attach_port_segment              `- not public
* `T jack_attach_shm                       `- not public
* `T jack_call_sync_client                 `- not public
* `T jack_call_timebase_master             `- not public
* `T jack_cleanup_shm                      `- not public
* `T jack_client_alloc                     `- not public
* `T jack_client_alloc_internal            `- not public
* `T jack_client_deliver_request           `- not public
* `T jack_client_handle_port_connection    `- not public
* `T jack_client_invalidate_port_buffers   `- not public
* `T jack_clock_source_name                `- not public
* `T jack_default_server_name              `- not public
* `T jack_destroy_shm                      `- not public
* `T jack_dump_timestamps                  `- public in <jack/timestamps.h>
* `T jack_generate_unique_id               `- not public
* `T jack_get_free_shm_info                `- not public
* `T jack_get_mhz                          `- not public
* `T jack_get_microseconds_from_cycles     `- not public
* `T jack_get_microseconds_from_system     `- not public
* `T jack_get_port_functions               `- not public
* `T jack_get_process_done_fd              `- not public
* `T jack_hpet_init                        `- not public
* `T jack_init_time                        `- not public
* `T jack_init_timestamps                  `- public in <jack/timestamps.h>
* `T jack_initialize_shm                   `- not public
* `T jack_messagebuffer_add                `- not public
* `T jack_messagebuffer_exit               `- not public
* `T jack_messagebuffer_init               `- not public
* `T jack_pool_alloc                       `- not public
* `T jack_pool_release                     `- not public
* `T jack_port_by_id_int                   `- not public
* `T jack_port_by_name_int                 `- not public
* `T jack_port_name_equals                 `- not public
* `T jack_port_new                         `- not public
* `T jack_register_server                  `- not public
* `T jack_release_shm                      `- not public
* `T jack_release_shm_info                 `- not public
* `T jack_reset_timestamps                 `- public in <jack/timestamps.h>
* `T jack_resize_shm                       `- not public
* `T jack_server_dir                       `- not public
* `T jack_set_clock_source                 `- not public
* `T jack_shmalloc                         `- not public
* `T jack_start_freewheel                  `- not public
* `T jack_stop_freewheel                   `- not public
* `T jack_timestamp                        `- public in <jack/timestamps.h>
* `T jack_transport_copy_position          `- not public
* `T jack_unregister_server                `- not public
* `T jack_user_dir                         `- not public
* `T silent_jack_error_callback            `- not public
* `T start_server                          `- not public

# Symbols present in jack2 but missing in jack1
* `Lot of mangled C++ names                `- not public
* `T jack_client_kill_thread               `- new API function, appeared in r2548
* `T jack_client_stop_thread               `- new API function, appeared in r2548
* `T jack_get_client_pid                   `- new API function, appeared in r2299
* `T jack_get_version                      `- new API function, appered in trunk in r2339, appeared in control branch in r2060
* `T jack_get_version_string               `- new API function, appered in trunk in r2339, appeared in control branch in r2060
* `T jack_log                              `- not public
* `T jack_port_type_id                     `- new API function, appered in trunk in r2339, appeared in control branch in r2051
* `T jack_tls_allocate_key(unsigned int*)  `- not public
* `T jack_tls_free_key(unsigned int)       `- not public
* `T jack_tls_get(unsigned int)            `- not public
* `T jack_tls_set(unsigned int, void*)     `- not public
* `T set_threaded_log_function             `- not public

nm -CDg 1/usr/lib/libjack.so | grep -v ' U '|sed 's/^[0-9a-f]* //'|sort> jack1.syms
	nm -CDg 2/usr/lib/libjack.so | grep -v ' U '|sed 's/^[0-9a-f]* //'|sort> jack2.syms
	diff -u jack1.syms jack2.syms > jack12.diff
	grep '^-[A-Z]' jack12.diff | sed 's/^-//'
	grep '^+[A-Z]' jack12.diff | sed 's/^+//'|grep -v ::
	for i in `grep '^+[A-Z]' jack12.diff | grep -v :: | sed 's/^+. \([^ (]*\).*/\1/'` ; do echo "searching for $i ..." ; find 2 -type f -exec grep -Hn $i {} \; ; done|grep -v 'Binary file'
	for i in `grep '^-[A-Z]' jack12.diff  | sed 's/^-. //'` ; do echo "searching for $i ..." ; find 1 -type f -exec grep -Hn $i {} \; ; done|grep -v 'Binary file'
	

