---
layout: post
title: What would the JIT compiler do?
date: 2011-06-30 18:00:39 +00:00
published: false
excerpt: | an exploration of the Java JIT compiler and its effect on performace
---

After reading a comment to a previous post, I realised that I know next to 
nothing about the java JIT compiler ("the JIT"), and yet I use it all the times. 

In short, the JIT analyzes the behviour or a program at runtime and finds opportunities to optimize the bytecode and re-compile it to machine code. This is awesome and magical and very opaque and may have different results on different machines (and maybe each time on the same machine depending on load or the environment in general?). The JIT is on by default, but can be disabled using  ``-Djava.compiler=none``

Because it is opaque and unpredictable, it is a bad idea to rely on the JIT for the performance of a program. But how do I know that my program is not actually going very fast on my enviroment  thanks to the JIT and will not perform badly on the target machine? Worse even, what if when deployed the program goes from acceptably fast to unacceptably slow?

## But is it faster?

Let's first work out if this JIT optimization works at all.

A simple class that does some float multiplcations
{% highlight java %}
public static void main(String[] args) {		
	int  count = Integer.parseInt(args[0]);		
	System.out.print("Run " + count + " times with JIT " + args[1] + "...\t");		
	long start = System.nanoTime();
	
	float f = 1f;
	for (int i=1; i<count; i++)
		f *= i;
	
	long end = System.nanoTime();										
	System.out.println("completed in " + (end-start) + " nanoseconds ");		
}	
{% endhightlight %}

And a simple script that executes that executes the code alternatively with the JIT on and OFF:
{% highlight bash %}
COUNT=50000000
for i in $(seq 5); do
	echo $i"-----"
	java -cp bin net.caprazzi.WWJD $COUNT ON
	java -Djava.compiler=none -cp bin net.caprazzi.WWJD $COUNT OFF
done
{% endhighlight %}

At the first execution it's clear that there with the JIT on, the execution is reliably 50 times faster: 
<pre class="terminal">
$ sh test.sh
1-----
Run 50000000 times with JIT ON...       completed in 166548543 nanoseconds
Run 50000000 times with JIT OFF...      completed in 8765290860 nanoseconds
2-----
Run 50000000 times with JIT ON...       completed in 161432520 nanoseconds
Run 50000000 times with JIT OFF...      completed in 8775846265 nanoseconds
3-----
Run 50000000 times with JIT ON...       completed in 180772121 nanoseconds
Run 50000000 times with JIT OFF...      completed in 9166550254 nanoseconds
4-----
Run 50000000 times with JIT ON...       completed in 152595716 nanoseconds
Run 50000000 times with JIT OFF...      completed in 8824074862 nanoseconds
5-----
Run 50000000 times with JIT ON...       completed in 157337479 nanoseconds
Run 50000000 times with JIT OFF...      completed in 8738365122 nanoseconds
</pre>

Cool. Let's make the code a bit more interesting and try again:

{% highlight java %}
float[] floats = new float[count];
float f = 1f;
for (int i=1; i < count; i++) {
	f *= i;
	floats[i] = f;
}
{% endhighlight %}

<pre class="terminal">
Run 50000000 times with JIT ON...       completed in 102871363 nanoseconds
Run 50000000 times with JIT OFF...      completed in 9154895167 nanoseconds
</pre>

Sweet, here the optimized version is 80 times faster.

## Always always faster?

This code was tested on an laptop with an intel multicore processor and
windows vista (java version: <s>sun</s> oracle 6.0.26, client VM, 32 bit)

Let's see what happens when we run it on

### - a scruffy linux virtual machine (as in "vmware") (OpenJDK 6 client VM 32 bit 6.0.20)

<pre class="terminal">
Run 50000000 times with JIT ON...       completed in 17990143548 nanoseconds
Run 50000000 times with JIT OFF...      completed in 27172812381 nanoseconds
</pre>
Boooo - that's just 1.5 times faster! 

### - a different linux virtual machine (oracle 6.0.20, server VM, 64 bit)
<pre class="terminal">
Run 50000000 times with JIT ON...       completed in 305513000 nanoseconds
Run 50000000 times with JIT OFF...      completed in 2135685000 nanoseconds
</pre>

Mhhhh - 7 times faster.

### - 1997 Mac Pro, oracle 6.0.24, server VM, 64 bit
<pre class="terminal">
Run 50000000 times with JIT ON...       completed in 435386000 nanoseconds
Run 50000000 times with JIT OFF...      completed in 1844200000 nanoseconds	
</pre>

4 times faster with JID





Ok, so there is a definite but very variable performance gain when using the JIT. To be fair with the

Ok we get that, but what kind of optimization is going on here? How is the time being saved? I have no idea.

-Djava.compiler=none

-XX:-PrintCompilation

-XX:CompileThreshold=10000

-XX:+AggressiveOpts

-XX:+UseCompressedStrings	Use a byte[] for Strings which can be represented as pure ASCII. (Introduced in Java 6 Update 21 Performance Release) 

-XX:+OptimizeStringConcat	Optimize String concatenation operations where possible. (Introduced in Java 6 Update 20) 

-XX:-CITime