---
layout: default
title: The Joy of Bashing
date:   2020-01-02
categories: bash script retro programming
---
# Fixing Bash with Fixed Point Arithmetic

For my new open-source project [TokenCrypt][tokencrypt],
I used good old-fashioned Bash scripting. In the process I have become a fan
of *retro programming*.

I see you throwing your arms up in horror: 'what, no Python?'.
It has to be said that Bash still holds some big advantages: 

1. Bash needs no installations and library searching.

2. The REPL loop for trying things out is right there in any terminal.

3. Bash is tightly integrated with the operating system, right out of the box.

4. 'Everything is just a string'.

4. It is sheer fun to be learning old lore and magical incantations.

In fairness, there is also one problem: no floating point arithmetic.
Suppose we want to do a simple floating point division C=A/B. 
We could reach for `awk` but the code becomes a bit messy and uncharacteristically long-winded for Bash. Plus we have to start a new
sub-process for `awk`, even for simple calculations such as this one: 
 
{% highlight bash %}
C=$( awk "BEGIN { print $A / $B }" )
{% endhighlight %}

Luckily, in most situations, there is a better alternative: **fixed point arithmetic**.
Suppose we want to calculate the ratio of the byte sizes of two directories,
for example to report the compression ratio. 

By taking any ratio,
we are in effect standardising from the hard to predict range of [0,infinity] 
to a much more compact and convenient range [0,1].
In general, it is easy to check that we are always dividing by the bigger of the two numbers.
Ratios are easier to calculate with than percentages and any barely numerate person
ought to be able to mentally translate them to percentages by simply myltiplying
 them by a hundred.

Back to our example. The following line fills an array called SIZES with the 
sizes and names of two directories:
 
{% highlight bash %}
SIZES=( $( du -hbs $OUTDIR $INDIR ) ) # array of size1, dir1, size2, dir2
{% endhighlight %}

Next we use the magic of printf formatting `"0.%05d"` to turn an ordinary integer
into a pretend 'float':

{% highlight bash %}
printf "Output size:\t%s (%s)\nOriginal size:\t%s (%s)\n \
Compression:\t0.%05d\n" ${SIZES[0]} ${SIZES[1]} ${SIZES[2]} ${SIZES[3]} \
	$(( 100000*${SIZES[0]}/${SIZES[2]} ))
{% endhighlight %}

The last line is the actual fixed point (integer) calculation. We have chosen to calculate the
ratio to five decimal places, so we premultiply by 100000. The five zeroes correspond
to the five 'decimal' places we will get via the formatting string above.
We got this precision by
making a temporary left shift by five decimal places, carrying out ordinary integer division,
followed by shifting the same number of places back to the right.

How to shift right (divide by 100000) without a floating point number to hold 
the decimal places, though? Are we just going round in circles here?

Here is the solution:
we simulate the division by prepending the literal string `'0.'` 
before the integer result `"%05d"`. We can do that when we know that the ratio
will be less than one. I bet you missed seeing that trick above? 
Bash is just perfect for such textual hacks, as 'everything is just a string'.
There you have it, Dr Watson.

Check out the [TokenCrypt][tokencrypt] 

[tokencrypt]: https://github.com/liborty/TokenCrypt
