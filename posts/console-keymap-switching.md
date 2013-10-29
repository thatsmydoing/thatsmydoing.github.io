<!--
.. link:
.. description:
.. tags: sysadmin, systemd
.. date: 2013/10/29 20:02:06
.. title: Console Keymap Switching
.. slug: console-keymap-switching
-->

At the office, we have some people who use DVORAK. Normally, this isn't a problem. To each his own after all. It does become a bit problematic though, when we're dealing with the servers around the office.

We normally leave the servers on QWERTY. After all, most people start off as QWERTY typists and migrate to something else. That said, it's apparently difficult to stay fluent in both. People tend to forget how to type in QWERTY once they learn DVORAK or something else. While it is true that they can just look a the keyboard while typing, my coworkers would prefer it to just be in DVORAK.

For the console, they'd typically do `sudo loadkeys dvorak` after logging in. The problem with this is, after they logout, the keymapping is still on DVORAK. This has been quite annoying for a few times since I can't even login to change the keymap. What I wanted was something like you get in the graphical login screens where you can pick your keymap before logging in. Apparently, there isn't a readily available thing for the console.

I googled around for solutions and came across [a nice idea](http://superuser.com/questions/548234/how-can-i-easily-toggle-between-dvorak-and-qwerty-keyboard-layouts-from-a-linux). You could alias `asdf` to load the DVORAK mapping and `aoeu` (the equivalent to asdf in DVORAK) to load the QWERTY mapping. This actually makes sense since you don't really have to know where the letters are. The only problem is, you once again have to be logged in to change the key mappings.

After some further searching, I found [something close to what I wanted](http://unix.stackexchange.com/questions/2884/toggle-between-dvorak-and-qwerty). Apparently, Alt+Up sends a KeyboardSignal keycode to the init process, which can act on that. It also works anywhere, even before being logged in. For SysVinit systems, you can just add a line to your inittab for a command to be run when Alt+Up is pressed.

In the office, however, we generally use Arch Linux which uses SystemD. But apparently, it also has a mechanism of accepting the Alt+Up press. It runs the kbrequest target whenever it gets the keypress. `kbrequest.target` is normally aliased to run the rescue service though, so you have to manually create the file in `/etc/systemd/system/kbrequest.target` and fill it with a description:

    [Unit]
    Description=kbrequest target

We can then add a service to be run whenever the target is called. Something like `/etc/systemd/system/keymap-switch.service`:

    [Unit]
    Description=Keymap Switch Service

    [Service]
    Type=oneshot
    ExecStart=/usr/local/bin/keymap-switch

    [Install]
    WantedBy=kbrequest.target

After enabling said service, we only need the actual keymap switcher, `/usr/local/bin/keymap-switch`. The StackOverflow answer provides different ways of detecting the current keymap so we know which one to switch to. Since we're using SystemD, we can use that instead for managing which keymap we're actually using. It stores the current settings inside `/etc/vconsole.conf`. We can also then switch keymaps by using `localectl set-keymap`.

    #!/bin/sh
    source /etc/vconsole.conf

    if [ "$TERM" = "dumb" ]; then
      if [ "$KEYMAP" = "dvorak" ]; then
        localectl set-keymap us
      else
        localectl set-keymap dvorak
      fi
    fi

After putting it all together, it works! We can switch keymaps on the fly by simply pressing Alt+Up.
