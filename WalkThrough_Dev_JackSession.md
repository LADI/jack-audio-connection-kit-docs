

an approach to embed a really simple and straight forward session api into jack.


# Definition of Session

In this context a session means any number of applications on the jack graph which should behave like a single application in terms of saving and loading state.
From the users point of view it would be favourable to be able to load and save complex jack setups, like they were a single app.
Additionally it might be nice to be able to have 2 or more such sessions loaded at once.

# Basic goals

Get all apps to save their state with a single click/command, and be able to reload this. including connections.
It should also be possible to load a session while another one is already loaded.

# requirements


- an SM callback

- a way to invoke save and reply to the SM callback

- a way to restore the UUID when jack_client_open() is called

in most apps there is a thread safe way to save state from the gui thread.
So most apps will want to forward the session event from the callback to a callback in the GUI thread. for gtk+ the easiest way to achieve this
is g_idle_add(). for gtk apps the session_cb will therefor mainly look like this:

	
	int gui_session_cb( void *data )
	{
	    jack_session_event_t *ev = (jack_session_event_t *) data;
	    char filename[256];
	    char command[256];
	
	    snprintf( filename, sizeof(filename), "%smyfile.state", ev->session_dir );
	    snprintf( command,  sizeof(command),  "my_app -U %s ${SESSION_DIR}myfile.state", ev->client_uuid );
	
	    your_save_function( filename );
	
	    ev->command_line = strdup(command);
	    jack_session_reply( jack_client, ev );
	
	    if( ev->type == JackSessionSaveAndQuit )
	         gtk_main_quit();
	
	    jack_session_event_free( ev );
	
	    return 0;
	}
	
	void session_cb( jack_session_event_t *event, void *arg )
	{
	    g_idle_add( gui_session_cb, event );
	}
	

now we must make sure that the -U <uuid> option specifies this parameter to jack_client_open() again.

	
	    char *uuid=NULL; // fill it from getopt or however you parse commandline. 
	    
	    jack_client_open( "myapp", JackSessionID, NULL, uuid )
	


# client API

	
	/**
	 * session event types.
	 *
	 * if a client cant save templates, i might just do a normal save.
	 *
	 * the rationale, why there is no quit without save, is that a client
	 * might refuse to quit when it has unsaved data.
	 * however some other clients might have already quit.
	 * this results in too much confusion, so we just dont support that.
	 * the session manager can check, if the saved state is different from a previous
	 * save, and just remove the saved stuff.
	 *
	 * (an inquiry function, whether a quit is ok, followed by a quit event
	 *  would have a race)
	 */
	enum JackSessionEventType {
	    JackSessionSave = 1,
	    JackSessionSaveAndQuit = 2,
	    JackSessionSaveTemplate = 3
	};
	
	typedef enum JackSessionEventType jack_session_event_type_t;
	
	enum JackSessionFlags {
	    /**
	     * an error occured while saving.
	     */
	    JackSessionSaveError = 0x01,
	    /**
	     * client needs to be run in a terminal.
	     */
	    JackSessionNeedTerminal = 0x02
	};
	
	typedef enum JackSessionFlags jack_session_flags_t;
	
	struct _jack_session_event {
	    /**
	     * the actual type of this session event.
	     */
	    jack_session_event_type_t type;
	
	    /**
	     * session_directory with trailing separator
	     * this is per client. so the client can do whatever it likes in here.
	     */
	    const char *session_dir;
	
	    /**
	     * client_uuid which must be specified to jack_client_open on session reload.
	     * client can specify it in the returned commandline as an option, or just save it
	     * with the state file.
	     */
	    const char *client_uuid;
	
	    /**
	     * the command_line is the reply of the client.
	     * it specifies in a platform dependent way, how the client must be restarted upon session reload.
	     *
	     * it should contain ${SESSION_DIR} instead of the actual session dir.
	     * this would basically make the session dir moveable.
	     *
	     * memory will be freed along with jack_session_event_free()
	     * initially set to NULL by jack;
	     */
	    char *command_line;
	
	    /**
	     * flags to be set by the client. normally left 0.
	     */
	    jack_session_flags_t flags;
	
	    /**
	     * future flags. will be set to zero for now.
	     */
	    uint32_t future;
	};
	
	typedef struct _jack_session_event jack_session_event_t;
	
	/**
	 * Prototype for the client supplied function that is called
	 * whenever a session notification is sent via jack_session_notify().
	 *
	 * The session_id must be passed to jack_client_open on session reload (this can be
	 * done by specifying it somehow on the returned command line).
	 *
	 * @param event the event_structure.
	 * @param arg pointer to a client supplied structure
	 */
	typedef void (*JackSessionCallback)(jack_session_event_t *event, void *arg);
	
	/**
	 * Tell the JACK server to call @a save_callback the session handler wants
	 * to save.
	 *
	 * setting more than one session_callback per process is probably a design
	 * error. if you have a multiclient application its more sensible to create
	 * a jack_client with only a session callback set. 
	 *
	 * @return 0 on success, otherwise a non-zero error code
	 */
	int jack_set_session_callback(jack_client_t *client,
				    JackSessionCallback session_callback,
				    void *arg) JACK_WEAK_EXPORT;
	
	/**
	 * reply to a session_event
	 *
	 * this can either be called directly from the callback, or later from a different thread.
	 * so its possible to just stick the event pointer into a pipe and execute the save code
	 * from the gui thread.
	 *
	 * @return 0 on success, otherwise a non-zero error code
	 */
	
	int jack_session_reply( jack_client_t *client, jack_session_event_t *event ) JACK_WEAK_EXPORT;
	
	
	/**
	 * free memory used by a jack_session_event_t
	 * this also frees the memory used by the command_line pointer.
	 * if its non NULL.
	 */
	
	void jack_session_event_free (jack_session_event_t *event) JACK_WEAK_EXPORT;
	
	
	/**
	 * get the assigned uuid for client.
	 * safe to call from callback and all other threads.
	 * memory needs to be freed.
	 */
	
	char *jack_client_get_uuid (jack_client_t *client) JACK_WEAK_EXPORT;
	
	

# Example Code

	
	
	void
	session_callback (jack_session_event_t *event, void *arg)
	{
	        // this is a direct reply and we dont have state to save here.
	        // in a gtk app we would forward the event to the gui thread
	        // using g_idle_add() and execute similar code there.
	
		char retval[100];
	
		snprintf (retval, 100, "jack_simple_client %s", event->client_uuid);
		event->command_line = strdup (retval);
	
		jack_session_reply( client, event );
	
		if (event->type == JackSessionSaveAndQuit) {
			simple_quit = 1;
		}
	
		jack_session_event_free (event);
	}
	
	int 
	main( int argc, char **argv )
	{
	
	        // [some stuff deleted]
	
		if( argc == 1 )
			client = jack_client_open (client_name, JackNullOption, &status );
		else if( argc == 2 )
			client = jack_client_open (client_name, JackSessionID, &status, argv[1] );
	
	        // [some more stuff]
	
		/* tell the JACK server to call `session_callback()' if
		   the session is saved.
		*/
	
		jack_set_session_callback (client, session_callback, NULL);
	
	        // [even more stuff]
	
		while (!simple_quit)
			sleep(1);
	
	}
	
	

see also: [[WalkThrough_User_jack_session]]
