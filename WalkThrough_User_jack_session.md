session support is built into jack. currently only in jack1 svn, tschack git and jack2 git.


# session manager

at the current state there are 2 sessionmanagers. *qjackctl_' trunk and '_pyjacksm*.
it will live in its own git repository.

	
	git clone git://hochstrom.endofinternet.org/pyjacksm.git
	

install it with:

	
	python setup.py install
	

then start `jacksmtray`

you might want to look at this screencast:
http://hochstrom.endofinternet.org/files/pyjacksm-cast01.ogv

a developer walk through is available at [[WalkThrough_Dev_JackSession]].

# apps supporting jack-session

* *jack_simple_session_client* which is simply a test client that comes with jack
* *jack-rack* git version git://hochstrom.endofinternet.org/jackrack.git
* *specimen* svn
* *seq24* ([patch](http://trac.jackaudio.org/attachment/wiki/WalkThrough/User/jack_session/jack-session.patch?format=raw) sent to ML)
* *ghostess* patch
* *guitarix*
* *fst* is close.
* *ardour3* should work fine.
* i think *qtractor* too
* *yoshimi* http://www.graggrag.com/yoshimi/yoshimi-0.059-pre0.tar.bz2 looks fine.
* *bristol* is reported to have jack-session support too
* *connie* http://bitbucket.org/horo/connie
* *lv2_jack_host* svn http://drobilla.net/software/slv2
* *ingen* svn http://drobilla.net/software/ingen
