<!-- 
.. link: 
.. description: 
.. tags: philippine-transit-app, programming
.. date: 2013/07/07 10:32:36
.. title: Jeepney and Bus Routes
.. slug: jeepney-and-bus-routes
-->

In the [last post](philippine-transit-app-challenge.html), I talked about how we now have data about jeepney and bus routes in the Philippines. The data is actually in the [GTFS format](https://developers.google.com/transit/gtfs/), which is the format the Google Maps consumes transit data. Apparently, the government will be submitting the GTFS data later this year. Transit directions for Metro Manila in Google Maps would be wonderful. That said, it definitely raises the bar for the app challenge people.

In the last post, I mentioned the quality of the data isn't quite good. Even before seeing the data, I was already a bit unsure of it. The key problem is how you model the routes. The GTFS format was inherently designed for more well developed and organized transit agencies which isn't exactly what we have in the Philippines now.

One potential problem is the nature of the jeeps and buses. GTFS routes are a collection of trips which are a sequence of stops. However, we don't have jeepney stops, and even if we did they still just stop anywhere. There are also times where jeeps will take a shortcut if no passengers need to get dropped off along their normal route.

From what I've seen of the data, they handled the first problem well enough. Stops are defined as where people typically get on the jeep or bus. This is good, but they didn't define a shape for the routes. There is no information as to which exact roads they pass through. All we have to go by are the stops to show the route on a map.

![sample route](http://i.imgur.com/NSVlryE.jpg)

The problem isn't that bad though. The agencies could still add the shapes later on. Or maybe an app challenge participant could make an app around fixing the routes via crowd-sourcing or similar. The shape itself isn't that important for a rudimentary directions app, but if we want better apps, we will need better data.

There were also some minor issues with the data itself. Some of the files had extra columns. This normally isn't an issue, but it caused problems for [GTFS SQL importer](https://github.com/harrisony/gtfs_SQL_importer). There were also problems with matching the shape data with the stops when I tried it with [OneBusAway](http://onebusaway.org). They could probably be [fixed](https://github.com/OneBusAway/onebusaway-application-modules/wiki/Stop-to-Shape-Matching) but that's for another day.