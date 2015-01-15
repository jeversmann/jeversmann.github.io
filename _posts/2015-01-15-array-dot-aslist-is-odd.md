---
layout: post
title: "Array.asList is odd"
---

Java's `Arrays.asList` is a weird type of function, and can be dangerous.

Looking at the name alone, it seems like a useful way to view an array through the methods you get with a `Collection`.
For instance, List includes a `toString` method which arrays do not,
so you could put together some code to print an array like this:
{% highlight java %}
Integer[] arr = new Integer[]{ 1, 2, 3 };
System.out.println(Arrays.asList(arr)); // This prints "[1, 2, 3]"

// The oddness of this method is wth primative types
int[] primativeArr = new int[]{ 1, 2, 3 };
System.out.println(Arrays.asList(primativeArr)); // This prints "[[I@171e1813]"
                                                 // or some other object ID
{% endhighlight %}

What's happening here?

{% highlight java %}
public static <T> List<T> asList(T... a)
{% endhighlight %}
This is the method header from the [javadoc](http://docs.oracle.com/javase/7/docs/api/java/util/Arrays.html#asList(T...));
`asList` uses varargs instead of taking an input array.
This seems weird, but is necessary because a method which takes a generic array can't take primative arrays.
Java's primative types do not extend `Object`, and anywhere you use a type parameter the types must be bounded by `Object`
(you can manually add additional bounds but not loosen this bound).
However, arrays of primative types in Java are instances of `Object`,
which means that in the second above example `Arrays.asList` is returning a `List<int[]>` and the strange output is the result of `toString` being called on an `int[]`.

Knowing that varargs is what's at work here, the real question becomes:
{% highlight java %}
// Why is this:
Integer[] arr = new Integer[]{ 1 };
System.out.println(Arrays.asList(arr)); // This prints "[1]"

// The same as this?
System.out.println(Arrays.asList( 1 )); // Also prints "[1]"
{% endhighlight %}
Apparently, if a single `Object[]` is provided instead of multiple arguments to a vararg function,
it isn't collected into an `Object[][]` with one element.


There certianly must be reasons for all of this, and I don't know them;
just be careful where you put your primative arrays.
