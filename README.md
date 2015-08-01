
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

Directory structure
===================

todo: determine specifics here. off the top of my head i'm thinking $BASEDIR/someDescriptiveName/*.sh, where "someDescriptiveName" identifies what the scripts are handling (e.g. updating subscription info, setting udev rules, adding an iptables rule for jump boxes) and $BASEDIR is whatever place puppet puts stuff when it runs (which we don't care about since we're staying in that directory by default anyway).

File naming scheme
==================

todo: list each possible script name and what it's purpose is.

Other thoughts
==============

I love the quote "Perfect is the enemy of done" (No, not Voltaire) -- I think it really applies here. Puppet's great for giant datacenters who have abstracted the concept of servers into objects in their code, and who are so disconnected from the hardware they may never even lay hands on any server they work on. Personally, I'm pretty far from that category, but still in a large enough environment to really need a solid system like Puppet. Problem is, I don't have the time to sit down and learn Puppet, and I'm often in a situation where it's almost easier to rip through 50 servers pasting a bash one-liner than it is to write puppet manifests to do the same thing.

Basically, I want to flip the concept of "things" in puppet into a concept of "actions". Sure we can write code to add users to the server, but wouldn't it be easier to just use 'useradd'? Same with every other tool a good admin knows off the top of their head. That, plus bash if/switch statements, make a powerful enough toolkit, and bash has been around since '89 so it's way more likely to be around in 5 years. The barrier to entry with writing bash scripts is significantly lower than what it takes to (a) get used to puppet's annoyingly "familiar yet just different enough to bug me" syntax (b) wrap your head around the concepts of puppet (c) actually get good at writing puppet code, reading logs, pushing things out in a graceful way to a test system easily without it being a cumbersome process... the list goes on.

My initial idea was actually just the functions – the other part isn't as well-defined in my head right now. I'll have to think more on how exactly that would all fit together. The simpler initial stage of development might make sense as simply a standard puppet manifest template and puppet.sh with bash functions. In this state, we would simply write normal puppet "exec"/"onlyif" blocks. The next stage would be to add dynamic generation of those exec/onlyif blocks and matching script names e.g. exec.sh and onlyif.sh (if onlyif.sh isn't present, we'll call exec.sh every time, and assume it's doing a quick check and exiting quickly if it doesn't need to do anything in that cycle.)

Actually, now that I think about it, I'm pretty sure that's what I meant to start with, I just forgot for a minute. There's a reason I wrote this down! Very difficult to translate visual ideas into written form without forgetting half of it along the way... maybe that's just me.

