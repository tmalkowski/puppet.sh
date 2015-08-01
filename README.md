
> Note to anyone who may stumble across this page – I had the idea and wanted to write down my thoughts before I forgot, but I'm not sure if this would ever be something I have the time to develop to the level of quality it would need to be for us to want to use it. Feel free to take the idea and run with it if you're so inclined, I'd love to see it happen.

puppet.sh - set of bash functions for inclusion in a puppet manifest (like init.d)
=========

The idea is pretty simple: Strip puppet down to it's bare bones, and staple on a repo containing bundles of bash scripts. Inside each bundle directory, you've got an exec.sh and an optional onlyif.sh, and other files as necessary. exec.sh and onlyif.sh will start in an environment that has defined functions like info/warn/error, access to facter data, and a few other useful bits of glue to strip away all the puppet syntax and focus on the details we really care about -- mainly, getting functional change control of all datacenter servers across the farm in a reasonably organized way.

Functions
=========

todo: list of functions that get imported by the puppet.sh script. top of my head, info/warn/error, facter data, puppet variables (get and set)

Puppet module
=============

todo: architect the layer of puppet code that sits around these bash scripts: calls exec, handles exit codes, logging info/warn/error, etc.

automate creation of a manifest from a template, results would look something like this:

	class puppetsh::module {
		exec { whateverDirectoryIsNamed:
			command => (whatever you do in puppet to get a file's contents),
			onlyif => (whatever you do in puppet to get a file's contents),
			logoutput => true
		}
	}



File naming scheme
==================

	$BASEDIR/
		directoryName/   ## should describe what the scripts are doing
			README.md    ## .md so it's nice with github web interface, could contain additional notes if necessary.
			exec.sh      ## required; without it, an error gets thrown into the puppet logs and nothing happens.
			onlyif.sh    ## optional; you could also just write an if [[ "whatever" ]] || exit 0 at the top of exec.sh.
			unless.sh    ## is this useful or does this confuse things?

todo: figure out what else might be useful here, or if this is really all that's necessary. off the top of my head it seems like this is all we'd need.

Other thoughts
==============

I love the quote "Perfect is the enemy of done" (No, not Voltaire) -- I think it really applies here. Puppet's great for giant datacenters who have abstracted the concept of servers into objects in their code, and admins/developers who are so disconnected from the hardware they may never even lay hands on any server they work on. Personally, I'm pretty far from that category, but still in a large enough environment to really need a solid system like Puppet for the scalability / performance / scaffolding it provides. Problem is, I don't have the time to sit down and learn new Puppet syntax each time I need to make a sweeping change, and I'm often in a time-sensitive situation (e.g. SSL vulnerability patching) where it's almost easier to rip through 50 servers pasting a bash one-liner than it is to write puppet manifests to do the same thing.

Basically, I want to flip the concept of "things" with "states" in puppet into a concept of "actions" with "required conditions". Sure we can write code to add users to the server using puppet manifest syntax, but wouldn't it be easier to just use 'useradd' (with an optional grep "^$username:" /etc/passwd, or just returning useradd's exit value)? What about iptables rules, using -L/grep and -I/save instead of puppet code to check for lines, modify lines, but only if the exact line is formatted with a certain number of spaces or it might fail... probably some puppet-iptables module out there no doubt that wants to take ownership of all the iptables rules. No thanks.

Same with every other tool a good admin knows off the top of their head... if you've already got a high quality toolbox filled with well-worn tools you know well, why not just use that? Throw in the simplicity of bash if/switch statements, and you've got a powerful enough toolkit to do just about anything you need. Doesn't hurt that bash has been around since '89 so pretty much everyone already knows it. The barrier to entry with writing bash scripts is insanely lower than what it takes to get used to puppet's annoyingly "familiar yet just different enough to bug me" syntax, wrap your head around the concepts of puppet, actually get good at writing puppet code, reading logs, pushing things out in a graceful way to a test system easily without it being a cumbersome process... the list goes on.

This doesn't solve all those problems mentioned above -- for example, there's still the issue of how to gracefully organize puppet environments so it's easy to work in your local dev environment, push changes into each environment as you go, then finally merge into production. I think that process is pretty well handled on the git side of things with our current infrastructure, using r10k+git branches to keep things organized. I'd like to add some functionality for the puppetmaster that tracks upstream commits and fires off an r10k deploy command whenever it sees new changes in the upstream repo, but it would have to be something fast enough to not require a long delay before the new environment was available: in my mind, that's less than the time it takes to switch from tab A (committing/pushing the change) to tab B (running puppet agent -t on a dev box). For now, we'll just put a pin in that.

The simpler initial stage of development might make sense as simply a standard puppet manifest template and something like /etc/puppet/functions which brings in bash functions. In this state, we would simply write normal puppet "exec"/"onlyif" blocks in a manifest pointing to those scripts. The next stage would be to add dynamic generation of those exec/onlyif blocks and matching script names e.g. exec.sh and onlyif.sh (if onlyif.sh isn't present, we'll call exec.sh every time, and assume it's doing a quick check and exiting quickly if it doesn't need to do anything in that cycle.)

Let's see, what did I forget? There's a reason I wrote this down! Very difficult to translate visual ideas into written form without forgetting half of it along the way... maybe that's just me.

