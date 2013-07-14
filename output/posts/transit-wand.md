<!-- 
.. link: 
.. description: 
.. tags: philippine-transit-app, programming
.. date: 2013/07/15 22:45:20
.. title: Transit Wand
.. slug: transit-wand
-->

Link: [http://transitwand.com](https://play.google.com/store/apps/details?id=com.conveyal.transitwand)

Overall, this was the simplest of the [open-source transit tools](http://philippine-transit.hackathome.com/use-this-code/) to actually get up and running. There's already a deployed instance of the server, and you can easily download the phone app via the [Play Store](https://play.google.com/store/apps/details?id=com.conveyal.transitwand). Even running the server by yourself didn't have any of the hiccups I had with GTFS Editor.

The phone app is actually quite simple. It allows you to capture a trip, which will record your GPS coordinates as you ride public transit. It also allows you to mark points of the trip where you stop and also how long the stop took. Lastly, it allows you to record embarking and disembarking passengers which is potentially useful for ridership data.

After doing a capture session, you can review the data on the phone. It will plot out the route on a map, with markers for the stops. You then either delete the data if it looks wrong, or you can upload it to the Transit Wand server. Uploading involves registering an account, but it's free and you don't even actually need to put in a username or anything. It simply registers the phone's IMEI on the server and gives you a 6-digit identifier.

You can then use the 6-digit identifier to view the data on Transit Wand's server, which is good since uploading any data automatically deletes it from the phone. There really isn't much else you can do with it though. It just allows you to view the data, and export it as a [Shapefile](https://en.wikipedia.org/wiki/Shapefile).

As is, this is purely a data collection client-server app. Barring looking at the database, there is no way to get a list of phones which have collected data. Only the person who initiated the data collection knows the 6-digit code to view their data. There's also no way to extract the ridership information from the server yet. This isn't to say that the data won't eventually go public though.

An interesting thing you *can* do with the Transit Wand data is import it into GTFS Editor to make a new route. You don't even have to manually download and upload the data. Just type in your 6-digit identifier and it will give you a list of routes you've captured via Transit Wand. This is wonderful as you get all the stop data, as well as the shape of the route.

I imagine these two tools were how the DOTC came up with all the GTFS data we have now. What I don't understand is why the shape data isn't present. Importing from Transit Wand already gets you shape data. There are even facilities to edit the shape within the editor if clean up is necessary. The only problem I saw was the fact that you can't easily move stops, you have to input coordinates to change the position.

It *might* also be possible that when the DOTC was still collecting the data, the route collection or editing features weren't present yet. That would just be lame and depressing though.

Overall, Transit Wand does what it's supposed to do. You collect data, and then upload it to a server. There is a lot of room for improvement though. It would be nice to have a better API that allows access to more of the data. Building in analysis tools for the ridership data might also be a welcome thing. I imagine it would also be great if you could encourage people to use the app and upload their own trips.
