# Name

jackdrc - jack autostart configuration

# Description

JACK is evolving a mechanism for automatically starting the server when needed. Any client started without a running JACK server will attempt to start one itself using the command line found in the first line of $HOME/.jackdrc if it exists, or /etc/jackdrc if it does not. If neither file exists, a built-in default command will be used, including the -T flag, which causes the server to shut down when all clients have exited.

# Environment

As a transition, this only happens when $JACK_START_SERVER is defined in the environment of the calling process. In the future this will become normal behavior. In either case, defining $JACK_NO_START_SERVER disables this feature. 

# Files

* /etc/jackdrc - global defaults
* $HOME/.jackdrc - per-user override

# See also

* [wiki:jackd(1)]

