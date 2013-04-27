---
layout: post
title: "Async sequences using jQuery.Deferred"
date: 2013-04-27 12:18
comments: true
categories: 
---
At work, I've been coding on a project which has to wrangle a lot of
asynchronous processes. At first, I stuck with the Node.js convention
of using callbacks accepting `(error, data)` arguments, but I found
that tended to make the code a little circuitous. As the project
in question would largely be used on the front-end, I didn't feel I
really had to stick to those conventions, so I looked into
[jQuery's Deferred object](http://api.jquery.com/category/deferred-object/),
and especially its use of [promises](http://api.jquery.com/promise/).

This isn't a post about promises, so I won't go too deeply into what
they are, but suffice to say they offer an alternative way to manage
asynchrony, with an arguably cleaner syntax; the `Deferred` object
also simplifies chaining asynchronous functions, and provides a way to
signal when a group of functions has completed (when their `deferred`s
have `resolve`d, in the parlance). This is
[jQuery.when](http://api.jquery.com/jQuery.when/).

`when` is very useful; you pass it some functions (each of which must call
`Deferred.resolve` on completion), and it tells you when they're all
complete. Only problem is that the functions are called in
parallel. Now, as a default behaviour, this makes perfect sense: we
are talking about asynchrony after all. But in some cases, we want to
ensure that our functions execute in a particular sequence, each
function awaiting the completion of its
predecessor. `jQuery.Deferred`, so far as I was able to find out,
doesn't provide for such a use case.

There are of course other libraries which *do* --
[Async.js](https://github.com/caolan/async), for example -- but I'd
thrown in my lot with jQuery, and I was reluctant to import a
whole library for just this small and apparently simple piece of
functionality.

Now, at work, we're lucky enough to be using
[CoffeeScript](http://coffeescript.org) for almost all our
Javascript-y business, and the solution to my problem turned out to be
quite a beautiful little five-liner:

```coffeescript
sequence = (tasks) ->
    seq = tasks[0]()
    for task, i in tasks[1..]
        seq = do (i) -> seq.then -> tasks[i + 1]()
    seq
```

So, what's happening here? Well, we pass an array of tasks (functions
which implement `Deferred.resolve`, and return a `promise`) to `sequence`. Sequence then
executes the *first* task, and iterates over the rest, each time
calling `Deferred.then` on the new value of `seq`; the function passed
to `then` calls the next task, which returns a `promise`, and so
on. The sequence itself is returned; this value is in fact also a
`promise` -- the one returned by the last function in the sequence --
which allows you to act on the sequence's completion.

Note the use of CoffeeScript's `do` keyword, which creates an
immediately-executing function. This is necessary to preserve the
scope of the index variable in the function passed to `then`;
were we not to enclose the call to `then` in an outer function, `i`
would remain scoped to the `i` given in the loop declaration. When the
function using `i` was called, it would of course take the value of `i`
*at the time of being called*, not at the time of the function's
creation -- which value is most likely to be the value of `i` at the
end of the loop.

Of course, the simplicity of this code is bought at the expense of
error handling, of which there's none: you'd at least want to check
you had an array of length > 0, to avoid exceptions, and it would make
sense to avoid the iteration altogether if the array were only of
length 1. Nevertheless, I think it's a nice demonstration of both
CoffeeScript's concision and `$.Deferred`'s flexibility.

