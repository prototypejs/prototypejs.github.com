---
layout: default
title: "Defining classes and inheritance"
title_header: true
site_section: documentation
---

In early versions of Prototype, the framework came with basic support for
class creation: the [`Class.create()`](/api/class/create) method. Until now the
*only* feature of classes defined this way was that the constructor called a
method called `initialize` automatically.

**Prototype 1.6.0** now features a richer class system that's backward-compatible and adds some new features.

The cornerstone of class creation in Prototype is still
the [`Class.create`](http://api.prototypejs.org/language/Class/create/) method. If you have legacy Prototype code, your class-based code will continue to work as before, but now you don't have to work with prototypes directly or use [`Object.extend`](http://api.prototypejs.org/language/Object/extend/) to copy properties around.


### Example

Suppose you have legacy Prototype class-based code that looks like htis:

{% highlight js %}
/** obsolete syntax **/
    
var Person = Class.create();
Person.prototype = {
  initialize: function(name) {
    this.name = name;
  },
  say: function(message) {
    return this.name + ': ' + message;
  }
};
    
var guy = new Person('Miro');
guy.say('hi');
// -> "Miro: hi"
    
var Pirate = Class.create();
// inherit from Person class:
Pirate.prototype = Object.extend(new Person(), {
  // redefine the speak method
  say: function(message) {
    return this.name + ': ' + message + ', yarr!';
  }
});
    
var john = new Pirate('Long John');
john.say('ahoy matey');
// -> "Long John: ahoy matey, yarr!"
{% endhighlight %}

Observe the direct interaction with class prototypes and the clumsy inheritance
technique using `Object.extend`. Also, with `Pirate` redefining the `say()`
method of `Person`, there is no way of calling the overridden method like you can
do in programming languages that support class-based inheritance.

This has changed for the better. Compare the above with:

{% highlight js %}
/** new, preferred syntax **/
    
// properties are directly passed to `create` method
var Person = Class.create({
  initialize: function(name) {
    this.name = name;
  },
  say: function(message) {
    return this.name + ': ' + message;
  }
});
    
// when subclassing, specify the class you want to inherit from
var Pirate = Class.create(Person, {
  // redefine the speak method
  say: function($super, message) {
    return $super(message) + ', yarr!';
  }
});
    
var john = new Pirate('Long John');
john.say('ahoy matey');
// -> "Long John: ahoy matey, yarr!"
{% endhighlight %}

You can see how both class and subclass definitions are shorter because you
don't need to hack their prototypes directly anymore. We have also demonstrated
another new feature: `Pirate#say` "supercalls" the `say` method of its `Person` superclass.

### The `$super` argument in method definitions

When you override a method in a subclass, but still want to be able to call the
original method, you will need a *reference* to it. You can obtain that
reference by defining those methods with an extra argument in the front:
`$super`. Prototype will detect this and make the overridden method available to
you through that argument. But _to the outside world_, the `Pirate#say` method still expects a single argument. Keep this in mind.

### How to mix-in modules

So far you have seen the general way of calling `Class.create`.

{% highlight js %}
var Pirate = Class.create(Person, { /* instance methods */ });
{% endhighlight %}

But in fact, `Class.create` takes in an arbitrary number of arguments. If the first argument is a class, it will serve as that function's superclass. Any subsequent arguments (or _all_ arguments, should the first argument not be a class) are treated as "mixins"; their function properties are extended onto the object in the order they're listed.

This can conveniently be used for mixing in modules:

{% highlight js %}
// define a module
var Vulnerable = {
  wound: function(hp) {
    this.health -= hp;
    if (this.health < 0) this.kill();
  },
  kill: function() {
    this.dead = true;
  }
};
    
// the first argument isn't a class object, so there is no inheritance ...
// simply mix in all the arguments as methods:
var Person = Class.create(Vulnerable, {
  initialize: function() {
    this.health = 100;
    this.dead = false;
  }
});
    
var bruce = new Person;
bruce.wound(55);
bruce.health; //-> 45
{% endhighlight %}


### Types of inheritance in programming languages

Generally we distinguish between class-based and prototypal inheritance, the
latter being specific to JavaScript.

Prototypal inheritance, of course, is a very useful feature of the language, but
is often verbose when you are actually creating your objects. This is why we
emulate class-based inheritance (as in the Ruby language) through prototypal inheritance internally. This has certain implications.

For instance, in PHP you can define the initial values for instance variables:

{% highlight php %}
<?php
class Logger {
  public $log = array();
    
  function write($message) {
    $this->log[] = $message;
  }
}
    
$logger = new Logger;
$logger->write('foo');
$logger->write('bar');
$logger->log; // -> ['foo', 'bar']
?>
{% endhighlight %}

We can try to do the same in Prototype:

{% highlight js %}
var Logger = Class.create({
  initialize: function() { },
  log: [],
  write: function(message) {
    this.log.push(message);
  }
});
    
var logger = new Logger;
logger.log; // -> []
logger.write('foo');
logger.write('bar');
logger.log; // -> ['foo', 'bar']
{% endhighlight %}


It works. But what if we make **another** instance of Logger?


{% highlight js %}
var logger2 = new Logger;
logger2.log; // -> ['foo', 'bar']
    
// ... hey, the log should have been empty!
{% endhighlight %}

You can see that, although you may have expected an empty array in the new
instance, it has the same array as the previous logger instance. In fact, *all* 
logger objects will share the same array object because it is copied by reference, not by value.

Instead, **initialize your instances with default values:**

{% highlight js %}
var Logger = Class.create({
  initialize: function() {
    // this is the right way to do it:
    this.log = [];
  },
  write: function(message) {
    this.log.push(message);
  }
});
{% endhighlight %}


### Defining class methods

There is no special support for class methods in Prototype 1.6.0. Simply define
them on your existing classes:

{% highlight js %}
Pirate.allHandsOnDeck = function(n) {
  var voices = [];
  n.times(function(i) {
    voices.push(new Pirate('Sea dog').say(i + 1));
  });
  return voices;
}
    
Pirate.allHandsOnDeck(3);
// -> ["Sea dog: 1, yarr!", "Sea dog: 2, yarr!", "Sea dog: 3, yarr!"]
{% endhighlight %}

If you need to define several at once, simply use `Object.extend` as before:

{% highlight js %}
Object.extend(Pirate, {
  song: function(pirates) { ... },
  sail: function(crew) { ... },
  booze: ['grog', 'rum']
});
{% endhighlight %}

When you inherit from Pirate, the class methods are *not* inherited. This feature may be added in future versions of Prototype. Until then, copy the methods manually, but *not* like this:

{% highlight js %}
var Captain = Class.create(Pirate, {});
// this is wrong!
Object.extend(Captain, Pirate);
{% endhighlight %}

Class constructors are Function objects and it will mess them up if you just
copy everything from one to another. In the best case you will still end up
overriding `subclasses` and `superclass` properties on `Captain`, which is not
good.

### Special class properties

Prototype 1.6.0 defines two special class properties: `subclasses` and
`superclass`. Their names are self-descriptive: they hold references to
subclasses or superclass of the current class, respectively.

{% highlight js %}
Person.superclass
// -> null
Person.subclasses.length
// -> 1
Person.subclasses.first() == Pirate
// -> true
Pirate.superclass == Person
// -> true
Captain.superclass == Pirate
// -> true
Captain.superclass == Person
// -> false
{% endhighlight %}

These properties are here for easy class introspection.

### Adding methods on-the-fly with Class#addMethods

<div class="deprecated" style="margin-bottom:1em"><code>Class#addMethods</code> was named <code>Class.extend</code> in Prototype 1.6 RC0.<br />Please update your code accordingly.</div>

Imagine you have an already defined class you want to add extra methods to. In JavaScript's native prototypal inheritance, this can be accomplished by adding the methods onto one of the objects in the class's prototype chain. With Prototype, the easiest way to do this is to use `Class#addMethods`:

{% highlight js %}
var john = new Pirate('Long John');
john.sleep();
// -> ERROR: sleep is not a method
    
// every person should be able to sleep, not just pirates!
Person.addMethods({
  sleep: function() {
    return this.say('ZzZ');
  }
});
    
john.sleep();
// -> "Long John: ZzZ, yarr!"
{% endhighlight %}

The `sleep` method is now available not only on new Person instances, but
on instances of its subclasses, including any instances that had already been created.
