--- 
wordpress_id: 605
layout: post
title: simple webserver and rest with jetty and NO XML
excerpt: Simple no-frills implementation of a REST key-value store using jetty embedded and NO XML
date: 2011-02-16 13:57:40 +00:00
wordpress_url: http://caprazzi.net/?p=605
---
Starting a new web project (maybe with rest?) in Java requires mastery of many complex abstract components. You need a webapp 
for tomcat (fiddle with some xml files), spring (import plenty of jars and add some more xml files), pick a rest framework 
like spring, resteasy, restlset (fiddle with annotations), deploy to tomcat and go.

I won't deny all this components add structure, reusability and a lot of good stuff, but sometimes I have simple needs that 
scream for simple solutions.

Few days ago I needed just that: a simple database with a REST api, and the ability to serve a few static files.

I used jetty embedded and it took was one maven dependency, 3 classes, 150 lines of java, 15 minutes and NO XML.

I was so satisfied with the result work that I served myself a beer and [posted the code on 
github](https://github.com/mcaprari/simple-webserver-and-rest-with-jetty-and-no-xml). The actual project implemented a 
datastore on neo4j, but for sake of simplicity I released it on github as a dumb in-memory key-value store.

According to apache bench, with as little as 8m memory (-Xmx8m) the server was able to handle 2k concurrent users at a rate of 1500 requests per second.

The main is as simple as this:
{% highlight java%}
public static void main(String[] args) throws Exception {
	Server server = new Server(8080);
	Context root = new Context(server, "/");

	// configure the default servlet to serve static files from "htdocs"
	root.getInitParams().put("org.mortbay.jetty.servlet.Default.resourceBase", "htdocs");
	root.addServlet(new ServletHolder(new DefaultServlet()), "/");

	// use /uuid to get a fresh id
	root.addServlet(new ServletHolder(new UUIDServlet()), "/uuid");

	// the actual key/value store
	root.addServlet(new ServletHolder(new KeyValueServlet()), "/store/*");
	server.start();
}
{% endhighlight %}


Get the code: [Simple webserver and REST with jetty and NO XML on GitHub](https://github.com/mcaprari/simple-webserver-and-rest-with-jetty-and-no-xml)
