<!-- 
.. link: 
.. description: 
.. tags: philippine-transit-app, programming
.. date: 2013/07/13 21:15:09
.. title: Fare Data
.. slug: fare-data
-->

As part of the data released by the DOTC, we also have the [fare matrix](http://philippine-transit.hackathome.com/dataset-philippines-transit-information-service-gtfs/) for aircon buses, ordinary buses and jeeps. All as wonderful images. The data is also actually available from the [LTFRB website](http://ltfrb.gov.ph/main/farerates). Generally, the fare scheme is represented as "pay *X* pesos for the first *Y* kilometers, pay *Z* for every succeeding kilometer." Instead of a table, we can simply represent this as a formula instead,

    base_fare + (distance - initial) * per_km

The relevant values for the three services are:

<table>
  <tr>
    <td>type</td>
    <td>base_fare</td>
    <td>initial</td>
    <td>per_km</td>
  </tr>
  <tr>
  	<td>bus aircon</td>
  	<td>12.00</td>
  	<td>5 km</td>
  	<td>2.20</td>
  </tr>
  <tr>
  	<td>bus ordinary</td>
  	<td>10.00</td>
  	<td>5 km</td>
  	<td>1.85</td>
  </tr>
  <tr>
  	<td>jeep aircon</td>
  	<td>8.00</td>
  	<td>4 km</td>
  	<td>1.40</td>
  </tr>
</table>

It isn't as simple as that though. Fares are also rounded to the nearest 25 centavos. So we'd need to round them off correctly. This can be achieved by doing,

    round(calculated_fare * 4.0)/4.0

There's also the discounted fare for students, senior citizens and persons with disability. They get 20% off the fare (prior to rounding) and the resulting fare is rounded off as well.

Doing just this, we actually do get the same results as the fare matrices in the image for the most part. There are some discrepancies with the discounted jeep fares. I've tried to resolve it by tweaking around with the formulas, but it really doesn't make sense in any way. I presume these were manually adjusted for one reason or another.

Here's a [script](../uploads/farematrix.rb) that generates CSVs of all the three fare matrices. If you're too lazy to run it, here are links to the [aircon bus](../uploads/pub_aircon.csv), [ordinary bus](../uploads/pub_ordinary.csv) and [jeep](../uploads/puj.csv) fare matrices.

### GTFS compatibility

As is, the provided GTFS data does not have any fare data. I imagine this is because the existing spec doesn't have good support for distance-based fares like we have in the Philippines. Judging from the [fare examples](https://code.google.com/p/googletransitdatafeed/wiki/FareExamples), the only reasonable way we could implement distance-based fares is following example 6. This would involve setting a fare for each possible pair of stops based on the distance between them. This isn't exactly ideal. In fact, the people originally working on the DOTC project have voiced [issues](https://groups.google.com/forum/#!topic/gtfs-fare-wg/V63xRSnQJGw) and made [proposals](https://groups.google.com/forum/#!msg/gtfs-changes/uybrAokZ9Cg/rqlzXdMypUgJ) for having distance-based fares included into GTFS.

Apparently, public transit fares are a really complicated thing. You have fares based on distance, number of stops passed through, and transfers which may or may not cost extra. Not only that, you might have discounted fares, or first-class vs economy fares. The community will want to get it right before it's formally included in the spec. You can see the current state of the consolidated [GTFS fare proposal here](https://docs.google.com/document/d/1mK3--o5g4-3cCXaqmch92U63JTwChh0L2VCmcDViIlM/edit).

Even in it's proposal form though, we might have hope of being able to see these being used. There's currently a [pull request](https://github.com/OneBusAway/onebusaway-gtfs-modules/pull/30) for supporting the distance-based fare scheme into the OneBusAway libraries. The libraries actually used by GTFS Editor and OpenTripPlanner for working with GTFS data.

### Remaining Problems

Given all that, it would probably still be a long way before this allows us to make a really good routing app. We still don't have shape data, so the distance estimates would really be rough estimates at best. There's no support for rounding to the nearest centavo. I realize that's just nitpicking, but if we want something truly polished, even that has to be taken care of.

We also don't know if the jeeps or buses strictly follow the distance-based scheme. After all, if you can get on and off anywhere, you can't really measure distance that exactly. I assume they generally work off the notion of "zones" than actual distance travelled. In that sense, they work more similarly to the LRT which has fares based on how many stops you pass. For jeeps and buses, your fare is probably based more on how many "zones" you pass through.

### Conclusion

Philip, a co-worker of mine at By Implication, had suggested that we might want to use a different model than what the GTFS proposes. I have to agree with him. At this point, the GTFS doesn't really fit with our system. But I do think that open data and standards are great. In fact, I applaud the developers who made proposals for the fare system, as those are great first steps towards making the GTFS a more universal standard.

Side note: I'd also actually really like to hear about the DOTC developers' experience with the project. It would be nice if they had a devblog.