<!-- 
.. link: 
.. description: 
.. tags: philippine-transit-app, programming
.. date: 2013/07/23 14:48:29
.. title: GraphServer
.. slug: graphserver
-->

Link: [http://graphserver.github.io/graphserver/](http://graphserver.github.io/graphserver/)

One other routing webapp I saw was GraphServer. It's actually more of a general purpose Graph library which supports GTFS and OSM data than an actual dedicated routing software like OpenTripPlanner. It's also based off python and C instead of Java, so it feels a lot less heavy.

The instructions on the website are already pretty good. There are just some minor errors with it. Where it says `gs_gtfsdb_build`, you should actually use `gs_gtfsdb_compile`. Also, when running `gs_osmdb_compile` you might need to use `-t` for tolerant in case you follow the instructions on chopping up the original OSM data.

A nice suggestion from the GraphServer instructions was to crop the OSM data to minimize the graph size. This is actually quite helpful if you downloaded the entire Philippine OSM dump. It reduced the original 900MB file to 135MB which was a lot more workable. I did hit a problem with their instructions though. The linked version of osmosis is an old one, which doesn't support 64-bit ids. The [latest version of Osmosis](http://wiki.openstreetmap.org/wiki/Osmosis) easily did the job though.

The actual routing though, was not exactly good. I only tried one route which should normally take 1-2 transfers, it suggested a route which involved 4+ transfers. It also didn't provide any alternate routes aside from that one. I'm not sure if it's a limitation of the provided routeserver, but I didn't bother checking if it supported parameters which might provide better routes.

I think graphserver could be useful, but it seems more involved than say OpenTripPlanner. There do seem to be people who use graphserver for their routing apps, but for the bounds of the contest, or just as a side project, it might require too much effort.