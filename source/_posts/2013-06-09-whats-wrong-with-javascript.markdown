---
layout: post
title: "What's wrong with JavaScript"
date: 2013-06-09 14:11
comments: true
categories: programming
---
*(Updated 2013-06-10 with some corrections and fine-tunings)*

So from my [last post](/blog/2013/05/11/mod-lang/) you might guess I'm
an advocate for JavaScript. Well, I have more time for the language
than many Rubyists seem to: it has a nicely compact API and, once
you've learned to avoid the potholes, it's an easy language to get
things done in. CoffeeScript makes it considerably less painful to
read and write, and the Node ecosystem is generating some very useful
toolsets and ways of working.

There are, though, some significant pain-points involved in writing
non-trivial programs in JavaScript. Some of them are well-covered in
the literature already, but there are some that seem not to be. I'm
interested to gauge the extent to which others have come up against some
of these, so here's my take on what's *really* wrong with JavaScript.

##The obvious stuff
I'm not going to dwell on any of this, because it's [well-trodden territory](http://javascript.crockford.com/survey.html),
but to get it out of the way: the scoping is wacky (and the `var` trap
is a disaster); the comparison operators (`==` vs `===`) are a kludge;
and the treatment of the `this` keyword is tiresome. There are many
more dark corners (see, for instance, [Gary Bernhardt's WAT](http://www.youtube.com/watch?v=kXEgk1Hdze0)), but all
of these issues can be worked around more or less easily (and in fact
CoffeeScript largely does so). There are other issues with the
language that I'd argue are harder to deal with -- in some cases,
practically impossible.

##What is an object?
So, famously, JavaScript is not a class-based language. It uses
prototypal inheritance. This is a somewhat simpler type of
object-oriented language design, whereby object instances inherit both data and
behaviour from other object instances. It can work quite well (it's a
very memory-efficient system if used properly), and it can to some
extent be used to mimic class-based inheritance if that's what you
want. The problem with JavaScript, though, is what it uses for
'objects'.

In JavaScript, an 'object' is effectively just a key--value structure. It is what
in other languages is called a hash, a map or a dictionary. It looks like this:

```javascript
var obj = {
    foo: "I'm a property!",
    bar: function() {
        return "And I'm a method!"
    },
    baz: function() {
        return this.bar();
    }
}
```

We could either use this object as-is, in which case it's what in a
class-based language we'd call a Singleton, or we can use it as the
prototype for another object instance (there are several ways to
accomplish this -- they're covered elsewhere, so I won't expand).

As you can see, there's no distinction between a 'method' (a member of
the object which is a function) and a 'property'. Everything is just a
value indexed on a key. Nice and simple, but take a CoffeeScript
instance method like this one:

```coffeescript
thing: ->
  @thing ?= new Thing
```

This looks pretty and idiomatic: it's an accessor method which
assigns `@thing` if it doesn't already exist, then returns it.
See the problem?

CoffeeScript's syntax is helping to hide the fact that we've bound the
instance variable to the same key as the method (`this.thing`). I hope
this seems obvious to you, and that you're wondering how anyone could
be so stupid as to do such a thing. I am here to tell you that this is
something that a busy developer will do (especially one familiar with
Ruby), and that it will then manifest a very subtle bug which does not
immediately show up in unit tests. I will leave you to speculate as to
how I can be so sure about this.

Now, sure, we could be sensible and create `setThing` and `getThing`
methods, but, you know, we're not writing Java. We're in a highly
dynamic language. We want nice things. Or, we could create setters and
getters using the
[new ES5 syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/get)
-- but that's not available everywhere (pre-IE9, for instance), and, I
would argue, is too unwieldy to apply as a rule.

So what we're left with is doing something like:

```coffeescript
thing: ->
  @_thing ?= new Thing
```

Which is ... irritating. But if we want this kind of method syntax,
that's what we have to do, and we have to be vigilant that we don't
introduce mistakes like the above, because no lint tool will catch them.

It's tempting to presume that this is a 'feature' of prototypal
inheritance; it's not. The two next best-known prototypal languages,
[Self](http://selflanguage.org/_static/tutorial/Language/ObjectsAndSlots/ObjectsAndSlots.html) and [Lua](http://www.lua.org/pil/16.html#ObjectSec), distinguish properties and methods just fine.

##Everything is an object
And that's a good thing, right?

Well, first, it's not really true of JavaScript: you need to ['box'](http://blogs.adobe.com/webplatform/2012/08/27/javascript-types/)
primitives like numbers to get them to behave like objects, but fair
enough, many languages do something similar. Second, and more
importantly, see above as regards what objects in JavaScript *actually
are*. What we're really saying is 'everything is a key--value structure'. Is that
a good thing?

No. It isn't. Imagine another hypothetical CoffeeScript method:

```coffeescript
copyObject: (inputObject) ->
  copy = []
  for key, val of inputObject
    copy[key] = val
```

Spot the deliberate mistake? Busy developer has initialised `copy` to
an array rather than an object. The really bad news? *This method will
work almost as if it were correct*. We can get every key of
`inputObject` on `copy` correctly, because an array, in JavaScript, is
an object, and therefore *is just a key--value structure*. We can set
(pretty much) any key on it we like, and get it back intact. But
again, subtle bugs abound; arrays are not intended to be used like
this, and things will go wrong if, for example, you start trying to
iterate over `copy`'s properties, expecting it to be a 'real' object.

Ouch.

##Nothing is an object
Try this, in a CoffeeScript class:

```coffeescript
bindClick: ->
  $("#clickable").on "click", @onClick

respondToClick: ->
  window.alert "clicked!"

onClick: ->
  @respondToClick()
```

So presuming `bindClick` has been called, what happens when we click
on `"#clickable"`?

`respondtoClick` will never be called, and you'll almost certainly get
an exception. Remember, there are no methods in Javascript, only
functions. We bound the function `this.onClick` to the click event;
that function has no idea that it's also a method. Why should a value
in a key--value structure know which structure it belongs to? In the
context of the function `this.onClick`, `this` is the function itself
or, sometimes, the `window` object. Neither of these has a
`respondToClick` method, so we get an exception.

In short, we can call any function in scope, from anywhere, regardless of
whether it's part of an 'object'; and attaching a function to an object as
a method only binds `this` correctly *when the function is called in
the context of its parent object*.

The correct way to do the above is of course:

```coffeescript
bindClick: ->
  $("#clickable").on "click", => @respondToClick()
```

We pass an anonymous function to `.on`, explicitly binding `this` to
our current context using CoffeeScript's `=>` operator.

##_missing
All of the above can be worked around or avoided; at
present, one very important thing can't be. JavaScript has no
equivalent to [Ruby's `method_missing`](http://ruby-doc.org/core-2.0/BasicObject.html#method-i-method_missing), [Smalltalk's
`doesNotUnderstand`](http://c2.com/cgi/wiki?DoesNotUnderstand), [Objective-C's `forwardInvocation`](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtForwarding.html), [PHP's `__call`](http://php.net/manual/en/language.oop5.overloading.php),
etcetera. You can't catch a call to an undefined method in JavaScript
and then do something with it. This is, again, to do with the fact
that there's nothing to distinguish 'properties' from 'methods' in
JavaScript; plenty of 'properties' won't be callable as functions, but
that doesn't necessarily mean they *should be*.

It's important to be able to do this in traditional object-oriented
design; we need it to do proper [delegation](http://en.wikipedia.org/wiki/Delegation_(programming), for example. It's also a
tremendously convenient feature if you want to do metaprogramming
(e.g. adding properties or methods to an object dynamically at
runtime). There are two ways to sort-of do it right now: one is
[Mozilla's `__noSuchMethod__`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/noSuchMethod), which is only supported in SpiderMonkey
and likely to be deprecated; the other is the 'official' ECMAScript
proposal, [Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy?redirectlocale=en-US&redirectslug=JavaScript%2FReference%2FGlobal_Objects%2FProxy), which is characteristically circuitous and
counterintuitive. It can be [turned on in V8](http://stackoverflow.com/questions/10665892/enable-harmony-proxies-in-nodejs) (and thus in Node), but
you'd be pretty far out on a limb to use it.

##Finally
JavaScript was designed in a hurry, and it wasn't designed
to build large, long-lived, maintainable applications. The fact that
people are using it to do so is a testament to the ingenuity of
serious JavaScript users, and their insight that what JavaScript
*does* offer makes the failings tolerable.

They are significant failings, though. I've yet to hear anyone
convincingly argue that JavaScript's version of OOP offers anything in
addition to or distinction from more traditional versions; it's a kind
of Heisenbergian OOP, which breaks down as soon as you stop pretending
it's there. Similarly, I'm not convinced that the changes being made
(and proposed) are the right ones. They make the language seem more
complex -- and this is a language whose simplicity is its virtue.
