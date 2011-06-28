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
in theory if you stick to a principle, there is no need for enforcement. A [friend of mine](http://blog.acaro.org/) **has never written a single bug in his life.** (It's true, I swear). He clearly can go without a safety net.

Another friend added that proper code testing makes enforcement useless.

I hate to admit it, but I do do bugs sometimes, and some tests I wrote turned 
out to be not so great. I've learnt that I'm a better coder 
when there are some safety checks. One immediate benefit is that checking 
rules makes me less likely to cut corners "just this once". 

More deeply, I find that established check policies, much like widely used vaccines, have the potential to eradicate some categories of bugs. 

## No country for nulls

The principle I use most frequently is *nulls are no good*. Nulls are the 
source of Null Pointer Exceptions (NPE) and null checks. NPEs are a pain
because they tend to happen far from where the assignment has happened, while 
null checks are just plain ugly. *If I could, I would change the language and 
remove the concept of nullability*

Failing that, I have to turn the principle in a rule and implement it in the 
code. At first the best rule may seem "``always check for nullity``", but 
I'll show that in this case it is enough to implement half of the rule 
to cover your ass: "never assign a null"

## Never assign a null

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

Not only this program will end in NPE, but also the stack trace will not clarify were the null comes from:

    Exception in thread "main" java.lang.NullPointerException
    at Balloons.pinchAll(Balloons.java:16)
	at Balloons.main(Balloons.java:23)

To get rid of the NPE, you can just add a nullcheck inside pinchAll.

{% highlight java%}
void pinchAll() {
	for (Balloon b : balloons)
		if (b != null)
			b.pinch();	
}
{% endhighlight %}

True, this version avoids the exception, but if you ignore null values, *you’re better off not putting them in your collection in the first place*, yes? 

Let’s try again:
{% highlight java%}
void addBalloon(Balloon balloon) {
	if (balloon == null)
		trow new IllegalArgumentException(“balloon must not be null”);
	this.balloons.add(balloon);
}
{% endhighlight %}

Much better now: the output will actually help to pin down the cause of the problem.

	Exception in thread "main" java.lang.IllegalArgumentException: 
		balloon must not be null
	at Balloons.addBalloon(Balloons.java:10)
	at Balloons.main(Balloons.java:22)

Additionally, since you have protected your code so that balloons will never 
contain a null, we can remove the check in pinchAll() making the code a little 
neater.

I know, adding ``!= null`` all over your code is hardly neat, so I use some 
helper methods to make it less ugly. Here some example of how I use the "Protect" class" (full code at the bottom of the post):
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

I have picked up the habit of testing all arguments with Protect.notNull, and 
_then_ think if I really need it or if this code is implicitly protected by it 
call hierarchy. 

Once the check is in place in most of your methods, the way all the tests 
interlock makes for a warm feeling.

Yes, it’s bulky and your lean and clean code will look bloated, but mind that 
you'll save a lot of ``!=null`` checks after invoking methods. Once you get 
used to it, it also doubles as a form of documentation as any reader will 
quickly get that  nulls are not accepted.

Null protection is probably at its best when used at the constructor of an 
immutable class: you _know_ that all instances of that class do not contain 
nulls

{% highlight java%}
// a bunga bunga you can trust
public class BungaBunga {
	private final String secretLocation;
	public BungaBunga(String secretLocation) {
		Protect.valid(secretLocation);
		this.secretLocation= secretLocation;
	}
	// ....
}
{% endhighlight %}

Beware, testing for nullity will waste some cpu - on my laptop about a ns for 
the simple check (object == null) and maybe 10 for the version with varargs. 
It’s not much but they may add up, so in you may want to skip the checks in 
some regions of your codes, especially method invocations in loops. As most 
things in life, don't overdo it.

It's possible to add more complex checks or other kinds. In a recent piece of 
code I was enforcing all the calls to a class to happen from one single 
thread. In another app that was using some simple spatial geometry I enforced 
all the ints in some classes to be positive.

My Protect class is a very crude piece of code, but it does the job and it's easy to understand. Note how empty strings are treated as nulls.

{% highlight java%}
public class Protect {
	static void valid(Object object) {
		if (object == null)
			throw new IllegalArgumentException("Object in position 0 is null");
	}		
	static void valid(String string) {
		if (string == null || string.trim().length() == 0)
			throw new IllegalArgumentException("Str in pos 0 is null or empty");
	}
	static void valid(String a, String b) {
		if (a == null || a.trim().length() == 0)
			throw new IllegalArgumentException("Str in pos 0 is null or empty");
		if (b == null || b.trim().length() == 0)
			throw new IllegalArgumentException("Str in pos 1 is null or empty");
	}
	static void valid(String... strings) {
		for (int i=0; i<strings.length; i++) {
			if (strings[i] == null || strings[i].trim().length() == 0)
				throw new IllegalArgumentException("Str in pos " + i + " is null or empty");
		}
	}
	static void valid(Object a, Object b) {
		if (a == null)
			throw new IllegalArgumentException("Obj in pos 0 is null or empty");
		if (b == null)
			throw new IllegalArgumentException("Obj in pos 1 is null or empty");
		if (a instanceof String && ((String) a).trim().length() == 0)
			throw new IllegalArgumentException("Str in pos 0 is null or empty");
		if (b instanceof String && ((String) b).trim().length() == 0)
			throw new IllegalArgumentException("Str in pos 1 is null or empty");
	}
	
	// beware, this is maybe 10 times slower than the non-varargs version
	static void valid(Object... objects) {
		for (int i=0; i<objects.length; i++) {
			if (objects[i] == null)
				throw new IllegalArgumentException("Obj in pos " + i + " is null");
			if (objects[i] instanceof String && ((String) objects[i]).trim().length() == 0)
				throw new IllegalArgumentException("Str in pos " + i + " is null or empty");
		}
	}				
}
{% endhighlight %}

Thanks for reading this far, and always test your code, better if before 
writing it.
