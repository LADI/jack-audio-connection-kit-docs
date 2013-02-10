# Problems starting in realtime mode

## Operation not permitted

By default, Jack starts up in realtime mode. If the user starting jack is not allowed to start realtime processes, the following error might occur:

 `cannot use real-time scheduling (FIFO at priority 10) [for thread -1210562896, from thread -1210562896] (1: Operation not permitted)`

In this case, you might want to set up your limits.conf to allow the audio group access to realtime priorities as documented at http://wiki.linuxmusicians.com/doku.php?id=system_configuration#limits.conf .

## Invalid argument

If you get the following error:

 `cannot use real-time scheduling (FIFO at priority -4) [for thread -1238631568, from thread -1238631568] (22: Invalid argument)`

..  try setting the 'interface' parameter correctly. (see also [http://linuxmusicians.com/viewtopic.php?f=4&t=2588&view=unread#p12107])