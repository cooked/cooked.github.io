---
layout: single
title: Leaflet maps in Jekyll
tags: [Leaflet, Jekyll, JavaScript]

header:
    teaser: https://raw.githubusercontent.com/Leaflet/Leaflet/main/docs/docs/images/logo.png

---

Add great looking maps to your static Jekyll webpage.

![alt]({{ site.url }}{{ site.baseurl }}/assets/img/leaflet-jekyll.png)

## About

Like the [previous article on hiking maps]({% post_url 2022-10-22-brouter-wikiloc %}), this blog post is also about a bit of code used at [van42.com](https://www.van42.com/).
It shows a map on the home page where it is possible to pinpoint all of our travel articles.  

A possible approaches I consider first was the [jekyll-leaflet](https://github.com/DavidJVitale/jekyll-leaflet) by David J. Vitale. After a very quick start I found out that the plugin was falling short for what I was looking for and the last active development is dated back in 2020.  
What I was after was, on top of adding map markers automatically from the posts loop, select a **custom icon** for the marker, **add a link to the marker** to open the post when clicked and have **gesture support** like in [Google Maps](https://developers.google.com/maps/documentation/javascript/examples/interaction-cooperative).

### Implement Leaflet.js

So, instead of relying on the plugin I first added the [Leaflet.js](https://leafletjs.com/) library to the Jekyll home layout (see code snippets below).

**IMPORTANT** - Place both CSS anf JS parts of the library in the <<head>> section of the HTML file, with the CSS before the JS.
{: .notice--warning}

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.2/dist/leaflet.css" integrity="sha256-sA+zWATbFveLLNqWO2gtiw3HL/lh1giY/Inf1BJ0z14=" crossorigin="" />
<script src="https://unpkg.com/leaflet@1.9.2/dist/leaflet.js" integrity="sha256-o9N1jGDZrf5tS+Ft4gbIK7mYMipq9lqpVJ91xHSyKhg=" crossorigin=""></script>
```

Then for the home content I've added the following code directly in the index.md to create the map, and to the site css file to style it.

```html
<div id="map"></div>

<script>
    var map = new L.Map("map", {
        center: new L.LatLng(44.4949, 11.3426),
        zoom: 2,
    });

    var layer = new L.StamenTileLayer("toner-lite");
    map.addLayer(layer);
</script>
```

```css
#map {
    width: 100%;
    height: 300px;
}
```

**IMPORTANT** - The map needs a fixed **height** for the tiles to be sized and displayed correctly. The width, on the other hand, can be variable. 
{: .notice--warning}

### Custom markers loop

The markers on the map are created using the [Liquid](https://shopify.github.io/liquid/) snippet shown in the [jekyll-leaflet Getting Started page](https://davidjvitale.com/tech/jekyll-leaflet/getting-started/)

```js
{% raw  %}
{%- for post in site.posts -%}
    {% if post.location.latitude and post.location.longitude %}
        L.marker([ {{post.location.latitude}}, {{post.location.longitude}} ], { title: '{{post.title}}'}).addTo(map);
    {% endif %}
{% endfor %}
{% endraw  %}
```

### Marker link

Now to add an onclick behavior to open the linked post, the marker line inside the loop as to be modified as follows:

```js
{% raw  %}
L.marker([ {{post.location.latitude}}, {{post.location.longitude}} ], { title: '{{post.title}}'}).addTo(map).on('click', function(e) { window.open('{{post.url | relative_url}}','_self'); })
{% endraw  %}
```

In a bit convoluted way the code above adds an *on-click* event to the marker that will open the link in a new window (in this case current window since we specify "_self"). The url to link to is pulled in from the post and made "relative" again via Liquid

### Map gestures

Gestures are added via the [Leaflet.GestureHandling](https://github.com/elmarquis/Leaflet.GestureHandling) but currently are NOT WORKING for me.  
Import the plugin just after the Leaflet library

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet-gesture-handling@1.1.8/dist/leaflet-gesture-handling.min.css">
<script src="https://unpkg.com/leaflet-gesture-handling@1.1.8/dist/leaflet-gesture-handling.js"></script>
```

and enable it after the map has been created:

```js
map.gestureHandling.enable();
```
