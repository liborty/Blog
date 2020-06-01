---
layout: default
title: To Rust or not to Rust? That was my Challenge
date:   2020-05-27
categories: Errors, Rust programming
---
# To Rust or not to Rust? That Was my Challenge
# (A Personal Road to Golgotha)
# by Rustafarian

### Introduction
I have always had this perverse interest in new, complicated and difficult programming languages. What could then pose a better challenge than taking up Rust? Of course, Rust also has some good  (even excellent) points to commend it: 

- Zero cost abstractions. All kinds of high level convenience constructs and the strong types  get compiled away into moreless the same native code as if they were not there.

- Speed of native code. I never believed in wasting time when it can be avoided. Rust's speed is comparable to C(++). 

- Simplicity of native code. No need to carry massive interpreter runtimes on your back. No need to install all kinds of compatible libraries on your target machine (goodbye node). I can compile on my development machine and send the single file executable to the server (with the target architecture) and It Just Runs.

- Safe multi-threaded execution. Mutation (overwriting) and aliasing (multiple pointers) can never apply to the same memory location at the same time. This is a big one, with most performance gains to be still fuly realised and appreciated.

- Memory safety also means reliability and security.

- Relatively easy async/await constructs for handling asynchronous I/O. This is critical for net  connections, e.g. https, which are in terms of the CPU time extremely slow and it makes no sense to idly wait for them, blocking even a single thread.

#### Station One - Mutation and Ownership
The first thing that a new Rust user will notice is that the prevention of mutation with aliasing resuts in some fairly severe restrictions on one's programming style and the freedoms we used to enjoy. In particular, it is no longer possible to willy-nilly pass `left values` (things that boild down to a writeable memory location) around and to modify their contents at will. 

You have to explicitly mark as `mut` only those declared variables that absolutely must be overwritten,  pass them as `mut` parameters only to those functions where they actually are overwritten, and not do that from more than one place, even within the same block of code.

Mutability is actually quite a useful concept/distinction to make. An early pleasant surprise to me was to find out just how many variables never need to be mutable at all. Though variable  is perhaps no longer the correct term. More like 'assign once' variable. Their only difference from constants is that their value is computed and assigned at runtime to a location on the stack.

My only gripe with this is that when you declare a whole bunch of variables, the compiler remembers perfectly well which ones are mutable or not but you often don't. I recommend adopting special case style, such as HolE CasE, so that they are immediately humanly recognisable. Akin to CAPITALS for constants.

You also have to worry about `ownership` to solve the aliasing problem. For examle, you can  share several pointers to an immutable item but not to a mutable one. This is called `borrowing`, though I would have preferred the time honoured terminology of 'passing a pointer'. Inventing new terms for existing concepts 'just to be different' only adds potential confusion. Perhaps it was meant to convey that borrowed items should not be destroyed but then it just fails in this mission, as anyone who has ever lent his Ferrari to his teenage son will confirm.

All of this means that initially the compiler will complain at you like mad but its error messages are fairly self-explanatory and often it will even tell you what it wants you to do.

#### Station Two - Lifetimes
I hope I will be excused for leaving these to a future blog in its own right. Mostly because so far I have failed to get my head around the reasons why the lifetime annotations have to be quite so complicated (and yes, let us be blunt, so syntactically horrible). I just put them where the compiler tells me it wants them to be. With a better written program, they mostly go away.

#### Station Three - Crates
The Rust libraries are called crates and like everything in Rust, they are remarkable safe, even to import from a vast collection of them by every Tom Dick and Harry at crates.io.

They can take ages to compile, due to whole trees of dependencies.

You must specify them in Cargo.toml at the top level of your project directory. Just beware that there are various characters, such as '^'  that when prepended before a crate's name can alter in mysterious ways how that crate is going to get updated. As does leaving out or not leaving out various digits from the crate's version number. 

As if this was not complicated enough, some facilities of some crates are available only as their 'special features', which must be known by name and listed in curly brackets, otherwise they will not work.

#### Station Four - Async I/O
This uses `futures` or place markers for some computation which is yet to finish. The concepts themselves are not difficult to grasp. Though you must beware, that the results can come back in any unpredictable order. I like best the `!try_join(..)` macro and use it heavily to great effect.

That is not to say that I have not experienced a lot of practical difficulties. They were not so much with the concepts or the programming itself but rather with the state of the documentation and the crates, both being moving targets. It was not at all obvious which versions of which to use with which features and in what combinations, at what time. Different people used and recommended different crates in different ways and the same applied to the documentation. 

However, I believe that the situation has now somewhat settled from 'the bleeding edge' and anyone embarking on this adventure now should find it a lot easier.

#### Station Five - Modules
This is one area where I have nothing but praise. Perhaps because the modules are so simple and Just Work. I have originally written my implementation as one largish file but once I have discovered the modules, it was a matter of minutes to split it into its logical parts. The whole project gained much needed structure and readability and perhaps even shorter recompilation times.

#### Station Six - Testing
Coming from an interpreted (scripting) language, you might well miss the REPL (read-eval-print-loop) as a means of immediate testing of small units of code. I strongly recommend setting up a testing module, whereby you can continue moreless the same excellent incremental development practice at the touch of the  'cargo test' button. With the added benefit of not ever having to re-write your tests. You can easily comment out those no longer needed and, in any case, they are nicely out of sight in a separate module, not cluttering up your code.

#### Station Seven - Contributing Your Own Crate
Still to come but I think I now have most of the components needed under my belt, having written my own internal libraries. As well as used the handy /// self documenting comments.

#### Station Eight - Error Handling
When I started writing this blog, it was all to be just about Rust's error handling. That is how big a thing it is!




