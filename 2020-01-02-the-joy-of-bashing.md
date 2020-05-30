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
4. EIJAS = 'Everything is just a string'.
5. It is sheer fun to be learning old lore and magical incantations
(though this may be a matter of taste and opinion).

In fairness, there is also one problem: no floating point arithmetic.

Suppose we want to do a simple floating point division C=A/B. 
We could reach for `awk` (`dc,bc`, etc.) but the code becomes a bit messy 
and uncharacteristically long-winded for Bash. Most of the facilities that
these external programs offer are an overkill. Plus we have to start new
sub-processes for them, even for every simple calculations such as this example: 
 
{% highlight bash %}
C=$( awk "BEGIN { print $A / $B }" )
{% endhighlight %}

Luckily, in most situations, there is a more efficient alternative:
**fixed point arithmetic** implemented right there and then in Bash.
Suppose we want to calculate the ratio of the byte sizes of two directories,
let us say to report the compression ratio. 

By taking any ratio,
we are in effect standardising from the hard to handle range of [0,infinity) 
to a much more compact and convenient range [0,1].
The round closing bracket denotes an open interval, meaning that computers can not handle
infinite values, so we exclude it. 
In general, it is easy to check that we are always dividing by the bigger of the two numbers.
Ratios are easier to calculate with than percentages and any barely numerate person
ought to be able to mentally translate them to percentages by simply myltiplying
 them by a hundred.

Back to our example. The following line fills an array called SIZES with the 
sizes and names of two directories:
 
{% highlight bash %}
SIZES=( $( du -hbs $OUTDIR $INDIR ) ) # array [size1, dir1, size2, dir2]
{% endhighlight %}

Next we use the magic of printf formatting `"0.%05d"` to turn an ordinary integer
into a pretend 'float':

{% highlight bash %}
printf "Output size:\t%s (%s)\nOriginal size:\t%s (%s)\n \
Compression:\t0.%05d\n" ${SIZES[0]} ${SIZES[1]} ${SIZES[2]} ${SIZES[3]} \
	$(( 100000*${SIZES[0]}/${SIZES[2]} ))
{% endhighlight %}

The last line is the actual fixed point (integer) calculation. We have chosen here to calculate the
ratio to five decimal places, so we premultiply by 100000. The five zeroes correspond
to the five decimal places we will get via the formatting string above.
We got this precision by
making a temporary left shift by five decimal places, carrying out ordinary integer division,
followed by shifting the same number of places back to the right.

How to shift right (divide by 100000) without a floating point number to hold 
the decimal places, though? Are we just going round in circles here?

Here is the solution:
we simulate it by prepending the literal string `'0.'` 
before the integer result `"%05d"`. At a stroke of horrible type-violating magic
that would make any compiler die in convulsions, we turned 12345 into 0.12345. 
We can do that whenever we know that the ratio
will be less than one. I bet you missed seeing that trick above?

Should you wish to print the ratio as a percentage, this will do it:

{% highlight bash %}
CR=$(( 10000*${SIZES[0]}/${SIZES[2]} )) # compression ratio x 10^4 ( xx.xx % )
if [ $CR -gt 9999 ]; then 
	printf "Warning, no compression achieved!\n"
else 	
	printf "Compression to:\t${CR: 0:2}.${CR: -2}%%\n"
{% endhighlight %}

Bash is just perfect for such textual hacks, as 'everything is just a string'.

I have written a Bash script that implements these ideas, plus more. It is at this github repository: [fdiv](https://github.com/liborty/fdiv)  

Check out also [TokenCrypt](https://github.com/liborty/TokenCrypt) 
