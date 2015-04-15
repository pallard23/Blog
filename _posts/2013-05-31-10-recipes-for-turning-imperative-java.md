---
layout: post
title: 10 recipes for turning imperative Java code into functional Scala code
date: '2013-05-31T14:52:00.001-07:00'
author: Yevgeniy Brikman
tags:
- Software Engineering
modified_time: '2013-06-01T18:45:18.524-07:00'
thumbnail: http://3.bp.blogspot.com/-SLIrkeD61RM/UaiAl5RIxLI/AAAAAAAAM-E/L4OspA2FZAs/s72-c/scala-logo.png
blogger_id: tag:blogger.com,1999:blog-5422014336627804072.post-7191719948429771684
blogger_orig_url: http://brikis98.blogspot.com/2013/05/10-recipes-for-turning-imperative-java.html
---

At LinkedIn, [we've started to use the Play 
Framework](http://engineering.linkedin.com/play/play-framework-linkedin), 
which supports not only Java, but also Scala. Many teams have opted to write 
their apps in Scala, so I've spent a fair amount of time helping team members 
learn the language. 

<div class="separator" style="clear: both; text-align: center;">[<img 
border="0" 
src="http://3.bp.blogspot.com/-SLIrkeD61RM/UaiAl5RIxLI/AAAAAAAAM-E/L4OspA2FZAs/s1600/scala-logo.png" 
/>](http://3.bp.blogspot.com/-SLIrkeD61RM/UaiAl5RIxLI/AAAAAAAAM-E/L4OspA2FZAs/s1600/scala-logo.png) 
Most LinkedIn engineers are proficient in Java, so their early Scala code 
looks like a literal translation from Java to Scala: lots of for-loops, 
mutable variables, mutable collection classes, null values, and so on. While 
this code *works*, it's not taking advantage of one of Scala's biggest 
strengths: strong support for functional programming. 

In this post, I want to share 10 recipes for how to translate a few of the 
most common imperative Java patterns into functional Scala code. 

<span style="font-size: x-large;">**Why functional programming?** 

Why would you want to make your code "functional"? This question has been 
asked and answered many times, so rather than recreating the answers myself, 
I'll point you to a couple good starting points: 
1. [Why Functional Programming 
Matters](http://www.cs.kent.ac.uk/people/staff/dat/miranda/whyfp90.pdf) by 
John Hughes 
1. [Functional Programs Rarely 
Rot](http://michaelochurch.wordpress.com/2012/12/06/functional-programs-rarely-rot/) 
by Michael O. Church 
It's worth mentioning that Scala is *not* a pure functional language, but it 
is still worth trying to make as much of your code as possible out of (a) 
small functions that (b) use only immutable state and (c) are side effect 
free. If you do that, I believe your code will generally be easier to read, 
reason about, and test. 

Now, on to the cookbook! 

<span style="font-size: x-large;">**Recipe 1: building a list** 

Let's start easy: we want to loop over a List of data, process each item in 
some way, and store the results in a new List. Here is the standard way to do 
this in Java: 

<script src="https://gist.github.com/brikis98/5683445.js"></script> It's 
possible to translate this verbatim into Scala by using a 
<code>mutable.List</code> and a for-loop, but there is no need to use mutable 
data here. In fact, there is *very rarely* a reason to use mutable variables 
in Scala; think of it as a code smell. 

Instead, we can use the <code>map</code> method, which creates a new List by 
taking a function as a parameter and calling that function once for each item 
in the original List (see: [map and flatMap in 
Scala](http://www.brunton-spall.co.uk/post/2011/12/02/map-map-and-flatmap-in-scala/) 
for more info): 

<script src="https://gist.github.com/brikis98/5683455.js"></script> 
<span style="font-size: x-large;">**Recipe 2: aggregating a list** 

Let's make things a little more interesting: we again have a List of data to 
process, but now we need to calculate some stats on each item in the list and 
add them all up. Here is the normal Java approach: 

<script src="https://gist.github.com/brikis98/5683478.js"></script> Can this 
be done without a mutable variable? Yup. All you need to do is use  the 
<code>foldLeft</code> method (read more about it 
[here](http://oldfashionedsoftware.com/2009/07/30/lots-and-lots-of-foldleft-examples/)). 
This method has two parameter lists (Scala's version of 
[currying](http://www.scala-lang.org/node/135)): the first takes an <span 
style="font-family: monospace;">initial value and the second takes a function. 
<span style="font-family: monospace;">foldLeft will iterate over the contents 
of your List and call the passed in function with two parameters: the 
accumulated value so far (which will be set to <span style="font-family: 
monospace;">initial value on the first iteration) and the current item in the 
List. 

Here is the exact same <span style="font-family: 
monospace;">calculateTotalStats written as a pure Scala function with only 
immutable variables: 

<script src="https://gist.github.com/brikis98/5683501.js"></script> 
<span style="font-size: x-large;">**Recipe 3: aggregating multiple items** 

Ok, perhaps you can do some simple aggregation with only immutable variables, 
but what if you need to calculate multiple items from the List? And what if 
the calculations were conditional? Here is a typical Java solution: 

<script src="https://gist.github.com/brikis98/5683532.js"></script> Can this 
be done in an immutable way? Absolutely. We can use <span style="font-family: 
monospace;">foldLeft again, combined with [pattern 
matching](http://www.scala-lang.org/node/120) and a [case 
class](http://www.scala-lang.org/node/107) (case classes give you [lots of 
nice freebies](http://www.scala-lang.org/node/258)) to create an elegant, 
safe, and easy to read solution: 

<script src="https://gist.github.com/brikis98/5683557.js"></script> 
<span style="font-size: x-large;">**Recipe 4: lazy search** 

Imagine you have a List of values and you need to transform each value and 
find the first one that matches some condition. The catch is that transforming 
the data is expensive, so you don't want to transform any more values than you 
have to. Here is the Java way of doing this: 

<script src="https://gist.github.com/brikis98/5684075.js"></script> The normal 
Scala pattern for doing this would be to use the <span style="font-family: 
monospace;">map method to transform the elements of the list and then call the 
<span style="font-family: monospace;">find method to find the first one that 
matches the condition. However, the <code>map</code> method would transform 
*all* the elements, which would be wasteful if one of the earlier ones is a 
match. 

Fortunately, Scala supports 
[Views](http://www.scala-lang.org/api/current/index.html#scala.collection.SeqView), 
which are collections that lazily evaluate their contents. That is, none of 
the values or transformations you apply to a <code>View</code> actually take 
place until you try to access one of the values within the <code>View</code>. 
Therefore, we can convert our List to a <code>View</code>, call 
<code>map</code> on it with the transformation, and then call 
<code>find</code>. Only as the <code>find</code> method accesses each item of 
the <code>View</code> will the transformation actually occur, so this is 
exactly the kind of lazy search we want: 

<script src="https://gist.github.com/brikis98/5684093.js"></script> 
Note that we return an <code>Option[SomeOtherObject]</code> instead of null. 
Take a look at Recipe 7 for more info. 

<span style="font-size: x-large;">**Recipe 5: lazy values** 

What do you do if you want a value to be initialized only when it is first 
accessed? For example, what if you have a singleton that is expensive to 
instantiate, so you only want to do it if someone actually uses it? One way to 
do this in Java is to use <code>volatile</code> and <code>synchronized</code>: 

<script src="https://gist.github.com/brikis98/5684180.js"></script> Scala has 
support for the <code>lazy</code> keyword, which will initialize the variable 
only when it is first accessed. Under the hood, it does something similar to 
<code>synchronized</code> and <code>volatile</code>, but the code written by 
the developer is easier to read: 

<script src="https://gist.github.com/brikis98/5684220.js"></script> 
<span style="font-size: x-large;">**Recipe 6: lazy parameters** 

If you've ever worked with a logging library like 
[log4j](http://logging.apache.org/log4j/2.x/), you've probably seen Java code 
like this: 

<script src="https://gist.github.com/brikis98/5684134.js"></script> The 
logging statement is wrapped with an <code>isDebugEnabled</code> check to 
ensure that we don't calculate the expensive diagnostics info if the debug 
logging is actually disabled. 

In Scala, you can define lazy function parameters that are only evaluated when 
accessed. For example, the logger debug method could be defined as follows in 
Scala (note the <code>=&gt;</code> in the type signature of the 
<code>message</code> parameter): 

<script src="https://gist.github.com/brikis98/5684161.js"></script> This means 
the logging statements in my code no longer need to be wrapped in if-checks 
even if the data being logged is costly to calculate, since it'll only be 
calculated if that logging level is actually enabled: 

<script src="https://gist.github.com/brikis98/5684149.js"></script> 
<span style="font-size: x-large;">**Recipe 7: null checks** 

A common pattern in Java is to check that a variable is not null before using 
it: 

<script src="https://gist.github.com/brikis98/5683739.js"></script>If you're 
working purely in Scala, and have a variable that might not have a value, you 
should not set it to null. In fact, think of nulls in Scala as a code smell. 

The better way to handle this situation is to specify the type of the object 
as an [Option](http://www.scala-lang.org/api/current/index.html#scala.Option). 
<code>Option</code> has two subclasses: 
[Some](http://www.scala-lang.org/api/current/index.html#scala.Some), which 
contains a value, and 
[None](http://www.scala-lang.org/api/current/index.html#scala.None$), which 
does not. This forces the programmer to explicitly acknowledge that the value 
could be <code>None</code>, instead of sometimes forgetting to check and 
stumbling on a <code>NullPointerException</code>. 

You could use the <code>isDefined</code> or <code>isEmpty</code> methods with 
an <code>Option</code> class, but pattern matching is usually cleaner: 

<script src="https://gist.github.com/brikis98/5683775.js"></script> 
The <code>Option</code> class also supports methods like <code>map</code>, 
<code>flatMap</code>, and <code>filter</code>, so you can safely transform the 
value that may or may not be inside of an <code>Option</code>. Finally, there 
is a <code>getOrElse</code> method which returns the value inside the 
<code>Option</code> if the <code>Option</code> is a <code>Some</code> and 
returns the specified fallback value if the <code>Option</code> is a 
<code>None</code>: 

<script src="https://gist.github.com/brikis98/5692280.js"></script> 
Of course, you rarely live in a nice, walled off, pure-Scala garden - 
especially when working with Java libraries - so sometimes you'll get a 
variable passed to you that isn't an <span style="font-family: 
monospace;">Option but could still be null. Fortunately, it's easy to wrap it 
in an <code>Option</code> and re-use the code above: 

<script src="https://gist.github.com/brikis98/5683788.js"></script> 
<span style="font-size: x-large;">**Recipe 8: multiple null checks** 

What if you have to walk an object tree and check for null or empty at each 
stage? In Java, this can get pretty messy: 

<script src="https://gist.github.com/brikis98/5683803.js"></script> With 
Scala, you can take advantage of a [sequence 
comprehension](http://www.scala-lang.org/node/111) and 
[Option](http://www.scala-lang.org/api/current/index.html#scala.Option) to 
accomplish the exact same checks with far less nesting: 

<script src="https://gist.github.com/brikis98/5683828.js"></script> 
<span style="font-size: x-large;">**Recipe 9: instanceof and casting** 

In Java, you sometimes need to figure out what kind of class you're dealing 
with. This involves some <span style="font-family: monospace;">instanceof 
checks and casting: 

<script src="https://gist.github.com/brikis98/5683993.js"></script> We can use 
[pattern matching](http://www.scala-lang.org/node/120) and [case 
classes](http://www.scala-lang.org/node/107) in Scala to make this code more 
readable, even though it does the same <code>instanceof</code> checks and 
casting under the hood: 

<script src="https://gist.github.com/brikis98/5684007.js"></script> 
<span style="font-size: x-large;">**Recipe 10: regular expressions** 

Let's say we want to match a String and extract some data from it using one of 
a few regular expressions. Here is the Java code for it: 

<script src="https://gist.github.com/brikis98/5684026.js"></script> In Scala, 
we can take advantage of [extractors](http://www.scala-lang.org/node/112), 
which are automatically created for regular expressions, and pattern matching 
using [partial 
functions](http://blog.bruchez.name/2011/10/scala-partial-functions-without-phd.html), 
to create a much more readable solution: 

<script src="https://gist.github.com/brikis98/5684035.js"></script> **<span 
style="font-size: x-large;">Got some recipes of your own?** 

I hope this post has been helpful. It's worth noting that the recipes above 
are only one of many ways to translate the code; for example, many of the List 
examples could have also been done with recursion. 

If you've got suggestions on how to make the examples above even better or 
have some handy recipes of your own, leave a comment! 