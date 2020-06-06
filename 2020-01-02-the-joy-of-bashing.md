---
layout: default
title: The Joy of Bashing
date:   2020-01-02
categories: bash script retro programming
---
# Fixing Bash with Fixed Point Arithmetic

For my new open-source project [TokenCrypt][tokencrypt],
I used good old-fashioned Bash scripting. In the process I have become a fan of *retro programming*.

I hear you exclaiming in horror: "What? No Python?".
It has to be said that Bash still holds some big advantages: 

1. Bash needs no installations and library searching.
2. The REPL loop for trying things out is right there in any terminal.
3. Bash is tightly integrated with the operating system, right out of the box.
4. EIJAS = 'Everything is just a string'.
5. It is sheer fun to be learning old lore and magical incantations, 
though this may be a matter of taste and opinion.

In fairness, there is also one problem: no floating point arithmetic.

Suppose we want to do a simple floating point division C=A/B. 
We could reach for `awk` (`dc,bc`, etc.) but the code becomes a bit messy 
and uncharacteristically long-winded for Bash. Most of the facilities that
these external programs offer are an overkill. Plus we have to start new
sub-processes for them, even for every simple calculations such as this example: 
```bash
C=$( awk "BEGIN { print $A / $B }" )
```
Luckily, in most situations, there is a more efficient alternative:
**fixed point arithmetic** implemented right there and then in Bash.

####The Plan
Suppose we want to calculate the ratio of the byte sizes of two directories, let us say to report the compression ratio. 

By taking any ratio,
we are in effect standardising from the hard to handle range of [0,infinity) 
to a much more compact and convenient range [0,1].
The round closing bracket above denotes an open interval. Computers can not handle infinite values, so we exclude it. 
In general, it is easy to check that we are always dividing by the bigger of the two numbers.

Ratios are easier to calculate with than percentages and any barely numerate person
ought to be able to mentally translate them to percentages by simply myltiplying
 them by a hundred.

Back to our example application. The following line fills an array called SIZES with the sizes and names of two directories:
 
```bash
# array [size1, dir1, size2, dir2]
SIZES=( $( du -hbs $OUTDIR $INDIR ) ) 
```

Next we use the magic of printf formatting `"0.%05d"` to turn an ordinary integer into a pretend 'float':

```bash
printf "Output size:\t%s (%s)\nOriginal size:\t%s (%s)\n \
Compression:\t0.%05d\n" 
${SIZES[0]} ${SIZES[1]} ${SIZES[2]} ${SIZES[3]} \
	$(( 100000*${SIZES[0]}/${SIZES[2]} ))
```

The last line is the actual fixed point (integer) calculation. We have chosen here to calculate the
ratio to five decimal places, so we premultiply by 100000. The five zeroes correspond
to the five decimal places we will get via the formatting string above.

We got this precision by
making a temporary left shift by five decimal places, carrying out ordinary integer division,
followed by shifting the same number of places back to the right.

Good plan but how to shift right (divide by 100000) without a floating point type to hold the decimal places of the result? Are we going around in circles with this?

####The Magic Explained 
We just textually simulate the float number by prepending the literal string `'0.'` before the integer result. We can do that whenever we know that the ratio will be less than one. I bet you missed seeing that trick above?

At a stroke of horrible type-violating magic
that would make any compiler die in convulsions, we turned 1234 into 0.01234. In Bash this is not nearly as horrible as it looks because everything returns only strings, anyway. The leading zero in the formatting string  `"%05d"` that follows the decimal point is also important. It creates significant leading zeroes, if any.


Should you wish to print the ratio as a percentage, this will do it:

```bash
# compression ratio x 10^4, presented as xx.xx%
CR=$(( 10000*${SIZES[0]}/${SIZES[2]} )) 
if [ $CR -gt 9999 ]; then 
	printf "Warning, no compression achieved!\n"
else  
	printf "Compression to:\t${CR: 0:2}.${CR: -2}%%\n"
```

Bash is just perfect for such textual hacks, as 'everything is just a string'.

####References

I have written a Bash script [fdiv](https://github.com/liborty/fdiv) that implements these ideas, plus more, to give the best 'floating point' results of any integer division.

You may also like to look at [TokenCrypt](https://github.com/liborty/TokenCrypt). 
