# Whats this ?

For a basic explanation on what cgroups are, see Chapter 1 of [http://www.kernel.org/doc/Documentation/cgroups/cgroups.txt]

# Why does it touch jackd

On a kernel that has CONFIG_RT_GROUP_SCHED enabled, access to RT scheduling cpu bandwidth is controlled via cgroups.
The root cgroup has this setup correctly. However when libcgroup is installed and the cgconfig service has been started,
it creates a "sysdefault" cgroup and moves all tasks over there. In all distros i am aware of the sysdefault group does not have
RT bandwidth assigned to it. In this case jackd can not be started.

# Method 1

the easiest way would be to configure RT bandwith in the sysdefault cgroup.
By adding the following to `/etc/cgconfig.conf` 

	
	group sysdefault {
		perm {
			task {
				uid = root;
				gid = root;
			}
			admin {
				uid = root;
				gid = root;
			}
		}
		cpu {
			cpu.rt_runtime_us = 950000;
		}
	}
	

This is a quickfix. We will provide a more sophisticated way to do this very soon:

# Method 2

We have pending patches which move jack client RT threads into the cgroup of jackd.
(i hope i can commit them to svn before xmas)
So we only have to create an RT cgroup, and make sure jackd is moved into it, when its started.

the rtaudio cgroup is configured by adding 
	
	group rtaudio {
		perm {
			task {
				uid = root;
				gid = audio;
			}
			admin {
				uid = root;
				gid = root;
			}
		}
		cpu {
			cpu.rt_runtime_us = 950000;
		}
	}
	

to `/etc/cgconfig.conf` 

note that this would conflict with RT bandwidth configured for sysdefault cgroup.
(use either method 1 or 2, not both)

then add this to `/etc/cgrules.conf`

	
	@audio:jackd      cpu	rtaudio/
	

If you run into problems with other applications which require RT scheduling, you need to add these applications to `cgrules.conf` too
with the libcgroup support in jack most things will just work, though.

# Method 2.5

if you dont want to have `cgrulesengd` running all the time, you can also use jackds new -g option, which will probably also be committed soon.
with this option jackd will move itself to the correct cgroup.
it can be setup at configure time too with `./configure --with-default-cgroup="rtaudio"` for example. 


# Method 3 (for packagers)

i am working with upstream to provide an init script which also parses files in /etc/cgconfig.conf.d/
these files need to look like this:

	
	namespace {
	        cpu = /;
	}
	
	group rtaudio {
		perm {
			task {
				uid = root;
				gid = audio;
			}
			admin {
				uid = root;
				gid = root;
			}
		}
		cpu {
			cpu.rt_runtime_us = 950000;
		}
	}
	

The `namespace` keyword is not yet documented in the man page. libcgroup people are aware of this and hopefully fix it soon.

	
	for i in /etc/cgconfig.conf.d/*; do
	    cgconfigparser -l $i
	done
	

this will basically load all files.
