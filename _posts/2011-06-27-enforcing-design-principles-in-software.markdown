--- 
wordpress_id: 707
layout: post
title: Enforcing design principles in software
date: 2011-06-27 17:33:31 +00:00
wordpress_url: http://caprazzi.net/?p=707
---

Before starting to write a new piece of software, I come up with some simple design principles that I think will do some good.
For smallish projects I just keep the principles in mind and behave so, but for bigger stuff I try to actively enforce the policy.


<blockquote>"In theory, theory and practice are the same. In practice, they are not."
<br/>
    --  Lawrence Peter Berra</blockquote>

Actively "defending" your code from your own mistakes is controversial;
In theory if you state your principle and you stick to it, there will be no need for enforcement. A [friend of mine](http://blog.acaro.org/) **has never written a single bug in his life.** It's true I swear. He clearly can go without a safety net.

Another friend added that proper code testing would make policy enforcement useless.
 
I actually do bugs sometimes, and my tests could be leaky sometimes, so I benefit from having some safety checks. Actively checking rules makes it difficult to take shortcuts like "I'll fix this later" or "this will never be null right?". Well established check policies, much like widely used vaccines, have the potential of eradicating some categories of bugs. 

## No country for nulls

The principle I use the most is *nulls are no good*. Ideally one would change the language and remove the concept of nullability, avidoing the need for nullpointer exceptions (NPE) and nullchecks. NPE are often a pain because a they tend to happen far from where the assignment has happened.

To enforce a principle, you have to make up a rule and implement it in your code. At first the best rule to implement this principle may seem "``always check for nullity``", but I'll try to show that in this case, just half of the rule, "``check for nullity before assigning``" is just enough to cover your ass and is less verbose.

Let’s begin with an example - everybody likes the bang of a popping balloon, so I have implemented a balloons collection that allows to pop many balloons at once for a bigger, better bang.

{% highlight java%}
class Balloons {
	List<Balloon> balloons = new ArrayList<Balloon>();
	void addBalloon(Balloon balloon) {
		this.balloons.add(balloon);
	}
	void pinchAll() {
		for (Balloon b : balloons)
                   b.pinch();
	}
	public static void main(String[] args) {
		Balloons balloons = new Balloons();
		balloons.addBalloon(new Balloon());
		balloons.addBalloon(null); // one of your little helpers has run out of gas
		balloons.pinchAll();
	}
}
{% endhighlight %}

Clearly, running this class will cause a NPE and the program to terminate. Unfortunately the stack trace will not clarify who added that null:


    Exception in thread "main" java.lang.NullPointerException
    at Balloons.pinchAll(Balloons.java:16)at Balloons.main(Balloons.java:23)

To get rid of the NPE, you can just add a nullcheck inside pinchAll.

{% highlight java%}
void pinchAll() {
	for (Balloon b : balloons)
		if (b != null)
			b.pinch();	
}
{% endhighlight %}

This version will avoid the NPE, sure, but if you ignore null values, **you’re better off not putting them in your collection in the first place**, right? So let’s try this:

{% highlight java%}
void addBalloon(Balloon balloon) {
	if (balloon == null)
		trow new IllegalArgumentException(“balloon must not be null”);
	this.balloons.add(balloon);
}
{% endhighlight %}

Way better: this time the output will actually help to pin down the cause of the problem

	Exception in thread "main" java.lang.IllegalArgumentException: balloon must not be null
	at Balloons.addBalloon(Balloons.java:10)at Balloons.main(Balloons.java:22)

Additionally, since you have protected your code so that balloons will never contain a null, you can remove the check in pinchAll.At the bottom of the post I've attached snippets of my "Protect" class, that helps reducing the bloat of enforcing the policy.Some examples:

{% highlight java%}
void addBalloon(Balloon balloon) {
	Protect.valid(balloon);
	this.balloons.add(balloon);
}

void addMany(Balloon one, Balloon two) {
	Protect.valid(one, two);
        //....
}

void addWithLabel(String label, Balloon balloon) {
    // this will also check for empty strings
    Protect.valid(label, balloon);
    //....
}

{% endhighlight %}

I have picked up the habit of testing all arguments with Protect.notNull, and _then_ think if I really need it or if this code is implicitly protected by it call hierarchy. Yes, it’s bulky and your lean and clean code will look bloated, but mind that you'll save a lot of ``!=null`` checks after invoking methods. It is also useful as a form of “active documentation” as any reader will quickly get that nulls are not accepted.

Null protection is probably at its best when used at the constructor of an immutable class: you _know_ that all instances of that class do not contain nulls

{% highlight java%}
// rock solid bunga bunga!!
public class BungaBunga {
	private final String secretLocation;
	public BungaBunga(String secretLocation) {
		Protect.valid(secretLocation);
		this.secretLocation= secretLocation;
	}

	// ....
}
{% endhighlight %}


Beware, testing for nullity will take some time - on my laptop about a ns for the simple check (object == null) and maybe 10 for the version with varargs. It’s not much but they may add up, so in you may want to skip the checks in some regions of your codes, especially method invocations in loops. As most things in life, don't overdo it.

It's possible to add more complex checks or other kinds. In a recent app that was using some simple spatial geometry I enforced all the ints to be positive, because I knew there was no space for negative values in the app.

If you want to get rid of the checks before release to production, it's quite easy to comment out all the invocations with a script, but this may be brittle and make debugging the code a bit more complex. If you are into this kind if things, it is possible to enforce rules transparently using AspectJ and some annotations. This has the benefit of making the code cleaner, but you loose the "documentation" side of the checks.

[Note: in C# it’s easy to compile methods only in debug builds, using [Conditional("DEBUG")] ]

My Protect class. As you can see it is a very crude piece of code, but it does the job.

{% highlight java%}
/**
 * Set of methods to enforce not nullity of objects and not emptiness of strings
 * @author Matteo Caprari
 */
public class Protect {

	static void valid(Object object) {
		if (object == null)
			throw new IllegalArgumentException("Object in position 0 is null");
	}		

	static void valid(String string) {
		if (string == null || string.trim().length() == 0)
			throw new IllegalArgumentException("String in position 0 is null or empty");
	}

	static void valid(String a, String b) {
		if (a == null || a.trim().length() == 0)
			throw new IllegalArgumentException("String in position 0 is null or empty");
		if (b == null || b.trim().length() == 0)
			throw new IllegalArgumentException("String in position 1 is null or empty");
	}

	static void valid(String a, String b, String c) {
		if (a == null || a.trim().length() == 0)
			throw new IllegalArgumentException("String in position 0 is null or empty");
		if (b == null || b.trim().length() == 0)
			throw new IllegalArgumentException("String in position 1 is null or empty");
		if (c == null || c.trim().length() == 0)
			throw new IllegalArgumentException("String in position 2 is null or empty");
	}

	static void valid(String... strings) {
		for (int i=0; i<strings.length; i++) {
			if (strings[i] == null || strings[i].trim().length() == 0)
				throw new IllegalArgumentException("String in position " + i + " is null or empty");
		}
	}

	static void valid(Object a, Object b) {
		if (a == null)
			throw new IllegalArgumentException("Object in position 0 is null or empty");
		if (b == null)
			throw new IllegalArgumentException("Object in position 1 is null or empty");

		if (a instanceof String && ((String) a).trim().length() == 0)
			throw new IllegalArgumentException("String in position 0 is null or empty");

		if (b instanceof String && ((String) b).trim().length() == 0)
			throw new IllegalArgumentException("String in position 1 is null or empty");
	}

	static void valid(Object a, Object b, Object c) {
		if (a == null)
			throw new IllegalArgumentException("Object in position 0 is null or empty");
		if (b == null)
			throw new IllegalArgumentException("Object in position 1 is null or empty");
		if (c == null)
			throw new IllegalArgumentException("Object in position 2 is null or empty");

		if (a instanceof String && ((String) a).trim().length() == 0)
			throw new IllegalArgumentException("String in position 0 is null or empty");

		if (b instanceof String && ((String) b).trim().length() == 0)
			throw new IllegalArgumentException("String in position 1 is null or empty");

		if (c instanceof String && ((String) c).trim().length() == 0)
			throw new IllegalArgumentException("String in position 3 is null or empty");
	}

	// beware, this is maybe 10 times slower than the non-varargs version
	static void valid(Object... objects) {
		for (int i=0; i<objects.length; i++) {
			if (objects[i] == null)
				throw new IllegalArgumentException("Object in position " + i + " is null");
			if (objects[i] instanceof String && ((String) objects[i]).trim().length() == 0)
				throw new IllegalArgumentException("String in position " + i + " is null or empty");
		}
	}				

}

{% endhighlight %}


Thanks for reading this far
