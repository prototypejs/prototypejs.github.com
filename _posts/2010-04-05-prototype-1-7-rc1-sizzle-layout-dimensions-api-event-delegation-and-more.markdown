---
layout: blog_archive
author: Andrew Dupont
author_url: http://andrewdupont.net/
sections: Featured, Releases, blog
title: "Prototype 1.7 RC1: Sizzle, layout/dimensions API, event delegation, and more"
digest: "We've just tagged the first release candidate of Prototype 1.7: a major new version with some major new features."
---

We've just tagged the first release candidate of Prototype 1.7: a major new version with some major new features.

### Sizzle as the selector engine (or mix in your own)

With Prototype 1.7, we've finally realized our long-held goal of moving to [Sizzle](http://sizzlejs.com/ "Sizzle JavaScript Selector Library"), the middleware selector engine used by [jQuery](http://jquery.com/ "jQuery: The Write Less, Do More, JavaScript Library") and others. I wrote our previous selector engine, used since 1.5.1, but nevertheless I'm excited to switch to a more robust engine that's shared between frameworks.

So Sizzle is the new default. But there's more to it than that. In moving to Sizzle, we've modularized the selector engine entirely. If you want to use Diego Perini's [NWMatcher](http://javascript.nwbox.com/NWMatcher/ "NWMatcher - CSS3 Selector and Matcher") library in place of Sizzle, you can. Just check out the source code and build like so:

    rake dist SELECTOR_ENGINE=nwmatcher
    
If you're a sentimentalist, you can use the legacy Prototype selector engine by specifying `SELECTOR_ENGINE=legacy_selector`. Or add your own selector engine by creating a subdirectory in `vendor/` and following [some simple conventions](http://github.com/sstephenson/prototype/tree/master/vendor/legacy_selector/ "vendor/legacy_selector at master from sstephenson's prototype - GitHub").


### Element#on

