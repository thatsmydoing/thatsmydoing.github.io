<!-- 
.. link: 
.. description: 
.. tags: philippine-transit-app, programming
.. date: 2013/07/28 16:26:31
.. title: Jeep and Bus Schedules
.. slug: jeep-and-bus-schedules
-->

Wouldn't it be wonderful if there were no buses or jeepneys in the Philippines over the weekends? It would truly be a cyclist's paradise. Imagine biking along EDSA, normally that would be a death sentence, but according to the GTFS data, you shouldn't worry. I can assure you, it's still a death sentence.

The GTFS spec defines 2 ways of statically specifying trip schedules. You can define the exact times that a service will arrive at a stop. You can also specify between what times the service is active and how often a new bus or jeep leaves the first stop. You also define which days those rules apply. You could say every MWF, the bus operates from 9:00AM to 9:00PM and every TTH, the bus services from 3:00AM to 11:00PM.

This should be sufficient in theory, but real world conditions like traffic or the weather could throw the schedules off. To solve this, there's another spec, GTFS-realtime. This allows transit agencies to push temporary schedule updates and service announcements.

Like much everything else about the Philippine transit system, there aren't really any "schedules" to speak of. It's generally whenever the buses or jeeps feel like it. So we have no static schedules. We don't have a central agency or the tracking technology to make it feasible to push updates via GTFS-RT.

Ideally, we shouldn't bother inputting the schedule information into GTFS. Only the route data is really important for jeeps and buses. However, the schedule information is required in the GTFS, and routing apps wouldn't work without it. So we have to add a reasonable trip schedule for jeeps and buses.

The current GTFS data does define these trip schedules. We assume that jeeps and buses operate between 6:00AM and 11:00PM and a new jeep passes by every 10 minutes. Also, jeeps and buses are defined to only operate on weekdays.

While there might be jeeps who change routes or don't operate on weekends, I'm pretty sure that jeeps and buses run on weekends. We'll have to fix it ourselves temporarily since there's no central GTFS feed yet.

    :::sh
    # 724594 seems to be the service id used by jeeps and buses
    sed -i .bak '/^724594/ s/0,0/1,1/' calendar.txt

Another thing we could do is to adjust the time between buses, although the improvement is arguable. With the current 10 minutes between jeeps, it might provide some routes a significant advantage just because the timing is right. So you might get differing route suggestions depending on what time you planned the route. This makes sense when you're sure what the times are, so you can minimize the wait, but with jeeps, you never really know how long the wait will actually be.

If we set the frequency to one minute, it *might* give better routes by eliminating the timing issue. Or not, it's kind of hard to tell.

    :::sh
    # jeep and bus route ids tend to start with 72
    sed -i .bak '/^72/ s/,600/,60/' frequencies.txt

Overall, the problems we're having is a symptom of the mismatch between our transit system and the GTFS. It would be great if our transit system gets better and we don't need to do hackish things for it to fit the GTFS, but that's still a dream. For now, all we can really do is fit a triangle into a square hole.