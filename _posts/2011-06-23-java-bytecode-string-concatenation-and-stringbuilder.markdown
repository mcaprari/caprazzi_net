--- 
wordpress_id: 684
layout: post
title: Java bytecode, string concatenation and StringBuilder
date: 2011-06-23 10:08:43 +00:00
wordpress_url: http://caprazzi.net/?p=684
---
In my [earlier post](http://caprazzi.net/posts/evaluating-relative-speed-of-java-digest-hashing-algorithms/) I was making a fuss over picking the faster hash algorithm, when I realised I was using + to concatenate strings. Should I always use a [StringBuilder](http://download.oracle.com/javase/1.5.0/docs/api/java/lang/StringBuilder.html)? Should I care even for small strings? Heck, if I use the ``StringBuilder`` I'll surely create one extra object anyway...

I tried some variations of the test and I did not find any performance difference when comparing simple concatenation to using the string builder. Even if trying with bigger strings and other combinations, I could see no difference. I got curious and wrote a very simple class and looked resulting bytecode: This java code:

{% highlight java%}
{% endhighlight %}

{% highlight java%}
public static void main(String[] args) {
	String cip = "cip";
	String ciop = "ciop";
	String plus = cip + ciop;
	String build = new StringBuilder(cip).append(ciop).toString();
}
{% endhighlight %}

Generates this bytecode (see how the two concatenation styles generate the very same code):
{% highlight java%}
 L0
    LINENUMBER 23 L0
    LDC "cip"
    ASTORE 1
   L1
    LINENUMBER 24 L1
    LDC "ciop"
    ASTORE 2
// cip + ciop
   L2
    LINENUMBER 25 L2

    NEW java/lang/StringBuilder
    DUP
    ALOAD 1
    INVOKESTATIC java/lang/String.valueOf(Ljava/lang/Object;)Ljava/lang/String;
    INVOKESPECIAL java/lang/StringBuilder.<init>(Ljava/lang/String;)V
    ALOAD 2
    INVOKEVIRTUAL java/lang/StringBuilder.append(Ljava/lang/String;)Ljava/lang/StringBuilder;
    INVOKEVIRTUAL java/lang/StringBuilder.toString()Ljava/lang/String;

    ASTORE 3
// new StringBuilder(cip).append(ciop).toString()
   L3
    LINENUMBER 26 L3

    NEW java/lang/StringBuilder
    DUP
    ALOAD 1
    INVOKESPECIAL java/lang/StringBuilder.<init>(Ljava/lang/String;)V
    ALOAD 2
    INVOKEVIRTUAL java/lang/StringBuilder.append(Ljava/lang/String;)Ljava/lang/StringBuilder;
    INVOKEVIRTUAL java/lang/StringBuilder.toString()Ljava/lang/String;

    ASTORE 4
   L4
    LINENUMBER 27 L4
    RETURN
{% endhighlight %}

The compiler has transformed "``cip+ciop``" into "``new StringBuilder(cip).append(ciop).toString()``". In other words, "+" is effectively a shorthand for the more verbose ``StringBuilder`` idiom.

The compiler will do same trick for ``cip + "ciop"`` and ``"cip" + ciop``. (In case you wonder, ``"cip" + "ciop"`` will just be compiled as ``"cipciop"``).

This is great, but beware, the compiler is not a worthy substitute for you thinking at what you do. 

This code
{% highlight java%}
String big = "both";
big += cip;
big += ciop;
{% endhighlight %}

Will be compiled into
{% highlight java%}
String big = "both";
big = new StringBuilder(bag).append(cip).toString();
big = new StringBuilder(bag).append(ciop).toString();
{% endhighlight %}

While of course the most efficient way is 
{% highlight java%}
String big = new StringBuilder("both").append(cip).append(ciop).toString();
{% endhighlight %}

Now of course nobody in his right mind would ever write any of the above, but here is more common pattern:

{% highlight java%}
String boo = "both";
for (int i=1; i<100; i++)
     boo += cip + ciop;
{% endhighlight %}

Now the compiler will do the obvious thing and instantiate one new StringBuilder at each iteration:
{% highlight java%}
String boo = "both";
for (int i=1; i<100; i++)
     boo += new StringBuilder(boo).append(cip).append(ciop).toString();
{% endhighlight %}

In this case it is best to use this idiom:
{% highlight java%}
StringBuilder foo = new StringBuilder("both");
for (int i=1; i<2; i++)
    foo.append(cip).append(ciop);
String boo = foo.toString();
{% endhighlight %}

``-teo``
