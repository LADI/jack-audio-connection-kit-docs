# Difference between Jack1 and Jack2 headers
See jack-devel email list, thread "how you can love JACK back" June, 2012. [http://old.nabble.com/how-you-can-love-JACK-back-td33958251.html]

Complete diff: [https://github.com/x42/headers/compare/cmp_jack1...cmp_jack2]

{{{
                ------------------------------ JACK 1 ------------------------------               |                ------------------------------ JACK 2 ------------------------------

                                                                                                   >    /** Driver types */
                                                                                                   >    typedef enum
                                                                                                   >    {
                                                                                                   >        JackMaster = 1,         /**< @brief master driver */
                                                                                                   >        JackSlave               /**< @brief slave driver */
                                                                                                   >    } jackctl_driver_type_t;
                                                                                                   >


                                                                                                   >    /** opaque type for sigmask object */
                                                                                                   >    typedef struct jackctl_sigmask jackctl_sigmask_t;
                                                                                                   >



/**                                                                                                     /**
* Call this function to setup process signal handling. As a general                                     * Call this function to setup process signal handling. As a general
* rule, it is required for proper operation for the server object.                                      * rule, it is required for proper operation for the server object.
*                                                                                                       *
* @param flags signals setup flags, use 0 for none. Currently no                                        * @param flags signals setup flags, use 0 for none. Currently no
* flags are defined                                                                                     * flags are defined
*                                                                                                       *
* @return the configurated signal set.                                                                  * @return the configurated signal set.
*/                                                                                                      */
sigset_t                                                                                           |    jackctl_sigmask_t *
jackctl_setup_signals(                                                                                  jackctl_setup_signals(
    unsigned int flags);                                                                                    unsigned int flags);

/**                                                                                                     /**
* Call this function to wait on a signal set.                                                           * Call this function to wait on a signal set.
*                                                                                                       *
* @param signals signals set to wait on                                                                 * @param signals signals set to wait on
*/                                                                                                      */
void                                                                                                    void
jackctl_wait_signals(                                                                                   jackctl_wait_signals(
    sigset_t signals);                                                                             |        jackctl_sigmask_t * signals);


/**                                                                                                     /**
* Call this function to start JACK server                                                         |     * Call this function to open JACK server
*                                                                                                       *
* @param server server object handle                                                                    * @param server server object handle
* @param driver driver to use                                                                           * @param driver driver to use
*                                                                                                       *
* @return success status: true - success, false - fail                                                  * @return success status: true - success, false - fail
*/                                                                                                      */
bool                                                                                                    bool
jackctl_server_start(                                                                              |    jackctl_server_open(
    jackctl_server_t * server,                                                                              jackctl_server_t * server,
    jackctl_driver_t * driver);                                                                             jackctl_driver_t * driver);

/**                                                                                                     /**
                                                                                                   >     * Call this function to start JACK server
                                                                                                   >     *
                                                                                                   >     * @param server server object handle
                                                                                                   >     *
                                                                                                   >     * @return success status: true - success, false - fail
                                                                                                   >     */
                                                                                                   >    bool
                                                                                                   >    jackctl_server_start(
                                                                                                   >        jackctl_server_t * server);
                                                                                                   >
                                                                                                   >    /**
* Call this function to stop JACK server                                                                * Call this function to stop JACK server
*                                                                                                       *
* @param server server object handle                                                                    * @param server server object handle
*                                                                                                       *
* @return success status: true - success, false - fail                                                  * @return success status: true - success, false - fail
*/                                                                                                      */
bool                                                                                                    bool
jackctl_server_stop(                                                                                    jackctl_server_stop(
        jackctl_server_t * server);                                                                             jackctl_server_t * server);

                                                                                                   >    /**
                                                                                                   >     * Call this function to close JACK server
                                                                                                   >     *
                                                                                                   >     * @param server server object handle
                                                                                                   >     *
                                                                                                   >     * @return success status: true - success, false - fail
                                                                                                   >     */
                                                                                                   >    bool
                                                                                                   >    jackctl_server_close(
                                                                                                   >            jackctl_server_t * server);
                                                                                                   >


                                                                                                   >    /**
                                                                                                   >     * Call this function to get type of driver.
                                                                                                   >     *
                                                                                                   >     * @param driver driver object handle to get name of
                                                                                                   >     *
                                                                                                   >     * @return driver type. Must not be modified. Always same for same
                                                                                                   >     * driver object.
                                                                                                   >     */
                                                                                                   >    jackctl_driver_type_t
                                                                                                   >    jackctl_driver_get_type(
                                                                                                   >            jackctl_driver_t * driver);
                                                                                                   >

                                                                                                   >
                                                                                                   >    /**
                                                                                                   >     * Call this function to parse parameters for a driver.
                                                                                                   >     *
                                                                                                   >     * @param driver driver object handle
                                                                                                   >     * @param argc parameter list len
                                                                                                   >     * @param argv parameter list, as an array of char*
                                                                                                   >     *
                                                                                                   >     * @return success status: true - success, false - fail
                                                                                                   >     */
                                                                                                   >    int
                                                                                                   >    jackctl_driver_params_parse(
                                                                                                   >        jackctl_driver_t * driver,
                                                                                                   >        int argc,
                                                                                                   >        char* argv[]);




                                                                                                   >    /**
                                                                                                   >     * Call this function to get version of the JACK, in form of several numbers
                                                                                                   >     *
                                                                                                   >     * @param major_ptr pointer to variable receiving major version of JACK.
                                                                                                   >     *
                                                                                                   >     * @param minor_ptr pointer to variable receiving minor version of JACK.
                                                                                                   >     *
                                                                                                   >     * @param major_ptr pointer to variable receiving micro version of JACK.
                                                                                                   >     *
                                                                                                   >     * @param major_ptr pointer to variable receiving protocol version of JACK.
                                                                                                   >     *
                                                                                                   >     */
                                                                                                   >    void
                                                                                                   >    jack_get_version(
                                                                                                   >            int *major_ptr,
                                                                                                   >            int *minor_ptr,
                                                                                                   >            int *micro_ptr,
                                                                                                   >            int *proto_ptr) JACK_OPTIONAL_WEAK_EXPORT;
                                                                                                   >
                                                                                                   >    /**
                                                                                                   >     * Call this function to get version of the JACK, in form of a string
                                                                                                   >     *
                                                                                                   >     * @return Human readable string describing JACK version being used.
                                                                                                   >     *
                                                                                                   >     */
                                                                                                   >    const char *
                                                                                                   >    jack_get_version_string() JACK_OPTIONAL_WEAK_EXPORT;
                                                                                                   >
                                                                                                   >    /**
 

                                                                                                        /**
                                                                                                   >     * @return pid of client. If not available, 0 will be returned.
                                                                                                   >     */
                                                                                                   >    int jack_get_client_pid (const char *name) JACK_OPTIONAL_WEAK_EXPORT;


                                                                                                   >
                                                                                                   >     /**
                                                                                                   >     * Tell the JACK server to call @a rename_callback whenever a
                                                                                                   >     * port is renamed, passing @a arg as a parameter.
                                                                                                   >     *
                                                                                                   >     * All "notification events" are received in a seperated non RT thread,
                                                                                                   >     * the code in the supplied function does not need to be
                                                                                                   >     * suitable for real-time execution.
                                                                                                   >     *
                                                                                                   >     * NOTE: this function cannot be called while the client is activated
                                                                                                   >     * (after jack_activate has been called.)
                                                                                                   >     *
                                                                                                   >     * @return 0 on success, otherwise a non-zero error code
                                                                                                   >     */
                                                                                                   >    int jack_set_port_rename_callback (jack_client_t *client,
                                                                                                   >                                       JackPortRenameCallback
                                                                                                   >                                       rename_callback, void *arg) JACK_OPTIONAL_WEAK_EXPORT;
                                                                                                   >



                                                                                                   >    /**
                                                                                                   >     * @return the @a port type id.
                                                                                                   >     */
                                                                                                   >    jack_port_type_id_t jack_port_type_id (const jack_port_t *port) JACK_OPTIONAL_WEAK_EXPORT;
                                                                                                   >




static __inline__                                                                                       static __inline__
JSList*                                                                                                 JSList*
jack_slist_alloc (void)                                                                                 jack_slist_alloc (void)
{                                                                                                       {
        JSList *new_list;                                                                                       JSList *new_list;

        new_list (JSList*)malloc(sizeof(JSList));
        new_list->data = NULL;                                                                     |            if (new_list)
        new_list->next = NULL;                                                                     |            {
                                                                                                   >                    new_list->data = NULL;
                                                                                                   >                    new_list->next = NULL;
                                                                                                   >            }

        return new_list;                                                                                        return new_list;
}                                                                                                       }

static __inline__                                                                                       static __inline__
JSList*                                                                                                 JSList*
jack_slist_prepend (JSList* list, void* data)                                                           jack_slist_prepend (JSList* list, void* data)
{                                                                                                       {
        JSList *new_list;                                                                                       JSList *new_list;

        new_list (JSList*)malloc(sizeof(JSList));
        new_list->data = data;                                                                     |            if (new_list) {
        new_list->next data;
                                                                                                   >                    new_list->next = list;
                                                                                                   >            }

        return new_list;                                                                                        return new_list;
}                                                                                                       }






/** Get number of events in a port buffer.                                                              /** Get number of events in a port buffer.
*                                                                                                       *
* @param port_buffer Port buffer from which to retrieve event.                                          * @param port_buffer Port buffer from which to retrieve event.
* @return number of events inside @a port_buffer                                                        * @return number of events inside @a port_buffer
*/                                                                                                      */
jack_nframes_t                                                                                     |    uint32_t
jack_midi_get_event_count(void* port_buffer) JACK_OPTIONAL_WEAK_EXPORT;                                 jack_midi_get_event_count(void* port_buffer) JACK_OPTIONAL_WEAK_EXPORT;




#ifndef POST_PACKED_STRUCTURE                                                                      <
#ifdef __GNUC__                                                                                    <
/* POST_PACKED_STRUCTURE needs to be a macro which                                                 <
   expands into a compiler directive. The directive must                                           <
   tell the compiler to arrange the preceding structure                                            <
   declaration so that it is packed on byte-boundaries rather                                      <
   than use the natural alignment of the processor and/or                                          <
   compiler.                                                                                       <
*/                                                                                                 <
#define POST_PACKED_STRUCTURE __attribute__((__packed__))                                          <
#else                                                                                              <
/* Add other things here for non-gcc platforms */                                                  <
#endif                                                                                             <
#endif                                                                                             <
                                                                                                   <





                                                                                                   >    /**
                                                                                                   >     * Reset the internal "available" size, and read and write pointers, making an empty buffer.
                                                                                                   >     *
                                                                                                   >     * This is not thread safe.
                                                                                                   >     *
                                                                                                   >     * @param rb a pointer to the ringbuffer structure.
                                                                                                   >     * @param sz the new size, that must be less than allocated size.
                                                                                                   >     */
                                                                                                   >    void jack_ringbuffer_reset_size (jack_ringbuffer_t * rb, size_t sz);
                                                                                                   >


                                                                                                   >    /*
                                                                                                   >    Copyright (C) 2004-2012 Grame
                                                                                                   >
                                                                                                   >    This program is free software; you can redistribute it and/or modify
                                                                                                   >    it under the terms of the GNU Lesser General Public License as published by
                                                                                                   >    the Free Software Foundation; either version 2.1 of the License, or
                                                                                                   >    (at your option) any later version.
                                                                                                   >
                                                                                                   >    This program is distributed in the hope that it will be useful,
                                                                                                   >    but WITHOUT ANY WARRANTY; without even the implied warranty of
                                                                                                   >    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
                                                                                                   >    GNU General Public License for more details.
                                                                                                   >
                                                                                                   >    You should have received a copy of the GNU General Public License
                                                                                                   >    along with this program; if not, write to the Free Software
                                                                                                   >    Foundation, Inc., 675 Mass Ave, Cambridge, MA 02139, USA.
                                                                                                   >
                                                                                                   >    */
                                                                                                   >
                                                                                                   >    #ifndef __jack_systemdeps_h__
                                                                                                   >    #define __jack_systemdeps_h__
                                                                                                   >
                                                                                                   >    #ifndef POST_PACKED_STRUCTURE
                                                                                                   >
                                                                                                   >        #ifdef __GNUC__
                                                                                                   >            /* POST_PACKED_STRUCTURE needs to be a macro which
                                                                                                   >               expands into a compiler directive. The directive must
                                                                                                   >               tell the compiler to arrange the preceding structure
                                                                                                   >               declaration so that it is packed on byte-boundaries rather 
                                                                                                   >               than use the natural alignment of the processor and/or
                                                                                                   >               compiler.
                                                                                                   >            */
                                                                                                   >
                                                                                                   >            #define PRE_PACKED_STRUCTURE
                                                                                                   >            #define POST_PACKED_STRUCTURE __attribute__((__packed__))
                                                                                                   >
                                                                                                   >        #else
                                                                                                   >        
                                                                                                   >            #ifdef _MSC_VER
                                                                                                   >                #define PRE_PACKED_STRUCTURE1 __pragma(pack(push,1))
                                                                                                   >                #define PRE_PACKED_STRUCTURE    PRE_PACKED_STRUCTURE1
                                                                                                   >                /* PRE_PACKED_STRUCTURE needs to be a macro which
                                                                                                   >                expands into a compiler directive. The directive must
                                                                                                   >                tell the compiler to arrange the following structure
                                                                                                   >                declaration so that it is packed on byte-boundaries rather
                                                                                                   >                than use the natural alignment of the processor and/or
                                                                                                   >                compiler.
                                                                                                   >                */
                                                                                                   >                #define POST_PACKED_STRUCTURE ;__pragma(pack(pop))
                                                                                                   >                /* and POST_PACKED_STRUCTURE needs to be a macro which
                                                                                                   >                restores the packing to its previous setting */
                                                                                                   >            #else
                                                                                                   >                #define PRE_PACKED_STRUCTURE
                                                                                                   >                #define POST_PACKED_STRUCTURE
                                                                                                   >            #endif
                                                                                                   >
                                                                                                   >        #endif
                                                                                                   >
                                                                                                   >    #endif
                                                                                                   >
                                                                                                   >    #if defined(WIN32) && !defined(__CYGWIN__) && !defined(GNU_WIN32)
                                                                                                   >
                                                                                                   >        #include <windows.h>
                                                                                                   >
                                                                                                   >        #ifdef _MSC_VER     /* Microsoft compiler */
                                                                                                   >            #define __inline__ inline
                                                                                                   >            #if (!defined(int8_t) && !defined(_STDINT_H))
                                                                                                   >                #define __int8_t_defined
                                                                                                   >                typedef char int8_t;
                                                                                                   >                typedef unsigned char uint8_t;
                                                                                                   >                typedef short int16_t;
                                                                                                   >                typedef unsigned short uint16_t;
                                                                                                   >                typedef long int32_t;
                                                                                                   >                typedef unsigned long uint32_t;
                                                                                                   >                typedef LONGLONG int64_t;
                                                                                                   >                typedef ULONGLONG uint64_t;
                                                                                                   >            #endif
                                                                                                   >        #elif __MINGW32__   /* MINGW */
                                                                                                   >            #include <stdint.h>
                                                                                                   >            #include <sys/types.h>
                                                                                                   >        #else               /* other compilers ...*/
                                                                                                   >            #include <inttypes.h>
                                                                                                   >            #include <pthread.h>
                                                                                                   >            #include <sys/types.h>
                                                                                                   >        #endif
                                                                                                   >
                                                                                                   >        #if !defined(_PTHREAD_H) && !defined(PTHREAD_WIN32)
                                                                                                   >            /**
                                                                                                   >             *  to make jack API independent of different thread implementations,
                                                                                                   >             *  we define jack_native_thread_t to HANDLE here.
                                                                                                   >             */
                                                                                                   >            typedef HANDLE jack_native_thread_t;
                                                                                                   >        #else
                                                                                                   >            #ifdef PTHREAD_WIN32            // Added by JE - 10-10-2011
                                                                                                   >                #include <ptw32/pthread.h>  // Makes sure we #include the ptw32 version !
                                                                                                   >            #endif
                                                                                                   >            /**
                                                                                                   >             *  to make jack API independent of different thread implementations,
                                                                                                   >             *  we define jack_native_thread_t to pthread_t here.
                                                                                                   >             */
                                                                                                   >            typedef pthread_t jack_native_thread_t;
                                                                                                   >        #endif
                                                                                                   >
                                                                                                   >    #endif // WIN32 && !__CYGWIN__ && !GNU_WIN32 */
                                                                                                   >
                                                                                                   >    #if defined(__APPLE__) || defined(__linux__) || defined(__sun__) || defined(sun) || defined(__un
                                                                                                   >
                                                                                                   >        #if defined(__CYGWIN__) || defined(GNU_WIN32)
                                                                                                   >            #include <stdint.h>
                                                                                                   >        #endif
                                                                                                   >            #include <inttypes.h>
                                                                                                   >            #include <pthread.h>
                                                                                                   >            #include <sys/types.h>
                                                                                                   >
                                                                                                   >            /**
                                                                                                   >             *  to make jack API independent of different thread implementations,
                                                                                                   >             *  we define jack_native_thread_t to pthread_t here.
                                                                                                   >             */
                                                                                                   >            typedef pthread_t jack_native_thread_t;
                                                                                                   >
                                                                                                   >        #endif /* __APPLE__ || __linux__ || __sun__ || sun */
                                                                                                   >
                                                                                                   >    #endif






typedef int (*jack_thread_creator_t)(jack_native_thread_t*,                                        |    /**
                                                                                                   >     * Stop the thread, waiting for the thread handler to terminate.
                                                                                                   >     *
                                                                                                   >     * @param thread POSIX thread ID.
                                                                                                   >     *
                                                                                                   >     * @returns 0, if successful; otherwise an error number.
                                                                                                   >     */
                                                                                                   >    int jack_client_stop_thread(jack_client_t* client, jack_native_thread_t thread) JACK_OPTIONAL_WE
                                                                                                   >
                                                                                                   >    /**
                                                                                                   >     * Kill the thread.
                                                                                                   >     *
                                                                                                   >     * @param thread POSIX thread ID.
                                                                                                   >     *
                                                                                                   >     * @returns 0, if successful; otherwise an error number.
                                                                                                   >     */
                                                                                                   >    int jack_client_kill_thread(jack_client_t* client, jack_native_thread_t thread) JACK_OPTIONAL_WE
                                                                                                   >
                                                                                                   >    #ifndef WIN32
                                                                                                   >
                                                                                                   >    typedef int (*jack_thread_creator_t)(pthread_t*,
                                     const pthread_attr_t*,                                                                                  const pthread_attr_t*,
                                     void* (*function)(void*),                                                                               void* (*function)(void*),
                                     void* arg) JACK_OPTIONAL_WEAK_EXPORT;                         |                                         void* arg);
/**                                                                                                     /**
* This function can be used in very very specialized cases                                              * This function can be used in very very specialized cases
* where it is necessary that client threads created by JACK                                             * where it is necessary that client threads created by JACK
* are created by something other than pthread_create(). After                                           * are created by something other than pthread_create(). After
* it is used, any threads that JACK needs for the client will                                           * it is used, any threads that JACK needs for the client will
* will be created by calling the function passed to this                                                * will be created by calling the function passed to this
* function.                                                                                             * function.
*                                                                                                       *
* No normal application/client should consider calling this.                                            * No normal application/client should consider calling this.
* The specific case for which it was created involves running                                           * The specific case for which it was created involves running
* win32/x86 plugins under Wine on Linux, where it is necessary                                          * win32/x86 plugins under Wine on Linux, where it is necessary
* that all threads that might call win32 functions are known                                            * that all threads that might call win32 functions are known
* to Wine.                                                                                              * to Wine.
*                                                                                                       *
                                                                                                   >     * Set it to NULL to restore thread creation function.
                                                                                                   >     *
* @param creator a function that creates a new thread                                                   * @param creator a function that creates a new thread
*                                                                                                       *
*/                                                                                                      */
void jack_set_thread_creator (jack_thread_creator_t creator) JACK_OPTIONAL_WEAK_EXPORT;                 void jack_set_thread_creator (jack_thread_creator_t creator) JACK_OPTIONAL_WEAK_EXPORT;





/**                                                                                                     /**
* Gets the current transport info structure (deprecated).                                               * Gets the current transport info structure (deprecated).
*                                                                                                       *
* @param client the JACK client structure.                                                              * @param client the JACK client structure.
* @param tinfo current transport info structure.  The "valid" field                                     * @param tinfo current transport info structure.  The "valid" field
* describes which fields contain valid data.                                                            * describes which fields contain valid data.
*                                                                                                       *
* @deprecated This is for compatibility with the earlier transport                                      * @deprecated This is for compatibility with the earlier transport
* interface.  Use jack_transport_query(), instead.                                                      * interface.  Use jack_transport_query(), instead.
*                                                                                                       *
* @pre Must be called from the process thread.                                                          * @pre Must be called from the process thread.
*/                                                                                                      */
void jack_get_transport_info (jack_client_t *client,                                                    void jack_get_transport_info (jack_client_t *client,
                              jack_transport_info_t *tinfo) JACK_OPTIONAL_WEAK_DEPRECATED_EXPORT   |                                  jack_transport_info_t *tinfo) JACK_OPTIONAL_WEAK_EXPORT;

/**                                                                                                     /**
* Set the transport info structure (deprecated).                                                        * Set the transport info structure (deprecated).
*                                                                                                       *
* @deprecated This function still exists for compatibility with the                                     * @deprecated This function still exists for compatibility with the
* earlier transport interface, but it does nothing.  Instead, define                                    * earlier transport interface, but it does nothing.  Instead, define
* a ::JackTimebaseCallback.                                                                             * a ::JackTimebaseCallback.
*/                                                                                                      */
void jack_set_transport_info (jack_client_t *client,                                                    void jack_set_transport_info (jack_client_t *client,
                              jack_transport_info_t *tinfo) JACK_OPTIONAL_WEAK_DEPRECATED_EXPORT   |                                  jack_transport_info_t *tinfo) JACK_OPTIONAL_WEAK_EXPORT;




/**                                                                                                     /**
*  Ports have unique ids. A port registration callback is the only                                      *  Ports have unique ids. A port registration callback is the only
*  place you ever need to know their value.                                                             *  place you ever need to know their value.
*/                                                                                                      */
typedef uint32_t jack_port_id_t;                                                                        typedef uint32_t jack_port_id_t;

/**                                                                                                |    typedef uint32_t jack_port_type_id_t;
*  to make jack API independent of different thread implementations,                              <
*  we define jack_native_thread_t to pthread_t here.                                              <
*  (all platforms that jack1 runs on, have pthread)                                               <
*/                                                                                                <
typedef pthread_t jack_native_thread_t;                                                            <



/**                                                                                                     /**
* the new latency API operates on Ranges.                                                               * the new latency API operates on Ranges.
*/                                                                                                      */
                                                                                                   >    PRE_PACKED_STRUCTURE
struct _jack_latency_range                                                                              struct _jack_latency_range
{                                                                                                       {
    /**                                                                                                     /**
     * minimum latency                                                                                       * minimum latency
     */                                                                                                      */
    jack_nframes_t min;                                                                                     jack_nframes_t min;
    /**                                                                                                     /**
     * maximum latency                                                                                       * maximum latency
     */                                                                                                      */
    jack_nframes_t max;                                                                                     jack_nframes_t max;
};                                                                                                 |    } POST_PACKED_STRUCTURE;






                                                                                                   >    /**
                                                                                                   >     * Prototype for the client supplied function that is called
                                                                                                   >     * whenever the port name has been changed.
                                                                                                   >     *
                                                                                                   >     * @param port the port that has been renamed
                                                                                                   >     * @param new_name the new name
                                                                                                   >     * @param arg pointer to a client supplied structure
                                                                                                   >     *
                                                                                                   >     * @return zero on success, non-zero on error
                                                                                                   >     */
                                                                                                   >    typedef int (*JackPortRenameCallback)(jack_port_id_t port, const char* old_name, const char* new
                                                                                                   >




/**                                                                                                     /**
* Transport states.                                                                                     * Transport states.
*/                                                                                                      */
typedef enum {                                                                                          typedef enum {

        /* the order matters for binary compatibility */                                                        /* the order matters for binary compatibility */
        JackTransportStopped 0,       /**< Transport halted */
        JackTransportRolling 1,       /**< Transport playing */
        JackTransportLooping 2,       /**< For OLD_TRANSPORT, now ignored */
        JackTransportStarting 3,      /**< Waiting for sync ready */
                                                                                                   >            JackTransportNetStarting = 4,   /**< Waiting for sync ready on the network*/

} jack_transport_state_t;                                                                               } jack_transport_state_t;


typedef uint64_t jack_unique_t;         /**< Unique ID (opaque) */                                      typedef uint64_t jack_unique_t;         /**< Unique ID (opaque) */

/**                                                                                                     /**
* Optional struct jack_position_t fields.                                                               * Optional struct jack_position_t fields.
*/                                                                                                      */
typedef enum {                                                                                          typedef enum {

        JackPositionBBT 0x10, /**< Bar, Beat, Tick */
        JackPositionTimecode 0x20, /**< External timecode */
        JackBBTFrameOffset 0x40, /**< Frame offset of BBT information */
        JackAudioVideoRatio 0x80, /**< audio frames per video frame */
        JackVideoFrameOffset 0x100  /**< frame offset of first video frame */
} jack_position_bits_t;                                                                                 } jack_position_bits_t;

/** all valid position bits */                                                                          /** all valid position bits */
#define JACK_POSITION_MASK (JackPositionBBT|JackPositionTimecode|JackBBTFrameOffset|               |    #define JACK_POSITION_MASK (JackPositionBBT|JackPositionTimecode)
                            JackAudioVideoRatio|JackVideoFrameOffset)                              <
#define EXTENDED_TIME_INFO                                                                              #define EXTENDED_TIME_INFO






#ifndef JACK_WEAK_EXPORT                                                                                #ifndef JACK_WEAK_EXPORT
#ifdef __GNUC__                                                                                         #ifdef __GNUC__
/* JACK_WEAK_EXPORT needs to be a macro which                                                           /* JACK_WEAK_EXPORT needs to be a macro which
   expands into a compiler directive. If non-null, the directive                                           expands into a compiler directive. If non-null, the directive
   must tell the compiler to arrange for weak linkage of                                                   must tell the compiler to arrange for weak linkage of
   the symbol it used with. For this to work full may                                                      the symbol it used with. For this to work full may
   require linker arguments in the client as well.                                                         require linker arguments in the client as well.
*/                                                                                                      */
#define JACK_WEAK_EXPORT __attribute__((WEAK_ATTRIBUTE))                                           |
                                                                                                   >    #ifdef WIN32
                                                                                                   >        /*
                                                                                                   >            Not working with __declspec(dllexport) so normal linking
                                                                                                   >            Linking with JackWeakAPI.cpp will be the preferred way.
                                                                                                   >        */
                                                                                                   >        #define JACK_WEAK_EXPORT
                                               