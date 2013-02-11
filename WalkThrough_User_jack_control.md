_jack_control_ is a Python based tool that allows to control the JACK server, using the newly defined DBUS service available in JACK2. It can be used when JACK has been compiled and installed with the DBUS layer. You can get and install Jack2 from SVN with :
	
	  git clone https://github.com/jackaudio/jack2
	
  On Linux to build JACK with DBUS support, configure, build and install using :
	
	  ./waf configure --dbus
	  ./waf build
	  ./waf install (in root)
	

The _jack_control_ tool will be then installed. Look at what commands are available:

	
	Usage: jack_control [command] [command] ...
	Commands:
	    exit                       - exit jack dbus service (stops jack server if currently running)
	    status                     - check whether jack server is started, return value is 0 if runing and 1 otherwise
	    start                      - start jack server if not currently started
	    stop                       - stop jack server if currenly started
	    dl                         - get list of available drivers
	    dg                         - get currently selected driver
	    ds <driver>                - select driver
	    dp                         - get parameters of currently selected driver
	    dpd <param>                - get long description for driver parameter
	    dps <param> <value>        - set driver parameter
	    il                         - get list of available internals
	    ip <name>                  - get parameters of given internal
	    ipd <name> <param>         - get long description for internal parameter
	    ips <name> <param> <value> - set internal parameter
	    iload <name>               - load internal
	    iunload <name>             - unload internal
	    ep                         - get engine parameters
	    epd <param>                - get long description for engine parameter
	    eps <param> <value>        - set engine parameter
	

Commands allow to list the available drivers (backend), list parameters of all drivers, select a given driver, possibly set it's parameters, and start the JACK server with it. All server parameters can also be listed and changed. Internal clients can be listed and their parameters listed and changed.

State of server, drivers and internal clients are automatically saved and restored (using an XML based file).
