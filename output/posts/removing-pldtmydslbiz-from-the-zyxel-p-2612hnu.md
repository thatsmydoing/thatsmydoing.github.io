<!--
.. link:
.. description:
.. tags: sysadmin
.. date: 2013/11/27 10:12:31
.. title: Removing PLDTMyDSLBiz from the ZyXEL P-2612HNU
.. slug: removing-pldtmydslbiz-from-the-zyxel-p-2612hnu
-->

I've always thought that people were just too lazy to change their SSIDs when I see "PLDTMyDSLBizCafeJapan". It became apparent when we got our own PLDT line that it was because the bundled router/modem *does not* allow you to remove the prefix.

This is not the kind of thing you expect as a business customer. Even for home customers, I feel it's still a bit dishonest. I'd be fine if it was just the default SSID, but forcing people to have it as part of their SSID is like advertising that your company (I mean PLDT) is a douche.

Of course, we couldn't just leave the SSID prefix there, so we tried a number of things to get rid of it. There are articles for removing it from the [Prolink H5004N](http://www.phandroidinternet.com/2013/06/how-to-remove-on-wifi-name-or-ssid-on.html) or the [ZyXEL P-660HN-T1A](http://www.symbianize.com/showthread.php?t=730091) but not for the one we got which was the ZyXEL P-2612HNU-F1F.

We did still try the firebug/inspector tricks, but it seems that there is a server-side check that adds in the "PLDTMyDSLBiz". We tried a number of things, but the one that ultimately worked (and we had a good laugh about) was to backup the configuration, edit the dumped file and restore it.

The backup is actually just an XML file. You can search for SSID and change the parameter there. It's a bit annoying because the router has to restart after restoring the configuration, but it works!

A minor note, the router doesn't seem to support SSIDs with a comma (,) well. It just gets everything before the comma as the SSID for some reason.