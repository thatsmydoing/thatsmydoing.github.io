<!-- 
.. link: 
.. description: 
.. tags: philippine-transit-app, programming
.. date: 2013/09/25 12:26:59
.. title: Geocoding Services
.. slug: geocoding-services
-->

A key component for any routing service is being able to do geocoding. Most people who are looking for routes most probably don't know exactly where their start and end points are on the map. Even then, manually looking for a location on a map is a time-consuming task.

The gold standard for doing geocoding right now is Google Maps. It's hard to find a better location search experience. If they actually provided routing for jeeps here in the Philippines, I imagine there wouldn't be *that* much you could do for the competition.

When the competition started though, I took it as a challenge to avoid Google Maps as much as possible. I wanted to see how much is currently possible with other options such as OpenStreetMap. In fact, OSM does have a geocoding service called [Nominatim](http://nominatim.openstreetmap.org).

Sadly, for a mapping app, what you want to do is not simply just geocoding. With geocoding, you take an address and turn it into coordinates. When you want to search for a place in a mapping app, you take part of an address, infer the rest of it, and give the user options to choose from.

Given a typical mapping app, you might type in "ateneo" and expect it to give you Ateneo de Manila University. With typical geocoding services like Nominatim or even Google's [geocoding API](https://developers.google.com/maps/documentation/javascript/geocoding), you probably won't get any result for this. What you want to use is the [Places API](https://developers.google.com/maps/documentation/javascript/places) which provides an autocomplete search box. Using it, when you type in "ateneo", it automatically suggests in the dropdown, "Ateneo de Manila University".

A downside to using the Places API is that it's against the terms of service to use it with something that isn't Google Maps, which means no OpenStreetMap. If there were more time, writing your own autocompletion engine using OpenStreetMap's data will probably be a better long term solution.

For now, since the competition's deadline is just a few days away, I'll be using Google Maps.
