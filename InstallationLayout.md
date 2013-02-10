This page represents installation layout on a Linux system.

# Binaries
This includes:
* jackd - classic JACK launcher executable
* jackdbus - D-Bus JACK service (installed if D-Bus integration is built)
* examples clients
* tools

They are installed in <PREFIX>/bin/

# Libraries
libjack and libjackserver are installed in <PREFIX/lib/

libtool-like install is made with .la files and symbolic links, like this:
	
	libjack.la
	libjack.so --> libjack.so.0.0.28
	libjack.so.0 --> libjack.so.0.0.28
	libjack.so.0.0.28
	
## Drivers
jack drivers are installed into <PREFIX>/lib/jack/
## in-process clients
in-process clients are installed into <PREFIX>/lib/jack/
## D-Bus service file
If jackdbus is being installed, D-Bus service is installed too. This file instructs D-Bus session bus how to activate JACK controller object upon request.

By default org.jackaudio.service file is installed in <PREFIX>/share/dbus-1/services/

However user may force using real D-Bus service directory by specifying --enable-pkg-config-dbus-service-dir

<PREFIX>/share/dbus-1/services/ will differ from one specified in pkg-config file of D-Bus when they are installed in different prefixes. If service file is installed in different prefix, D-Bus session bus daemon should be instructed to search in different directory (out of scope of JACK installation process, as implemented in distribution tarball).

# Headers
They are installed in JACK specific header directory, <PREFIX>/include/jack/
# pkg-config
jack.pc is installed in <PREFIX>/lib/pkgconfig/
# User documentation
Man pages are installed in <PREFIX>/share/man/man1/
# Development documentation
HTML documentation is installed in JACK specific directory <PREFIX>/share/jack-audio-connection-kit
Index is of the HTML documentation is within <PREFIX>/share/jack-audio-connection-kit/reference/html/

Some unusual things related to installation relocatability:
* in-process clients and drivers are loaded from fixed path (<PREFIX>/lib/jack/), specified literally during build. Drivers load directory may be overridden using JACK_DRIVER_DIR environment variable. ATM there is no way to override in-process client directory.
* D-Bus session bus daemon configuration may need modification to be able to auto-activate JACK controller service.
* jack.pc file contains PREFIX
