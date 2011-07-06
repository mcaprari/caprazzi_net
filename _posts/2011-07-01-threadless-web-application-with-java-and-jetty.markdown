---
layout: post
title: Threadless web app with java and jetty
date: 2011-06-30 19:00:39 +00:00
published: false
---

Traditionally, web servers and servlet containers allocate one thread for each 
incoming request: a request "owns" its thread and is assumed independent from 
other requests. 

One alternative way that has been gaining traction lately is to have one or
few threads to handle all the requests. 

While the first method is common place in java, the "threadless" method is 
less common; in this post I'll show how to implement such an application and 
try to understand when and why to prefer one over the other.

# Let's start with the tradition

This is a classic implementation of an HttpServlet ((full code here)[]):

{% highlight java %}
protected void doGet(request, response) { // simplified signature
	doSomeFileOperations();
	getSomeDataFromDb();
	doSomeComputations();
	reponse.getWriter().writeLine("done");
} 		
{% endhighlight %}

In this case, a thread will be dedicate to the execution of ``doGet()``, so it 
can take its time and block the thread with the peace of mind that the other 
requests will not be waiting for it.

This is true in an ideal world, where threads are inexpensive, there is 
spare capacity, and all shared resources can be accessed concurrently and 
without waiting.

In reality the CPU will only run a finite number of threads at any time and 
managing each thread will have a memory and processor cost and there will be 
shared resources that are not infinitely parallel.

On my mac, I have 4 cores, each thread takes on average 2ms to start and eats 
about 2k bytes of memory. Additionally, mac os has a hard limit of 2560 active 
threads, and the JVM goes belly up when it is reached. (I've written 
[TestMemory.java](https://gist.github.com/1059031) to get some rough numbers)

# But it works, right?

As it happens, using one thread per request works quite well and it is how 
most websites run today. Your other tomcat web application are perfectly fine.

Even if each core can execute one thread at a time, the number of "active" 
requests is usually greater then the sum of the CPU cores: if a thread is 
stopped waiting for a database to respond, it is still logically "advancing".

As long as each request completes "quickly enough" to keep the number of 
threads in check, a "threaded" server will support plenty of users for each 
available thread. 

# So why can't I just keep my threads?

There are several reason why you may not want to use one thread per request:

* Pherhaps the most commonIf the requests are long lasting, as it is in case of 
(comet)[http://en.wikipedia.org/wiki/Comet_(programming)]' and 
(http 
streaming/push)[http://en.wikipedia.org/wiki/Push_technology#HTTP_server_push]
, your application will need one thread for each connected user, even if the 
user is idle. With many users (and therefore many threads) the costs may 
become very high.

* You think that by precisely controlling the number of thrr

As we've seen using one thread to handle each connection is 

In some cases, such as "comet" or http streaming of data, the connections will 
be long lived, so you'll need a thread for each connection. 

In some cases the requests 
are designed to last quite some time, as it is in case of "comet" techniques, in other cases







