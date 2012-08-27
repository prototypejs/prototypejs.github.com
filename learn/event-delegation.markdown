---
layout: default
title: "Event delegation in Prototype 1.7"
title_header: true
site_section: documentation
---

Event delegation is an advanced technique for event-driven programming. The idea: instead of attaching one listener onto each element in a group, attach one listener onto an ancester shared by all the elements in that group. Then, when the listener is triggered, determine whether to act by looking at which element originally received the action.

### The approach

For example, let's say I've got a bunch of list items and I'd like to apply some sort of click effect to each one.

{% highlight html %}
<div id="items">
  <ul>
    <li>lorem</li>
    <li>ipsum</li>
    <li>dolor</li>
  </ul>
</div>
{% endhighlight %}

I could do this:

{% highlight js %}
$$('#items li').each( function(item) {
  item.observe('click', function(event) {
    doSomethingWith(event.target);
  });
});
{% endhighlight %}

But there are a couple of problems with this approach. First of all, if I've got a _lot_ of list items — pretend we're dealing with 300 instead of three — attaching that many event listeners will have performance implications. Second, if I ever need to replace the contents of `div#items` dynamically, I'm going to have to go back and re-attach all those event listeners.

Instead, let's try this:

{% highlight js %}
$('items').observe('click', function(event) {
  if (event.target.tagName === 'LI') {
    doSomethingWith(event.target);
  }
});
{% endhighlight %}

Now we're listening in _one_ place — the container `div`. Our handler checks to see if a list item was clicked on, and only then does it do the thing we want.

So by doing slightly more work _inside_ the handler, we're able to make our application more performant and more adaptable.

Prototype 1.7 has extracted this idea into a pattern. [`Element#on`](http://api.prototypejs.org/dom/Event/on/) is a new way to use the Prototype event API. It provides first-class support for event delegation and simplifies event handler removal.

It lets us write the above example far more concisely:

{% highlight js %}
$('items').on('click', 'li', function(event, element) {
  doSomethingWith(event.element);
});
{% endhighlight %}

### How it works

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
  
`Element#on` differs from `Element#observe` in one other important way: its return value is an object with a `#stop` method. Calling this method will remove the event handler. (Technically, this is an instance of a new class called [`Event.Handler`](http://api.prototypejs.org/dom/Event/Handler/).) With this pattern, there's no need to retain a reference to the handler function just so you can pass it to `Element#stopObserving` later.

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
