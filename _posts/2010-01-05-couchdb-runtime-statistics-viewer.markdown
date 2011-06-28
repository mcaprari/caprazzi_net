--- 
wordpress_id: 440
layout: post
title: Couchdb runtime statistics viewer
date: 2010-01-05 14:16:49 +00:00
wordpress_url: http://caprazzi.net/?p=440
---

CouchDB comes with a runtime statistics module that lets you inspect how CouchDB performs.The statistics module collects metrics like requests per second, request sizes and a multitude of other useful stuff. _From [Couchdb wiki](http://wiki.apache.org/couchdb/Runtime_Statistics)_.I created a simple app that uses [chronoscope](http://timepedia.org/chronoscope/ "timepedia chronoscope") to display how some metrics change over time.Jump to the [demo](http://caprazzi.net:5984/stats/_design/app/_list/table_multi/http_requests_hits_by_method?group_level=4 "couchdb runtime stats viewer"), go to the [source](http://github.com/mcaprari/couchdb-stats "couchdb runtime stats source") or use the [docs](http://caprazzi.net:5984/stats/_design/app/docs/index.html "couchdb runtime stats documentation").[![Couchdb Stats Screenshot](http://caprazzi.net/wp-content/uploads/2010/01/couchdb_stats_screenshot.png)](http://caprazzi.net:5984/stats/_design/app/_show/index/bogus "couchdb runtime stats viewer")
## Install

couchdb-stats is a simple couchapp plus a python script that hits /_stats and stores the results in couchdb server_Attention: this app is tested with Couchdb 0.11.x, not yet released at the time of this post._
<pre class="term">$ git clone git://github.com/mcaprari/couchdb-stats.git$ cd couchdb-stats$ couchapp push app http://localhost:5984/stats$ python stats_copy.py localhost localhost stats 60</pre>
Couchdb reports metrics since server start or of the last 1, 5 or 15 minutes. There is no way to see yesterday's stats. To do that, we need to keep hitting /_stats and store results in a couchdb database.[stats_copy.py](http://github.com/mcaprari/couchdb-stats/blob/master/stats_copy.py "python couchdb _stats  consumer") does just that.

At this point it's possible to write several views, each focused on a particular metric. See the [documentation](http://caprazzi.net:5984/stats/_design/app/docs/index.html "couchdb runtime stats documentation") for more details.**questions, comments and suggestions welcome**
