---
layout: post
title: "Building a (fairly) sane dev environment in Windows"
date: 2013-09-28 19:42
comments: true
categories: programming
---
I recently left a job that mandated Windows as a development
environment. Well, that's not strictly true -- they were happy enough
for me to use Linux in a VM, but that was ultimately more trouble than
it was worth.

I've had to dip into Windows before (having worked a job where the
server-side code was .Net), but I've never had to use it as a primary
dev environment. Long story short: it is *extremely* painful to use
Windows for modern web development, or any kind of development which
is not explicitly *for* Windows.

Here is a statement of the main problems, and the measures which can
get you at least close to a solution.

##Git
An odd one to start with perhaps, but it has a significant impact on
the rest of the setup. There are two ways you can get a usable Git
setup working on Windows: one is to use
[Cygwin](http://www.cygwin.com), of which more below, and one is
[Git for Windows](http://msysgit.github.io), otherwise known as
MSYS-Git. You can download a lightweight version of
MSYS-Git as [Git Bash](http://git-scm.com/downloads) -- which, you
shouldn't be surprised to hear, comes with an implementation of Bash
-- but I'm going to advise getting the
[full installer](http://code.google.com/p/msysgit/downloads/list?q=net+installer). It
should become clear why.

MSYS-Git is quite slow; apparently the version that you can use with
Cygwin is faster, but Cygwin was too much of an overhead for my purposes.

##The command line
Where to begin. The Windows command line is a  disaster, unless
all you need to do is change directories and open files. No grep. No
vi. No sed. The knock-on effects, on editors and other tools which rely
on these utilities, are not fun. I'd been recommended
[clink](http://code.google.com/p/clink/), which offers Bash-like
completion, but it's still ... the Windows command line.

The options, if you want Bash or something like it, are
[Cygwin](http://www.cygwin.com), effectively a complete replacement
for the Windows command system,
[GoW (Gnu on Windows)](https://github.com/bmatzelle/gow/wiki), a
lighter-weight port of a set of Gnu utilities, or
[MSYS](http://www.mingw.org/wiki/MSYS), a package similar to GoW,
intended for compiling Gnu utilities on Windows. An outside choice is
[Eshell](http://www.emacswiki.org/emacs/CategoryEshell) if you're an
Emacs user; I tried it, but found it didn't fit well with my workflow.

Cygwin is a very comprehensive solution: it presumes you're basically
going to *exist* within it, which really didn't work for me. GoW was
nice, but didn't play well with the Git-integrated Bash (see above),
and neither did a separate install of MSYS. For these reasons, I
suggest installing the complete MSYS-Git environment I mentioned
above: you get Git, Bash and a decent subset of the Gnu utilities, all well-integrated.

##A terminal emulator
That this is not a solved problem should give you some idea of The
State of Things. There are two contenders, neither of which really represents an
resounding victory:
[ConEmu](http://code.google.com/p/conemu-maximus5/) and
[Console2](http://sourceforge.net/projects/console/files/). (Incidentally,
are you noticing how much of this stuff is on Google Code? And
*SourceForge* for heaven's sake? The mind boggles). ConEmu is highly
configurable. I do not need my terminal emulator to be highly
configurable. Console2 is marginally easier to use, but pug ugly. I
went for Console2.

##Package management
The only real game in town here is
[Chocolatey](http://chocolatey.org). Like most community projects in
the Windows world, it's undermaintained and generally a bit thin, but at least
it's there. (And no disrespect to the maintainers here: it's numbers
I'm complaining about, not individual effort). Installs largely work;
uninstalls or upgrades often don't. But until
[Winbrew](https://github.com/nddrylliog/winbrew) is reliable, there we
are. I managed to get [ack](http://chocolatey.org/packages/ack)
installed via Chocolatey, so two cheers for it, at least; all the
packages mentioned above are available too.

##Runtime/language version managers
I'm talking about the likes of
[nvm](https://github.com/creationix/nvm) and
[chruby](https://github.com/postmodern/chruby) here. I used
[Nodist](https://github.com/marcelklehr/nodist) (because it was
available on Chocolatey and [nvmw](https://github.com/hakobera/nvmw)
wasn't); there's also [pik](https://github.com/vertiginous/pik) for
Ruby, but I couldn't get it working.

##Type
Call me frivolous, but I have a background in typography and I *cannot
stand* the look of type on Windows. The fault lies with the
anti-aliasing, which only works in the vertical plane; type looks
spindly and horrible, no matter how you set it up. If you're used to
modern-looking type, as rendered on OS X or Ubuntu, and you have any
sort of sensitivity to these things, you just can't live with it.

It's not a total solution, but
[MacType](https://code.google.com/p/mactype/) saved me clawing my eyes
out, at least.

##In summary
Attempting web development on Windows feels like being thrown back in
time five years or so. I've touched on the main pain points, but I
can't even remember how many, many times I had to ransack Stack
Overflow to find out why such-and-such a gem wouldn't work properly,
or dig about in the awful Registry. How many times I restarted the
machine to be told that Windows had decided to install 127
updates. How long and often I watched the spinny blue circle while a bleedin'
*Explorer window* opened. Ubuntu, *in a VM*, was considerably faster
and more usable in every way.

The larger problem is that the whole experience of working on Windows
is so rotten that *nobody wants to fix this stuff*. Plenty of GitHub
issues I followed, on large, respectable repos, were
unresolved. 'Windows,' said the maintainers. Won't fix. Closed.

I really can't blame them, and similarly, I can see why the
whole open-source movement is so backward on the Windows platform;
hardly anyone's there, and community is what keeps all this going.
I hope, for the sake of the people who have to use it, that it gets
better. I won't be hanging around to see, though.