[`Element#on`](http://api.prototypejs.org/dom/element/on/) is a new way to access the Prototype event API. It provides first-class support for event delegation and simplifies event handler removal.

In its simplest form, `Element#on` works just like `Element#observe`:

{% highlight js %}
$("messages").on("click", function(event) {
  // ...
});
{% endhighlight %}

An optional second argument lets you specify a CSS selector for event delegation. This encapsulates the pattern of using `Event#findElement` to retrieve the first ancestor element matching a specific selector. So this Prototype 1.6 code...

{% highlight js %}
$("messages").observe("click", function(event) {
  var element = event.findElement("a.comment_link");
  if (element) {
    // ...
  }
});
{% endhighlight %}
  
...can be written more concisely with `Element#on` as:

{% highlight js %}
$("messages").on("click", "a.comment_link", function(event, element) {
  // ...
});
{% endhighlight %}
  
`Element#on` differs from `Element#observe` in one other important way: its return value is an object with a `#stop` method. Calling this method will remove the event handler. (Technically, this is an instance of a new class called [`Event.Handler`](http://api.prototypejs.org/dom/event/handler/).) With this pattern, there's no need to retain a reference to the handler function just so you can pass it to `Element#stopObserving` later.

For example, in Prototype 1.6, where you'd need to write something like...

{% highlight js %}
start: function() {
  this.clickHandler = function(event) {
    // ...
  };

  $("messages").observe("click", this.clickHandler);
},

stop: function() {
  $("messages").stopObserving("click", this.clickHandler);
}
{% endhighlight %}
  
...you can now write:

{% highlight js %}
start: function() {
  this.clickHandler = $("messages").on("click", function(event) {
    // ...
  });
},

stop: function() {
  this.clickHandler.stop();
}
{% endhighlight %}

Also note that the `Event.Handler` class has a corresponding `#start` method that lets you re-attach an observer you've removed with `#stop`.

So, to review, `Element#on` is _both_ a new approach to event observation _and_ an implementation of event delegation. Feel free to eschew `Element#observe` and use `Element#on` exclusively; or use `Element#on` just for event delegation; or keep using `Element#observe` the way you always have.


### Element.Layout: Your digital tape measure

The second major feature in 1.7 is `Element.Layout`, a class for pixel-perfect measurement of element dimensions and offsets.

Now you don't have to decide between properties like `offsetWidth` (which return numbers, but not the numbers you _want_) or retrieving computed styles (which have their own set of quirks and require a call to `parseInt`).

#### The simple case

If you want a one-off measurement of an element, use the new [`Element#measure`](http://api.prototypejs.org/dom/element/measure/):

{% highlight js %}
$('troz').measure('width'); //-> 150
$('troz').measure('border-top'); //-> 5

// Offsets, too:
$('troz').measure('top'); //-> 226
{% endhighlight %}
    
The argument passed to `measure` is one of a handful of intuitive names, most of which are derived from their CSS equivalents. So `width` means the width of the content box, just like in CSS — but we throw in extra properties (e.g., `padding-box-width`, `margin-box-height`) for some common measurements. This approach gives you far more granularity than common DHTML properties like `offsetWidth` and `clientHeight`.

These measurements are guaranteed to be in pixels. Even in IE. (In fact, Prototype works around a handful of IE quirks that would ordinarily result in inaccurate measurments.) It can even measure elements that are _hidden_, as long as their parents are visible. (Like when you want to animate an element from a hidden state and need to know how tall it will be.)

#### The complex case

If you need to measure several things at once, though, `Element#measure` is not the most efficient way to do it. Often an element will need a bit of manipulation before it reports its dimensions accurately, which means measurements can be costly.

The `Element.Layout` class tries to minimize that cost. It's a read-only subclass of `Hash` that remembers values in order to avoid re-computing. 

First, use [`Element#getLayout`](http://api.prototypejs.org/dom/element/layout/) to obtain an instance of `Element.Layout`:

{% highlight js %}
var layout = $('troz').getLayout();
{% endhighlight %}
    
Now use `Element.Layout#get` to retrieve values, using the same property names you used for `Element#measure`:

{% highlight js %}
layout.get('width');  //-> 150
layout.get('height'); //-> 500

layout.get('padding-left');  //-> 10
layout.get('margin-left');   //-> 25
layout.get('border-top');    //-> 5
layout.get('border-bottom'); //-> 5

layout.get('padding-box-width'); //-> 170
layout.get('border-box-height'); //-> 510

layout.get('width');  //-> 150
{% endhighlight %}

Here's where the remembered values (or _memoization_, if you prefer) come in. When I ask for `width`, Prototype measures the element – which, as we discussed, is a costly operation — and returns a value. A few lines later, I ask for `width` again, and I get the same value. But this time it didn't do any measuring. It remembered the value from last time.

There's more. When I ask for `border-box-height`, Prototype knows that's just `height` plus `border-top` plus `border-bottom`. All three of those properties are already memoized, since I asked for them earlier, so it skips the measurement phase and just gives me the sum.
    
How does it know when an element's dimensions change? It doesn't. Don't hang onto an instance of `Element.Layout` for too long; it's meant for short-term efficiency, not long-term caching. You can grab a new instance by calling `Element#getLayout` again.

Believe it or not, this is the short version. [Read the documentation](http://api.prototypejs.org/dom/element/layout/) to learn more.


### JSON fixes, ES5 compliance

The JSON interface slated for ECMAScript 5 is already being implemented in major browsers. It uses many of the same method names as Prototype's existing JSON implementation, but with different behavior, so we rewrote ours to be ES5-compliant and to fall back to the native JSON support where possible. A few other methods, like `Object.keys`, received similar treatment.


### And, of course, bug fixes

Consult the [CHANGELOG](http://github.com/sstephenson/prototype/blob/1.7_rc1/CHANGELOG) for further details.

### Download, report bugs, and get help

* [Download Prototype 1.7 RC1](http://prototypejs.org/assets/2010/4/1/prototype.js)
* [View the API documentation](http://api.prototypejs.org)
* [Check out the Prototype source code](http://github.com/sstephenson/prototype/) on GitHub
* [Submit bug reports](https://prototype.lighthouseapp.com/projects/8886-prototype/overview) to Lighthouse
* [Get prototype help](/discuss) on the mailing list or #prototype IRC channel
* [Talk to the core team](http://groups.google.com/group/prototype-core) on the prototype-core mailing list

As always: thanks to the many contributors who made this release possible!