---
layout: default
title: To Rust or not to Rust?
date:   2020-05-27
categories: Errors, Rust programming
---
## To Rust or not to Rust?
### That is my Road to Golgotha
##### by Rustafarian

#### Introduction
I have always had this perverse interest in new, complicated and difficult programming languages. What could then pose a better challenge than taking up Rust? However, Rust also has some good  (even excellent) points to commend it: 

- Zero cost abstractions. All kinds of high level convenience constructs and the strong types  get compiled away into more or less the same native code as if they were not there.

- Speed of native code. I never believed in wasting time when it can be avoided. Rust's speed is comparable to C(++). 

- Simplicity of native code. No need to carry massive interpreter runtimes on your back. No need for having to install all kinds of strictly compatible libraries on your target machine (goodbye node). I can compile on my development machine and send the single file executable to the server (with the target architecture) and It Just Runs.

- Safe multi-threaded execution. Mutation (overwriting) and aliasing (via multiple pointers) can never apply to the same memory location at the same time. This is a big win, with most performance gains to be still fully realised and appreciated.

- Memory safety also means reliability and security.

- Relatively easy async/await constructs for handling asynchronous I/O. This is critical for net  connections, e.g. https, which are in terms of  CPU time extremely slow and it makes no sense to idly wait for them, blocking even a single thread.

#### Station One - Mutation and Ownership

The first thing that a new Rust user will notice is that preventing those aliasing mutations results in some fairly severe restrictions on one's programming style and the freedoms we used to enjoy. In particular, it is no longer possible to have a `left value` (something  that boils down to a writeable memory location) in lots of different places (or threads) and also to modify its contents anywhere at will. 

You have to explicitly mark as `mut` only those declared variables that absolutely must be overwritten,  pass them as `mut` parameters only to those functions where they actually are overwritten, and not from more than one place, even within the same block of code.

Mutability is actually quite a useful concept/distinction to make. A pleasant surprise was to find out just how many variables never really need to be mutable at all. After all, this has for long been the cornerstone of functional programming. Though variable  is perhaps no longer the best term. It has become an 'assign once' variable. Its only difference from constants is that its value is computed and assigned (once only) at runtime to a location on the stack.

My only gripe with this is that when you declare a lot of variables, the compiler remembers perfectly well which ones are mutable  but I often don't. I recommend adopting special case style, such as HolE CasE, so that they are immediately humanly recognisable. Akin to CAPITALS for constants.

You also have to worry about `ownership` to solve the aliasing problem. 'Ownership' confers overwriting rights to a mutable variable and there can only ever be just one owner. For example, you can  share several pointers to an immutable item (so called 'borrowing') but not to a mutable one, that one requires unique ownership. I would have probably preferred the time honoured terminology of 'passing a pointer' instead of 'borrowing'. Inventing new terms for existing concepts 'just to be different' only adds potential confusion. Perhaps it was meant to convey that borrowed items should not be destroyed. However, it fails to persuade, as those who had lent their Ferrari to their teenage offspring will confirm.

All of this means that initially the compiler will complain like mad but its error messages are fairly self-explanatory and often it will even tell you what it wants you to do. You will get compiler errors even after you grasped these principles but perhaps not quite so many.

#### Station Two - Lifetimes
I am leaving these to a future blog. Mostly because so far I have failed to get my head around the reasons why the lifetime annotations have to be quite so complicated (and yes, let us be blunt, so syntactically horrible). I just put them where the compiler tells me it wants them to be. Which is a horrible back-to-front approach but at the same time a testament to the Rust compiler's error reporting. With a better written program, they often go away anyway.

#### Station Three - Crates
Rust libraries are called crates and like everything else in Rust, they are remarkably safe, even to import from a vast collection by every Tom Dick and Harry at crates.io.

They can take ages to compile, due to a whole trees of dependencies of dependencies.

You must specify them in Cargo.toml at the top level of your project directory. Just beware that there are various characters, such as '^'  that when prepended before a crate's name can alter in mysterious ways how that crate is going to get updated. As does leaving out various digits from its version number. 

As if this was not complicated enough, some facilities of some crates are available only as their 'special features', which must be known by name and listed within square brackets, otherwise they will not work. This is annoying because typically somebody might avise you to use a particular crate but almost always forgets to tell you which of its 'special features' you need.

