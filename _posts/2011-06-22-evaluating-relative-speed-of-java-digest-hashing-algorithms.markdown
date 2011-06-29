--- 
wordpress_id: 672
layout: post
title: Evaluating relative speed of java digest (hashing) algorithms
date: 2011-06-22 14:27:20 +00:00
wordpress_url: http://caprazzi.net/?p=672
---
It's best practice to encrypt security tokens such as passwords and sessions ids in your database. I was just doing that to the session tokens for a project at work, and I wondered which algorithm to pick if speed was the only consideration.

I cranked up some code that measures the wall-clock that it takes for each algorithm to hash a bunch of strings (10 millions). I found that MD5 is the fastest, and MD2 is the slowest, taking roughly twice the time. I would have gone for MD5 anyway as it is the standard choice, but it's good to see that it's quick too.

The usual disclaimers apply: this test is very un-scientific and is only relevant for my specific use case (the tokens I handle are the same length of the random strings in the test) so don't plan your business on it. That's pretty quick stuff anyway (15 to 30 microsecond each encryption), so you may want to pick an algorithms for its security features rather then for its execution speed. See [the docs](// see http://download.oracle.com/javase/1.5.0/docs/guide/security/CryptoSpec.html#AppA) for more info.

Output:

<pre class="terminal">
Creating 10000000 random strings... Created.
Testing algo MD2...	Completed in 339126 milliseconds
Testing algo MD5...	Completed in 169690 milliseconds
Testing algo SHA-1...	Completed in 200398 milliseconds
Testing algo SHA-256...	Completed in 211560 milliseconds
Testing algo SHA-384...	Completed in 303999 milliseconds
Testing algo SHA-512...	Completed in 316265 milliseconds
Test Complete.
</pre>


And code:
{% highlight java %}
import java.math.BigInteger;
import java.security.MessageDigest;
import java.security.SecureRandom;

public class CompareHashFunctions {

	private static SecureRandom random = new SecureRandom();

	public static void main(String[] args) throws Exception {

		int runs = 10000000;				

		System.out.print("Creating " + runs + " random strings... ");
		String salt = randomString();
		String[] strings = new String[runs];
		for (int i=0; i<strings.length; i++) {
			strings[i] = randomString();
		}

		System.out.println("Created. ");	

		runTest(salt, strings, "MD2");
		runTest(salt, strings, "MD5");
		runTest(salt, strings, "SHA-1");
		runTest(salt, strings, "SHA-256");
		runTest(salt, strings, "SHA-384");
		runTest(salt, strings, "SHA-512");		

		System.out.println("Test Complete.");
	}

	static void runTest(String salt, String[] strings, String algo) throws Exception {
		System.out.print("Testing algo " + algo + "...\t");
		MessageDigest instance = MessageDigest.getInstance(algo);
		long start = System.nanoTime();
		for (int i=0; i<strings.length; i++) {
			byte[] bytes = (salt + strings[i]).getBytes("UTF-8");
			MessageDigest clone = (MessageDigest)instance.clone();
			new String(clone.digest(bytes));
		}
		long elapsed = (System.nanoTime() - start) / 1000000;
		System.out.println("Completed in " + elapsed +  " milliseconds ");
	}		

	/**
	 * Random string
	 * @return a random string of 25 or 26 chars
	 */
	static String randomString() {
		// 130 bit random integer converted to string in base 32
		return new BigInteger(130, random).toString(32);
	}
}
{% endhighlight %}

-teo
