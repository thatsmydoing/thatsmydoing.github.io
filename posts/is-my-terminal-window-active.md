<!--
.. title: Is My Terminal Window Active?
.. slug: is-my-terminal-window-active
.. date: 2015-06-07 16:20:45 UTC+08:00
.. tags: programming
.. category:
.. link:
.. description:
.. type: text
-->

I've been working in OSX for almost 3 years now, but I recently switched back to Linux because of all the problems people encountered with Yosemite. There are some things I missed from OSX though. One of which is [zsh-notify](https://github.com/marzocchi/zsh-notify). It's a zsh plugin that alerts you if your long-running task is complete, and whether it failed or not.

It's pretty convenient when you're compiling something and then go on to browse reddit while waiting. Usually, I spend too much time just reading and forget about the compilation entirely. With the plugin, I get the notification and maybe go back to work.

One nice feature it has is that if you're currently looking at the terminal window of the job that just finished, it won't notify you. It only notifies on windows that aren't currently in focus. To do this, it has to actually talk to Terminal.app or iTerm2 to see if the window and tab are active.

This is alright in OSX since those 2 are the generally most used terminal emulators. On Linux though, everyone has their own favorite terminal. Given that, I figured I could probably rely on talking to X to see if the window is active instead of each single terminal emulator. X can't tell if the tab is active though, but I don't use tabs in my current setup so it should still be good.

## xdotool

[Preliminary research](http://superuser.com/questions/382616/detecting-currently-active-window) reveals that we can easily get what the active window is with xdotool. `xdotool getactivewindow` gives us the X window id of the active one. Now all we need is a way to get the window id of the terminal we're in.

## First Attempt: $WINDOWID

Apparently, xterm and similar terminal emulators define an environment variable called `$WINDOWID` with the window id of the terminal. Obviously, this is too good to be true. In xterm and konsole the `$WINDOWID` was correct, but in VTE-based terminal emulators, `$WINDOWID` had the wrong value. In terminology, it didn't define `$WINDOWID` altogether. So `$WINDOWID` wasn't going to work.

## Second Attempt: xdotool search $MAGIC

My second idea was that you can use zsh to change the window title to a magic number and then just check if the active window is the same one as the window with the magic number. This sort of worked for most terminals, except konsole which does whatever it wants with the window title. There's also the problem of some zsh configs automatically settings the window title to the current command.

In hindsight, I could probably have just done `xdotool search --name xdotool` since in most cases, when you run the search, zsh or konsole will set the window name to the current command. Maybe that's another option I can explore some day.

## Third Attempt: $PPID

My third idea was another environment variable called `$PPID`, which is the process id of the parent of the shell. As it happens, the parent is the window containing the zsh instance. This is actually pretty consistent across most terminals. The only problem was if you launched zsh from another shell since your new zsh's parent will now be another zsh instance instead of an X window.

At first glance, launching zsh within zsh doesn't seem like something most people would do, but this is what happens when you run screen or tmux. To work around this, we can actually just save the original `$PPID` in a different variable and use that instead.

Now that we have the PID of the window from zsh, we can once again use xdotool to get the PID of the current active window with `xdotool getactivewindow getwindowpid`. We just simply compare that with our `$PPID` and we can tell if we're in an active window or not. Overall, this approach worked surprisingly well so that's the final solution I went with.