#### Station Four - Async I/O
This uses `futures` or place markers for some computation which is yet to finish. The concepts themselves are not difficult to grasp. Though you must beware that the results can come back in any unpredictable order. I like best the `!try_join(..)` macro and use it heavily to great effect.

That is not to say that I have not experienced a lot of practical difficulties. They were not so much with the concepts or the programming itself but rather with the state of the documentation and the crates, both being moving targets. It was not at all obvious which versions of which to use with which features and in what combinations, at what time. Different people used and recommended different crates in different ways and the same applied to the documentation. 

However, I believe that the situation has now somewhat settled away from 'the bleeding edge' and anyone embarking on this adventure now should find it a lot easier.

#### Station Five - Modules
This is one facility for which I have nothing but praise. Perhaps because the modules are so simple and Just Work. I have originally written my implementation as one largish file but once I have discovered the modules, it was a matter of minutes to split it into its logical parts. The whole project gained much needed structure and readability and perhaps even shorter recompilation times.

#### Station Six - Testing
Coming from an interpreted (scripting) language, you might well miss the REPL (read-eval-print-loop) as a means of immediate testing of small units of code. I strongly recommend setting up a testing module, whereby you can continue more or less the same excellent incremental development practice at the touch of the  'cargo test' button. With the added benefit of not ever having to re-write your tests. You can easily comment out those that are no longer needed and, in any case, they are nicely out of sight in a separate module, not cluttering up your main code.

#### Station Seven - Contributing Your Own Crate
Still to come but I think I now have most of the components needed under my belt, having written my own internal libraries, as well as used the handy /// self documenting comments.

#### Station Eight - Error Handling
When I first started writing this blog, it was all going to be just about error handling in Rust. That is how big a subject it is.  The difficulty is that in its generality, Rust admits to quite a few diverse styles and philosophies of error handling and most programmers have developed their own. 

Some just use `!panic` macro a lot, others write their own error handling helper functions to provide more debugging information in a more general way, still others laboriously write out detailed implementations of their own custom error types for everything imaginable, in mostly forlorn hope of making them 'recoverable'. I believe that the general guidance is to lean towards the former in simple stand-alone applications and towards the latter when writing reusable crates.

I myself had gone through several iterations of handling errors in different ways, mostly those in the middle of the road category. The Rust Book was not of much help in this regard. Yes, it does argue for error recovery but does not offer much practical help.  Then I came across the `anyhow` crate and I gladly threw all that previous rubbish away. 

Besides recoverable/unrecoverable, I find another classification of errors useful: 

- Hard errors, that is errors usually triggered by your own tests of some wrong/unexpected values, which ought to be reported immediately using the `!bail` macro from `anyhow` (or just `!panic` if you do not need much information and you just want to quit there and then).

- Errors returned from some function call, often to an external crate. These will be wrapped up in Result, so from purely technical point of view they need a different handling mechanism. These can be divided further into errors that:

	- only need a `static &str` error message. This can be embedded in `.context('such and such error happened');` 
	where the dot follows a Result type returned by some function, that could be either `Ok(some valid value)` or `Error`.
	Anyhow's `.context` thus acts rather like `.expect` but it automatically passes up its general `anyhow::Error`, whereas with .expect you are expected to choose/define your own error type and pass it up yourself. With that you might run into difficulties when several different types of Errors are  produced within the same function, such as IOError, HttpError, plus your own Error(s).
	
	- need some further processing, which might even include an immediate simple attempt at error recovery. Then use the `.with_context( some closure );` form. Here is an example of my usage in asynchronous I/O:
	
``` 
.await.with_context(
	|| format!("{}: getprice client-request failed at {},{}",
	file!(), line!(), column!()))?
```
		
The space between the vertical bars is for any arguments of the closure (lambda function).

#### Conclusion

Do not be put off by the initial struggles against the Rust compiler. The reward is that once you get that clean compilation out of it, it is actually very rare to get any runtime errors. Making all those anxious discussions about error handling somewhat academic.

There is no need to be afraid of Rust. When you are prepared to put some effort and application into it, it will repay you handsomely with safety and performance. 










