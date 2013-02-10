* "jack server frontend" - virtual package provided by "jackd" and "jackdbus" packages
* "jack server (library)" - package containing libjackserver.so at least. Maybe also: essential tools, drivers, inprocess clients.
* "jackd" - package containing jackd binary. This package depends on "jack server (library)"
* "jackdbus" - package containing jackdbus binary and jack_control script. This package depends on "jack server (library)"
* "jack client library" - package containing libjack.so
* "jack client library dev" - package containging headers and pkgconfig file for libjack.so
* "jack server library dev" - package containging headers and pkgconfig file for libjackserver.so

"jack client library" and "jack server (library") packages should be version coupled because they use internal IPC protocol that is not guaranteed to be compatible between versions.

All different releases of JACK should be considered internally incompatible - that is, it should never be considered possible to mix versions of the JACK server with other versions of the JACK library/ies, drivers or internal clients. Packaging should ensure that no packages associated with different releases of JACK are ever installed simultaneously. Especially, having two versions of libjack.so installed simultaneously, often causes JACK programs using one libjack version not being able to operate with JACK server of other version.

* app that uses jack as only audio interfaces, (jack_keyboard for example), depends on "jack client library" package and on the "jack server frontend" virtual package
* app that uses jack as alternative (not the only) audio interface, (mplayer for exmaple), depends on "jack client library" and suggests "jack server frontend" virtual package
* ardour depends on "jack client library" and depends on "jackd" package, because it can start jack server through jackd binary.
* qjackctl depends on "jack client library" and depends on "jack server frontend" package, because it can start jack server through either jackd or jackdbus binary.
* laditools depend on "jackdbus" only
