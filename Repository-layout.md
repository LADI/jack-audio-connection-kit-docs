* The official repo is the subversion one (see below)
* There is a [git repo](http://repo.or.cz/w/jack2.git) for jack2 that tracks svn trunk in the master branch and has several git only branches.
   The [[jack2_branching]] page describes jack2 source repositories, branches and the associated practices. 

What used to be the old jackmp trunk is now part of the main repo. Or as Stephane puts it:

"like an alien jack2 is now somewhere in the old rusty space ship..."

To get your own personal alien, use the following svn url

http://subversion.jackaudio.org/jack/jack2/trunk

instead of 

http://subversion.jackaudio.org/jackmp/trunk

The easiest transition is to checkout a completely new tree for jackmp. svn switch most likely is not going to work since the merge allocated new revision numbers. For the normal jack tree, no changes are needed.

For reference.
	
	Transitional layout:
	====================
	
	branches/       (unchanged)
	    EXP/
	    clockfix/
	    jackit/
	    pbd/
	 
	tags/           (unchanged)
	    [...]
	 
	trunk/          (unchanged)
	    CVSROOT/
	    jack/
	    www/
	 
	jack2/          (added)
	    branches/
	        control/
	        direct/
	        exp/
	        fade/
	        midi/
	        network/
	        video/
	    tags/
	        [...]
	    trunk/
	 
	 
	Long-term
	=========
	 
	/trunk => /branches/1.0-ongoing
	/branches cleaned up
	 
	/jack2/branches merged/cleaned up, then => /branches
	/jack2/trunk => /trunk
	/jack2/tags/* => /tags
	
	
