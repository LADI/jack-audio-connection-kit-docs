This document describes jack2 source repositories, branches and the associated practices.

jack2 and jack1 codebases share a common Subversion repository. The RepositoryLayout page describes the repository layout in detail. jack2 release tarballs are released from the jack2 svn trunk. Paul Davis gives SVN commit access to new developers (do we have policy for this?). 

# Practices
Although svn commit access is shared between jack1 and jack2, the accepted practice is that only Stephane Letz and Nedko Arnaudov commit to jack2 trunk. Intrusive or controversial changes must be first approved by Stephane Letz.

# SVN Branches
* trunk - the mainline of development
* libjacknet -  a branch with some cleanup and CELT support for NetJack2, as well as a exposed API (the net.h file in public headers) to access NetJack protocol without having to run a Jack server. I did that for some experiments with iPhone for instance.  But since there is still this NetJack1/NetJack2, I don't know what do to with this development on the long term... (Stéphane)
* pipelining -  is the stuff allowing to better use mutiprocessor machines with pure sequential part in the jack graph.

The http://repo.or.cz/w/jack2.git repository is often used for cooperation among jack2 developers. Nedko Arnaudov gives push access to new developers. GIT is a distributed and thus it is easy to maintain a jack2 branch in a different repository. GIT has other advantages over Subversion but they are beyond the scope of this wiki page.

The repo.or.cz GIT repository in fact is a git-svn repository as well. It can be used for speeding up the creation of a new local git-svn repositories (the instructions are at http://repo.or.cz/w/jack2.git). It is important to note that *the GIT master branch contains only git-svn commits*. Pure git commits (ones that are not created by git-svn) must not be pushed to the master branch. The GIT repository also has the non-trunk git-svn remote branches. To view all the git-svn remote refs:

	
	git ls-remote git://repo.or.cz/jack2.git | grep refs/remotes
	

# Practices
	
	1) we want to develop and collaborate on a "public" git branch, so one of us create a local branch, push it on repo.or.cz, the others pull it do some developments... At some point we do a merge on master (possibly getting first new stuff done on SVN with 'git svn rebase") and push in on SVN with "git svn dcommit" and on git with "git push cz master "
	
	2) the other is private dev: same as 1) but "rebase" can be preferred over "merge" to sync with master, at some point push in on SVN with "git svn dcommit" and on git with "git push cz master "
	
## Updating git-svn branches
When there are svn commits that are not yet in the GIT repo, the following steps have to be made in order to sync the git-svn branches:
	
	git stash # ensures that the working tree is clean
	git checkout master # switch the working three to the master branch
	git fetch cz # get new commits through git (see below)
	git merge --ff-only cz/master # update the local master branch to have the new commits
	git svn fetch # fetches new svn commits
	git rebase trunk # fast forward the git master branch to the latest svn trunk
	git push cz master libjacknet pipelining # push the new commits to the public GIT repo. if there are other svn remote refs that were updated during git-svn fetch, push them as well.
	

Before fetching new commits through git-svn, they are eventually fetched from the public repo. This is needed because SVN commits have only one parent but GIT merge commits can have two (or even more) parents. Merge commits that happened originally in git but were later git-svn dcommit-ed, have two parents. If you obtain these commits through git-svn, they will have single parent as opposed to the commits that are in the public GIT repo - they have two parents and thus their SHA-1 is different.

## Merging a git branch to svn trunk
Here are steps to merge a hypothetical dev branch called mydevbranch into svn trunk.
	
	git stash # ensures that the working tree is clean
	git checkout master # switch the working three to the master branch
	git fetch cz # get new commits through git (see above)
	git merge --ff-only cz/master # update the local master branch to have the new commits
	git svn rebase # fetch new commits that may be in svn only
	git merge --no-ff mydevbranch # do the git merge, creates commit with two parents
	git svn dcommit # create a new svn commit, modifies the previous commit but the modified commit will still have two parents
	git push cz master # mandatory push to the public repo
	

# Branches
* master - tracks the svn trunk
* ladi - collects source modifications that are usable for LADI setups, especially [ladish](http://ladish.org) ones. It is based on latest jack2 release.
* ladi-experimental - as ladi but contains changes that happened in the svn trunk that are not yet released in a tarball.
* no-self-connect - a branch that adds new engine option to JACK. This engine option, when enabled, allows self-connect policy to be enforced. I.e. when this option is enabled, apps that attempt to connect their own ports (often to system ones) will be restricted.

We maintain a [[jack2_branching_history]] of branch related operations (like new branch creation, branch deletion and [important] merges). 
