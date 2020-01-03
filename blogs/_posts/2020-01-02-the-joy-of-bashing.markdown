---
layout: post
title: The Joy of Bashing
date:   2020-01-02
categories: bash script retro programming
---
## (Retro Programming)

Within my new open-source project [TokenCrypt][tokencrypt]
I went for good,
simple, old-fashioned Bash scripting and in the process I have become a fan of 'retro programming'.

'What? No Python?' I see you throwing your arms up in horror.
It has to be said that Bash still holds some advantages: 

1. Bash needs no installations and library searching.

2. The REPL loop for trying things out is right there in any terminal.

3. Bash is tightly integrated with the operating system, right out of the box.

4. 'Everything is just a string'.

4. It is sheer fun to learn old lore and magical incantations.

In fairness, there is also one problem: no floating point arithmetic.
Suppose we want to do a simple floating point division C=A/B.

We can reach for `AWK` but the code is a bit messy.
It is uncharacteristically long-winded for Bash. Plus we have to start a new
sub-process for `awk`, even for simple calculations such as this one: 
 
{% highlight bash %}
C=$( awk "BEGIN { print $A / $B }" )
{% endhighlight %}

Luckily, in most situations, there is a better alternative: **fixed point arithmetic**.
Suppose we want to calculate the ratio of the byte sizes of two directories,
for example to report their compression ratio. 

It will be a number between zero and one.
In general, it is easy to check that we are always dividing by the bigger of the two numbers.
Ratios are better to calculate with than percentages and any barely numerate person
ought to be able to mentally translate them to percentages by myltiplying them by 100.

The following line fills an array called SIZES with the sizes and names of the
two directories:
 
{% highlight bash %}
SIZES=( $( du -hbs $OUTDIR $INDIR ) ) # array of size1, dir1, size2, dir2
{% endhighlight %}

Next we use the magic of printf formatting `"0.%05d"` to turn an ordinary integer into a 'float':
{% highlight bash %}
printf "Output size:\t%s (%s)\nOriginal size:\t%s (%s)\n \
Compression:\t0.%05d\n" ${SIZES[0]} ${SIZES[1]} ${SIZES[2]} ${SIZES[3]} \
	$(( 100000*${SIZES[0]}/${SIZES[2]} ))
{% endhighlight %}
The last line is the actual fixed point (integer) calculation. We have chosen to calculate the
ratio to five decimal places, so we premultiply by 100000. The five zeroes correspond
to the five 'decimal' places in the formatting string above. It is just a temporary 
left shift by five decimal places, followed by a right shift back, simulated by
prepending the literal string '0.' before the integer result. And there you have it, Dr Watson.

Check out the [TokenCrypt][tokencrypt] 

[tokencrypt]: https://github.com/liborty/TokenCrypt
