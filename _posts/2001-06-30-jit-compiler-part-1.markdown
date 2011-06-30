---
layout: post
title: What would the JIT compiler do?
date: 2011-06-30 18:00:39 +00:00
hide: true
excerpt: | an exploration of the Java JIT compiler and its effect on performace
---

Most java VMs have a built in Just In Time compiler ("the JIT")

In short, the JIT analyzes the behviour of a program while it runs and finds opportunities to optimize the bytecode and re-compile it to machine code using the most appropriate instruction set. All mainstream java VMs have a JIT compiler and you would not want to use one without.

This is awesome and magical and very opaque and the gains will be different on different machines, VMs and configurations. The JIT is on by default, but can be disabled using  ``-Djava.compiler=none``

Because it is opaque and unpredictable, it is a bad idea to rely on the JIT for the performance of a program. Still, it is interesting to see it at work.

## But is it faster?

Let's first work out if this JIT optimization works at all.

I just added ``-Djava.compiler=none`` to my eclipse.ini and restarted it. Itfeels slower on "big operations" like clean & rebuild all or searching for the string "this" in all java files in the workspace. 

That's good but very difficult to measure.

This is simple class that does some float multiplcations

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
{% endhighlight %}

And this is a simple script that executes the code alternatively with the JIT ON and OFF:
{% highlight bash %}
COUNT=50000000
for i in $(seq 5); do
	echo $i"-----"
	java -cp bin net.caprazzi.WWJD $COUNT ON
	java -Djava.compiler=none -cp bin net.caprazzi.WWJD $COUNT OFF
done
{% endhighlight %}

It's immediately clear that with the JIT on, the execution is reliably 50 times faster: 
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

Cool. Let's change the code a bit and try again:

{% highlight java %}
float[] floats = new float[count];
float f = 1f;
for (int i=1; i < count; i++) {
	f *= i;
	floats[i] = f;
}
{% endhighlight %}

<br/>

<pre class="terminal">
Run 50000000 times with JIT ON...       completed in 102871363 nanoseconds
Run 50000000 times with JIT OFF...      completed in 9154895167 nanoseconds
</pre>

Sweet, here the optimized version is 80 times faster.

## Always always faster faster?

This code was tested on an laptop with an intel multicore processor and
windows 7 (java version: <s>sun</s> oracle 6.0.26, client VM, 32 bit)

Let's see what happens when we run it on

* A scruffy linux virtual box, java OpenJDK 6 client VM 32 bit 6.0.20

<pre class="terminal">
Run 50000000 times with JIT ON...       completed in 17990143548 nanoseconds
Run 50000000 times with JIT OFF...      completed in 27172812381 nanoseconds
</pre>
Boooo - that's just 1.5 times faster! 

* A different linux virtual box, java oracle 6.0.20, server VM, 64 bit

<pre class="terminal">
Run 50000000 times with JIT ON...       completed in 305513000 nanoseconds
Run 50000000 times with JIT OFF...      completed in 2135685000 nanoseconds
</pre>

Mhhhh - 7 times faster.

* A 1997 Mac Pro, oracle 6.0.24, server VM, 64 bit

<pre class="terminal">
Run 50000000 times with JIT ON...       completed in 435386000 nanoseconds
Run 50000000 times with JIT OFF...      completed in 1844200000 nanoseconds	
</pre>

That's 4 times faster.

Ok, so there is a definite but very variable performance loss in handling floats and array when turning off the JIT.

## Let's try with strings

{% highlight java %}
String s = "0";
for (int i=0; i < count; i++) {
	s += "0";
}
s.charAt(0);
{% endhighlight %}

* Windows 7, intel cpu, java client VM 6.0.26: **no difference**

<pre class="terminal">
Run 50000 times with JIT ON...  completed in 1859603439 nanoseconds
Run 50000 times with JIT OFF... completed in 1832164432 nanoseconds
</pre>

* linux virtual box, java 6.0.20, server vm 64 bit, **1.4x**

<pre class="terminal">
Run 50000 times with JIT ON...  completed in 1362850000 nanoseconds
Run 50000 times with JIT OFF... completed in 2003949000 nanoseconds
</pre>

Magic magic magic

## The JIT can speak

``-XX:-PrintCompilation`` is option to the Oracle's VM supports that makes the jit output some information on what it compiles a method. This allows to see how the JIT intervenes differently on each run of the same program.

If we execute our test 1000 times, we see no output:
<pre class="terminal">
$ java -XX:+PrintCompilation -cp bin net.caprazzi.WWJD 1 ON
Run 1 times with JIT ON...      completed in 23000 nanoseconds	
</pre>

I've incremented the size of the test loop until at 3250 iterations we start seeing some output:
<pre class="terminal">
$ java -XX:+PrintCompilation -cp bin net.caprazzi.WWJD 4000 ON
Run 4000 times with JIT ON...   ---   n   java.lang.System::arraycopy (static)
completed in 40601000 nanoseconds
</pre>

At 4500 iteration, more output:
<pre class="terminal">
	n   java.lang.System::arraycopy (static)
    1	java.lang.Object::<init> (1 bytes)
</pre>

At 5000:

<pre class="terminal">
	n   java.lang.System::arraycopy (static)
	1       java.lang.Object::<init> (1 bytes)
	2       java.lang.String::getChars (66 bytes)
	3       java.lang.AbstractStringBuilder::append (60 bytes)
	4       java.lang.StringBuilder::append (8 bytes)
</pre>

10000:

<pre class="terminal">
	n   java.lang.System::arraycopy (static)
	1       java.lang.Object::<init> (1 bytes)
	2       java.lang.String::getChars (66 bytes)
	3       java.lang.AbstractStringBuilder::append (60 bytes)
	4       java.lang.StringBuilder::append (8 bytes)
	5       java.lang.Math::min (11 bytes)
	6       java.lang.String::<init> (72 bytes)
	7       java.util.Arrays::copyOfRange (63 bytes)
	8       java.lang.AbstractStringBuilder::<init> (12 bytes)
	9       java.lang.StringBuilder::toString (17 bytes)
	10       java.lang.String::valueOf (14 bytes)
	11       java.lang.StringBuilder::<init> (18 bytes)
</pre>

And finally, at 15000 the most verbose yet:

<pre class="terminal">
	n   java.lang.System::arraycopy (static)
	1       java.lang.Object::<init> (1 bytes)
	2       java.lang.String::getChars (66 bytes)
	3       java.lang.AbstractStringBuilder::append (60 bytes)
	4       java.lang.StringBuilder::append (8 bytes)
	5       java.lang.Math::min (11 bytes)
	6       java.lang.String::<init> (72 bytes)
	7       java.util.Arrays::copyOfRange (63 bytes)
	8       java.lang.AbstractStringBuilder::<init> (12 bytes)
	9       java.lang.StringBuilder::toString (17 bytes)
	10       java.lang.String::valueOf (14 bytes)
	11       java.lang.StringBuilder::<init> (18 bytes)
	12       java.lang.String::toString (2 bytes)
	1%      net.caprazzi.WWJD::main @ 59 (133 bytes)
</pre>

# But what is it trying to say?

I didn't find any explaination on the output generated by -XX:+PrintCompilation, but from what I can understand, each line shows a method that has been compiled to native code and a progressive id. The first and last line are a bit of a puzzle, showing 'n' and '1%'

