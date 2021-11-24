---
layout: default
title: Joy of Bashing 2
date:   2021-11-24
categories: bash script retro programming
---

# Linux Multithreading Automation

## by Libor Spacek (liborty@github.com)

## Introduction

For one of my open-source github projects ([1]),
I used good old-fashioned Bash scripting and C. I admit to being a fan of *retro programming*. After all, I have been programming for over half a century myself.

In this blog I describe how I automated the spawning of background jobs within a Bash script. This technique is of general use, especially nowadays, when 32+ core CPUs are easily available for home PCs. This will considerably speed up many naturally parallel tasks, such as processing several files.

A better way of doing intricate multithreaded programming (and many other things besides) is Rust, of course. However, Bash remains by far the easiest for simple tasks.

## My Rant about Security and Privacy

This project was born out of my frustration with the state of encryption on the internet. Many of the offerings appear to be achieving the objectives of obfuscation and insecurity. Amongst them even some well intentioned ones. The underlying fact is that the Deep State in cahoots with the  Deep Tech do not like the idea of being prevented from spying on their subjects and thus have zero motivation to improve this situation.

However, there is a very simple and effective alternative: XOR symmetric encryption. Symmetric means that the same key is used for both encryption and decryption, which much simplifies the key management. It also allows such useful operations as merging of keys for multi authorisations.

Furthermore, with the key of the same length as the message, this encryption is theoretically unbreakable, i.e. the strongest possible. I like simplicity and effectiveness, so I implemented it myself in an easily usable form, see ([1]). Also, being 'old school', I believe in diy (do-it-yourself). More detailed description of that project can be found in my previous blog ([2]).

There is one reasonable objection, that generating such long keys in effect doubles the amount of storage needed. This is mitigated against by compressing the files first and some care is taken to do it effectively. So, for example, with 50% compression achieved, there is no storage overhead at all. In any case, disks are cheap and the security is worth it even with some overhead. On the plus side, the encryption and decryption are very fast, being so simple.

## Encrypting Many Files Faster

The tasks of compressing, encrypting, decompressing and decrypting all the files in a given directory are clearly I/O bound. They only depend on one file at a time, so can be performed in parallel, without cross communications.

The most suitable compression method is selected and applied for each file individually and also the key of the right length is generated on encryption or looked up on decryption. The code is general enough to take care of this. I will not dwell on the details here, they are to be found in ([3]),([4]).

Our aim is to fire off background jobs for longer files first and leave the shorter ones to be processed in the current shell in the end, to avoid the subshell overheads for them. This will minimise the overall script execution time. We also want simple code without too many tests and complicated 'if' statements.

First the script arguments and flags are processed and used to set up the correct input and output directories. Then the input files are sorted by their lengths:

```bash
# size of files to be encrypted in the subshells
BACKSZ=10000 
# sort files in INDIR by size (in bytes)
ITEMS=$(ls -SL $INDIR)
# prepare to fire off subshells
BJ=\&
```

The last line above is a wonderous hack to set up the default textual command ending `$BJ` to value `&`, so that jobs will be run in the background sub-shells.

What remains now is to process all the files in the order of their decreasing size and when the threshold `BACKSZ` is reached, permanently turn off the sub-shelling:

```bash
for FILEND in $ITEMS; do
   INFILE=$INDIR/$FILEND # reconstruct one full path/filename
   [ -f "$INFILE" ] || continue # only process a genuine file
   let COUNT+=1 # count the files processed
   # turn off subshells when file size falls below BACKSZ
   [ -z "$BJ" ] || [ $( stat -c%s $INFILE ) -ge $BACKSZ ] || BJ='' 
   processfile $BJ  # the function that processes one file
done
wait # for all proccessfiles to finish
```

Bash functions are executed in a sub shell only when their invocation ends in `&`. When `$BJ` variable is programmatically unset, it simply disappears and execution is thereafter done in-place. Self modifying program, ha! (The beautitudes of 'everything is just text' Bash).

We use logical or `||` instead of `if`. It only executes the last command, unsetting `BJ=''`, if `$BJ` was previously set and the size of the current `$INFILE` has fallen below the threshold `$BACKSZ`. The only check performed thereafter is the first part: [ -z "$BJ" ] is `$BJ` unset? (evaluates to true and exits || immediately). If there are lots of short files, it saves the relatively expensive `stat` system calls on all the rest of them. They are already sorted, so none of them will exceed the threshold anyway.

## Link References

1 https://github.com/liborty/TokenCrypt, TokenCrypt, github repository.   
2 https://oldmill.cz/2020-06-10-crypt.html, Encryption and E-Democracy, previous blog.  
3 https://github.com/liborty/TokenCrypt/blob/master/ncrpt, ncrpt, bash script within ([1]).  
4 https://github.com/liborty/TokenCrypt/blob/master/dcrpt, dcrpt, bash script within ([1]).   

[1]: https://github.com/liborty/TokenCrypt "TokenCrypt"
[2]: https://oldmill.cz/2020-06-10-crypt.html "Encryption and E-Democracy"  
[3]: https://github.com/liborty/TokenCrypt/blob/master/ncrpt "ncrpt"
[4]: https://github.com/liborty/TokenCrypt/blob/master/dcrpt "dcrpt"
