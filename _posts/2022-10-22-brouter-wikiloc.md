---
layout: single
title: BRoute, a better routing for Wikiloc
tags: [Wikiloc, OpenStreetMap]

header:
    teaser: /assets/img/wikiloc_router_1.png

---

Create hiking tracks with routing along trails with BRoute and use them in Wikiloc.

### Background  

At [van42.com](https://www.van42.com/) we want to keep a record of all our hikes so we did a quick comparison of  [AllTrails](https://www.alltrails.com/) and [Wikiloc](https://www.wikiloc.com/) as possible tools for the job.  

Playing around with AllTrails we soon realized that while it being a nice tool for users looking for hikes, is not that great for our purpose of keeping a "list of our hikes". This is because once a new trail is created, AllTrails takes a validation step where, for what we could tell, they group "similar" hikes under the same name. From that point on, it becomes almost impossible to distinguish between similar (but not identical) hikes done along the same trails.
On its side AllTrails' got a simple and easy to use editor capable of routing between waypoints along an existing trail on the map; if the waypoint is off trail then the editor defaults to link waypoints in a straight line.  

Testing Wikiloc on the other hand we found it just exactly right as a log book, but being designed to import GPS tracks more than drawing them by hand, its editor can't do much more than straight lines and is not routing-capable.  

![alt]({{ site.url }}{{ site.baseurl }}/assets/img/wikiloc_router_1.png)

### BRouter and Wikiloc

While I couldn't do much about how AllTrails' handle their catalogs, I could definitely find a better routing tool for Wikiloc, or code one myself (via [Leaflet.js](https://leafletjs.com/), [leaflet-routing](https://github.com/Turistforeningen/leaflet-routing) plugin and [OpenStreetMap API](https://wiki.openstreetmap.org/wiki/API)).  

In the end, it turned out that [BRouter](https://brouter.m11n.de/) does an awesome job at routing hikes (even if originally meant for bikes) and allows the export of tracks in GPX format that can be imported directly in Wikiloc.  

For all the nerds out there BRoute is a very active project that is open-sourced on [GitHub](https://github.com/abrensch/brouter). Technically speaking BRouter is a server-side routing engine (backend). Clients (frontend) can be found for Android all desktop platforms but the one linked above and that I use the most, is [brouter-web](https://github.com/nrenner/brouter-web) an online implementation built on top of the BRouter HTTP server.

![alt]({{ site.url }}{{ site.baseurl }}/assets/img/wikiloc_router_2.png)
