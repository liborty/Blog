---
layout: default
title: Better Branching in Bash
date:   2021-11-30
categories: bash script retro programming
---

# Better Branching in Bash

## Libor Spacek (liborty@github.com)

## Introduction

I love Bash really but it is a love-hate relationship. Take those `if` statements, for example. Forget to put a semicolon  **before** `then` and you are in deep trouble (?!?!). It would be a pity if this puts people off shell programming.

Fortunately, there is a much better way of branching on boolean tests, as exemplified in some of my code in my github encryption project ([1]).

## Short Circuit Logic Evaluation

When evaluating a logical (boolean) expression, in any context, the value of the result is often known well before all the parts of the expression have been evalauted. How is this even possible?

In a conjuction (`&&`), the whole result is known to be false as soon as the first conjuct returns `false`.

Conversely, in a disjunction (`||`), the whole result is known to be true as soon as the first disjunct returns `true`.

This property of logic is used in AI for efficient evaluation of so called AND-OR trees in game-playing and search applications. However, we can use it too, even in our simple `bash` programs.

## Going beyond `if`

In keeping with the previous section, we introduce two kinds of conditionals:

### The positive conditional

`[ test1 ] && [ test2 ] && [ testn ] && { do-something-positive }`

Here the block of statements at the end is executed only if each of the tests 1..n returns `true`. As soon as the first one evaluates to `false`, the whole thing quits (hence the expression 'short circuit evaluation').

### The negative conditional

`[ test1 ] || [ test2 ] || [ testn ] || { do-something-negative }`

Here the block at the end is executed only if each of the tests 1..n returns `false`. As soon as the first one evaluates to `true`, the whole thing quits.

## Discussion

This syntax is much cleaner, more compact, and less error prone than the `if` statements provided in `bash` and most other scripting language variants.

The semantics is more expressive, with two types of if-statements instead of just one.

Efficiency can be gained by ordering the individual tests. In a positive conditional, the tests most likely to fail should come first.

In a negative conditional, the tests most likely to succeed should come first.

When unsure, then ordering the tests from the fastest to the slowest is a good idea, in both cases.

Arbitrarily complex propositional logic expressions can be built within this framework, using `!` (not) and brackets. So, for example, to intermix a negative test1 into the positive conditional, just use `!test1`.

## Link References

([1]) TokenCrypt, github repository.

([2]) `ncrpt`, bash script in ([1]).  

[1]: https://github.com/liborty/TokenCrypt "TokenCrypt"
[2]: https://github.com/liborty/TokenCrypt/blob/master/ncrpt "ncrpt"
