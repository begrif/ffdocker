ffdocker
--------

This is my personal *Firefox-in-a-Container* config. It's not going to
be for everyone. I use it to run a locked down Firefox instance on my
Linux desktop.

My goal is to isolate Firefox from the base computer for an added level
of security. Any security exploit in Firefox will have to break out of
the container to harm the system. (And I'm including "access webcam",
"use microphone" and even "play audio" as as "harms" protected by this.)

Since it is a container for an app on an X11 desktop, there's a chunk of
config for that. And since it's got *X11* access, a Firefox exploit that
targets that windowing system will not be hindered by this container. 

One tricky thing I do here is install Firefox inside the container,
but run a copy that is outside the container. I do this so that I do
not need to rebuild the container for a new version of Firefox, just
update the -v mount option in the runner shell script. 

Setup
-----

For me, the first thing to do was create a new profile for use by this.
The profile lives outside the container, so that the container is
stateless. I used the Firefox GUI to create the profile, starting with:

`firefox -ProfileManager -no-remote`

(I don't know who named the options for Firefox, but that camel case one
grinds my gears.) Then I chowned those files to a new user which matches
the user I have inside the container. (Quit Firefox before the chown.)

For the last few years on Linux, the browser profiles are configured in
`$HOME/.mozilla/firefox/profiles.ini`. That will tell you the directory
name for the newly created profile, and the directory will probably be
under `$HOME/.mozilla/firefox/`. 

```text
$ cat $HOME/.mozilla/firefox/profiles.ini
[General]
StartWithLastProfile=0

[Profile0]
Name=default
IsRelative=1
Path=a1b2c3d4.default

[Profile1]
Name=aug-2017-tmp
IsRelative=1
Path=abcdef12.aug-2017-tmp

[Profile2]
Name=Docker
IsRelative=1
Path=123abcde.docker
Default=1
```

From that I see that `$HOME/.mozilla/firefox/123abcde.docker/` is the
directory to recursively chown. I also make sure that the profiles.ini
file is readable.

```sh
sudo chown -R 8080 $HOME/.mozilla/firefox/123abcde.docker/
chmod go+x $HOME/.mozilla $HOME/.mozilla/firefox 
chmod go+r $HOME/.mozilla/firefox/profiles.ini
```

I then build my Docker image with my wrapper script:

```
run-ffdocker build
```

And, assuming that worked, run it with:

```
run-ffdocker run [ -firefox -options -here ]
```

Config Comments
---------------

The runner script takes care of `xhost` access controls and mounts the
typical X11 socket directory inside the container. 

The runner does not daemonize and spews Firefox stdout to stdout. I'm
okay with this, you might not be.

A directory `/tmp/downloads`, is provided for accessing the downloads
inside the browser. You can copy stuff there, too, for uploading. To get
this to work cleanly, you and the user inside the container should be in
the same group. I use group id 1000 for that.

The entry point is `/usr/lib/firefox-latest/firefox` and the directory
`/usr/lib/firefox-latest` is mounted via the command line. I use the
regular tar.bz2 Firefox downloads to populate the directory I mount.

Observations
------------

I started running Firefox in this way in late 2015 / early 2016
and I've found only a few gotchas. One thing is keeping fonts up to
date. The Unicode standard changes approximately once a year, and
new glyphs are added. Things like emojis.

There's no attempt to get sound generated in the container to speakers
on the actual hardware. This is a feature.(TM)

Depending on how your printer is configured, you might not be able to
print directly. Print to file works, and then the file can be printed
from the command line.

Various niceties of your video hardware may be hidden behind
abstractions. You probably don't want to play graphics heavy browser
video games or videos. (See lack of sound, above.)

Author
------
Benjamin Elijah Griffin

Version 1.0 -- August 2017
