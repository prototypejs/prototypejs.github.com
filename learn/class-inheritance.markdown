---
layout: default
---

Defining classes and inheritance
================================

In the early versions of Prototype, the framework came with basic support for
class creation: the [`Class.create()`](/api/class/create) method. Until now the
*only* feature of classes defined this way was that the constructor called a
method called `initialize` automatically. **Prototype 1.6.0** now comes with
inheritance support through the `Class` module, which has taken several steps
further since the last version; you can make richer classes in your code with
more ease than before.

The cornerstone of class creation in Prototype is still
the [`Class.create()`](/api/class/create) method. With the new version of the
framework, your class-based code will continue to work as before; only now you
don't have to work with object prototypes directly or use
[`Object.extend()`](/api/object/extend) to copy properties around.

### Example

Let's compare the old way and a new way of defining classes and inheritance in
Prototype:

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

Observe the direct interaction with class prototypes and the clumsy inheritance
technique using `Object.extend`. Also, with `Pirate` redefining the `say()`
method of `Person`, there is no way of calling the overridden method like you can
do in programming languages that support class-based inheritance.

This has changed for the better. Compare the above with:

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

You can see how both class and subclass definitions are shorter because you
don't need to hack their prototypes directly anymore. We have also demonstrated
another new feature: the "supercall", or calling the overridden method (done with
special keyword `super` in the Ruby language) in `Pirate#say`.

### How to mix-in modules

So far you have seen the general way of calling `Class.create`.

    var Pirate = Class.create(Person, { /* instance methods */ });

But in fact, `Class.create` takes in an arbitrary number of arguments. The first&mdash;if it is another class&mdash;defines that the new class should inherit from it. All other arguments are added as instance methods; internally they are subsequent calls to `addMethods` (see below). This can conveniently be used for mixing in modules:

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

### The `$super` argument in method definitions

When you override a method in a subclass, but still want to be able to call the
original method, you will need a *reference* to it. You can obtain that
reference by defining those methods with an extra argument in the front:
`$super`.  Prototype will detect this and make the overridden method available to
you through that argument. But, from outside world the `Pirate#say` method still
expects a single argument; this implementation detail doesn't affect how your
code interacts with the objects.

### Types of inheritance in programming languages

Generally we distinguish between class-based and prototypal inheritance, the
latter being specific to JavaScript. While upcoming JavaScript 2.0 will support true
class definitions, current versions of the JavaScript language implemented in
modern browsers are restricted to prototypal inheritance.

Prototypal inheritance, of course, is a very useful feature of the language, but
is often too verbose when you are actually creating your objects. This is why we
are emulating class-based inheritance (like in the Ruby language) through
prototypal inheritance internally. This has certain implications. For instance,
in PHP you can define the inital values for instance variables:

    language: php
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

We can try to do the same in Prototype:

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

It works. But what if we make **another** instance of Logger?

    var logger2 = new Logger;
    logger2.log; // -> ['foo', 'bar']
    
    // ... hey, the log should have been empty!

You can see that, although some of you expected an empty array in the new
instance, we have the same array as the previous logger. In fact, *all* logger
objects will share the same array object because it is copied by reference, not
by value. **The correct way is to initialize your instances to default values:**

    var Logger = Class.create({
      initialize: function() {
        // this is the right way to do it:
        this.log = [];
      },
      write: function(message) {
        this.log.push(message);
      }
    });

### Defining class methods

There is no special support for class methods in Prototype 1.6.0. Simply define
them on your existing classes:

    Pirate.allHandsOnDeck = function(n) {
      var voices = [];
      n.times(function(i) {
        voices.push(new Pirate('Sea dog').say(i + 1));
      });
      return voices;
    }
    
    Pirate.allHandsOnDeck(3);
    // -> ["Sea dog: 1, yarr!", "Sea dog: 2, yarr!", "Sea dog: 3, yarr!"]

If you need to define several at once, simply use `Object.extend` as before:

    Object.extend(Pirate, {
      song: function(pirates) { ... },
      sail: function(crew) { ... },
      booze: ['grog', 'rum']
    });

When you inherit from Pirate, the class methods are *not* inherited. This feature may be added in future versions of Prototype. Until then, copy the methods manually,
but *not* like this:

    var Captain = Class.create(Pirate, {});
    // this is wrong!
    Object.extend(Captain, Pirate);

Class constructors are Function objects and it will mess them up if you just
copy everything from one to another. In the best case you will still end up
overriding `subclasses` and `superclass` properties on `Captain`, which is not
good.

### Special class properties

Prototype 1.6.0 defines two special class properties: `subclasses` and
`superclass`. Their names are self-descriptive: they hold references to
subclasses or superclass of the current class, respectively.

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

These properties are here for easy class introspection.

### Adding methods on-the-fly with `Class#addMethods()`

<div class="deprecated" style="margin-bottom:1em"><code>Class#addMethods</code> was named <code>Class.extend</code> in Prototype 1.6 RC0.<br />Please update your code accordingly.</div>

Imagine you have an already defined class you want to add extra methods to.
Naturally, you want those methods to be instantly available on subclasses and
every existing instance in the memory! This is accomplished by injecting a
property in the prototype chain, and the safest way to do it with Prototype is
to use `Class#addMethods`:

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

The `sleep` method was instantly available not only on new Person instances, but
on its subclasses and existing instances currently in memory.
