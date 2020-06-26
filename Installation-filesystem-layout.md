This page represents installation layout on a Linux system.

# Binaries

This includes:

* `jackd` - classic JACK launcher executable
* `jackdbus` - D-Bus JACK service (installed if D-Bus integration is built)
* examples clients
* tools

They are installed in `<PREFIX>/bin/`.

# Libraries

The `libjack`, `libjacknet` and `libjackserver` libraries are installed in `<PREFIX>/lib/`.

A libtool-like installation is made with `.la` files and symbolic links, like this:
	
    libjack.la
    libjack.so --> libjack.so.0.1.0
    libjack.so.0 --> libjack.so.0.1.0
    libjack.so.0.1.0
	
## Drivers

Jack drivers are installed into `<PREFIX>/lib/jack/` as dynamic libraries.

## In-process clients

In-process clients are installed into `<PREFIX>/lib/jack/`.

# D-Bus service file

If `jackdbus` is being installed, a D-Bus service file is installed too. This file instructs the D-Bus session bus how to activate the JACK controller object upon request.

By default the `org.jackaudio.service` file is installed in `<PREFIX>/share/dbus-1/services/`.

However, a user may force using the real D-Bus service directory by specifying `--enable-pkg-config-dbus-service-dir` when configuring the the sources for building.

The D-Bus service file directory `<PREFIX>/share/dbus-1/services/` will differ from the one specified in the pkg-config file of D-Bus when they are installed in different prefixes. If the service file is installed in a different prefix, the D-Bus session bus daemon should be instructed to search in the appropriate directory (out of scope of JACK installation process, as implemented in the distribution tarball).

# Systemd unit

A systemd unit file is installed as `<PREFIX>/lib/systemd/user/jack@.service`. System-wide configuration  files for this systemd unit should be put in `/etc/jack` and user configuration files in `~/.config/jack`  with a `.conf` filename extension.

# Headers

C headers are installed in a JACK specific header directory, `<PREFIX>/include/jack/`.

# pkg-config

`jack.pc` is installed in `<PREFIX>/lib/pkgconfig/`.

# User documentation

Man pages are installed in `<PREFIX>/share/man/man1/`.

# Development documentation

HTML documentation is installed in a JACK specific directory `<PREFIX>/share/jack-audio-connection-kit`.

The index file of the HTML documentation is in `<PREFIX>/share/jack-audio-connection-kit/reference/html/`.

# Additional notes

Some unusual things related to installation relocateability:

* in-process clients and drivers are loaded from a fixed path (`<PREFIX>/lib/jack/`), specified literally during build. Drivers load directory may be overridden using the `JACK_DRIVER_DIR` environment variable. At the moment there is no way to override the in-process client directory.
* The D-Bus session bus daemon configuration may need modification to be able to auto-activate the JACK controller service.
* The `jack.pc` file contains `<PREFIX>`.