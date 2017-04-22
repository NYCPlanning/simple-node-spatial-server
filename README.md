# simple-node-spatial-server
Roll your own spatial data API using PostGIS, Node.js &amp; Express

## Why
This repo is a place to document the Free and Open Source spatial data API stack we have been tinkering with here at DCP.  Someday it will include a simple working example, but for now it's just a readme with some details about the stack.

## Summary
The goal was to build simple web API that could run [spatial queries](https://en.wikipedia.org/wiki/Spatial_query) and return [GeoJSON](https://en.wikipedia.org/wiki/GeoJSON) Features to the browser .  This is acheivable in with a stunningly small amount of code thanks to the many helper packages in the Node.js ecosystem.  This stack is used under the hood in [PostGIS Preview](https://github.com/NYCPlanning/postgis-preview), among other projects here at DCP.

![presentation1](https://cloud.githubusercontent.com/assets/1833820/17250362/ff5c514e-5571-11e6-8d3a-b2526fcada24.png)


##Database
[PostgreSQL](https://www.postgresql.org/) is an open source relational database, and [PostGIS](http://postgis.net/) is an extension that allows you to have spatial datatypes in addition to the usual numbers, text, dates, etc.  This means you can store vector geometries (like those usually seen in shapefiles in the Desktop GIS world) in the database, and run spatial queries on them (to do things like count points that fall within a polygon, determine nearest neighbors, locate centroids, etc)

There are [installers and packages](http://postgis.net/install/) for PostGIS for Windows, MacOS, and Linux.  Once you have it installed and its service is running, the simplest way to load spatial data is using the command line tool ['shp2pgsql'](http://www.bostongis.com/pgsql2shp_shp2pgsql_quickguide.bqg) to load a shapefile.  

## Node
[Node](https://nodejs.org/en/) allows you to run javascript code on the server.  For this stack, all of logic that brokers information between the client and the database is running in a node.js app.  

## Express
[Express](https://expressjs.com/) allows you to run a webserver and quickly expose endpoints.  
  
Here is the express 'Hello World!' example that shows how simple it is to spin up a web server and expose a route at the root path.
  
```
var express = require('express');
var app = express();

app.get('/', function (req, res) {
  res.send('Hello World!');
});

app.listen(3000, function () {
  console.log('Example app listening on port 3000!');
});
```

To create an API endpoint, the process is the same, but instead of sending HTML to the client you send JSON data.  

## pg-promise
The [pg-promise](https://github.com/vitaly-t/pg-promise) package for node handles communication with the PostGIS database, and allows you to send SQL queries through a simple API, like:
```
db.query('select * from users where name=${name} and active=$/active/', {
    name: 'John',
    active: true
});
```
It wraps everything into promises, so you can do a `.then()` to handle the response from the database.  pg-promise doesn't need to know that it's talking to a spatial database, it simply passes your SQL query along and gives you back the data.

## dbgeo and WKB
When you request data from PostGIS, the geometry columns are returned in a format called Well-Known-Binary, or WKB.  This allows them to be efficiently stored and transferred as text strings, but will need to be converted into another format before being sent to the browser.  
For example, here is a point geometry in WKB:
```
0101000020E6100000AC9CF1D4577D52C09EC5E21C2F5E4440
```
When converted to geojson, it looks like this:
```
{"type":"Point","coordinates":[-73.9584858283512,40.7358127696309]}
```
The `dbgeo` node package transforms spatial data between various formats, and can easily take the database response from `pg-promise` and convert it into a valid GeoJSON feature.  Then this is passed back to express, to be passed back to the client, where it can be mapped using Leaflet or any other frontend library that supports geojson!

### Feedback
We would love to hear your comments or suggestions about this stack.
