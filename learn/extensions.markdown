---
layout: default
---

How Prototype extends the DOM
=============================

The biggest part of the Prototype framework are its DOM extensions. Prototype adds many convenience methods to elements returned by the `$()` function: for instance, you can write `$('comments').addClassName('active').show()` to get the element with the ID 'comments', add a class name to it and show it (if it was previously hidden). The 'comments' element didn't have those methods in native JavaScript; how is this possible? This document reveals some clever hacks found in Prototype.

### The Element.extend() method

Most of the DOM methods are encapsulated by the `Element.Methods` object and
then copied over to the `Element` object (for convenience). They all receive the
element to operate with as the first parameter:

    Element.hide('comments');
    var div_height = Element.getHeight(my_div);
    Element.addClassName('contactform', 'pending');

These examples are concise and readable, but we can do better. If you have an
element to work with, you can pass it through `Element.extend()` and it will
copy all those methods directly to the element. Example, to create an element
and manipulate it:

    var my_div = document.createElement('div');
    	
    Element.extend(my_div);
    my_div.addClassName('pending').hide();
    	
    // insert it in the document
    document.body.appendChild(my_div);

Our method calls just got shorter and more intuitive! As mentioned before,
`Element.extend()` copies all the methods from `Element.Methods` to our element
which automatically becomes the first argument for all those functions. The
`extend()` method is smart enough not to try to operate twice on the same
element. What's even better, **the dollar function `$()` extends every element
passed through it** with this mechanism.

In addition, `Element.extend()` also applies `Form.Methods` to FORM elements
and `Form.Element.Methods` to INPUT, TEXTAREA and SELECT elements:

    var contact_data = $('contactform').serialize();
    var search_terms = $('search_input').getValue();

Note that not only the dollar function automatically extends elements!
`Element.extend()` is also called in `document.getElementsByClassName`,
`Form.getElements`, on elements returned from the `$$()` function (elements
matching a CSS selector) and other places - in the end, chances are you will
rarely need to explicitly call `Element.extend()` at all.


### Adding your own methods with Element.addMethods()

If you have some DOM methods of your own that you'd like to add to those of
Prototype, no problem! Prototype provides a mechanism for this, too. Suppose
you have a bunch of functions encapsulated in an object, just pass the object
over to `Element.addMethods()`:

    var MyUtils = {
    	truncate: function(element, length){
    		element = $(element);
    		return element.update(element.innerHTML.truncate(length));
    	},
    	updateAndMark: function(element, html){
    		return $(element).update(html).addClassName('updated');
    	}
    }
    	
    Element.addMethods(MyUtils);
    	
    // now you can:
    $('explanation').truncate(100);

The only thing to watch out here is to make sure the first argument of these methods is the element itself. In your methods, you can also return the element in the end to allow for chainability (or, as practiced in the example, any method which itself returns the element).


### Native extensions

There is a secret behind all this.

In browsers that support adding methods to prototype of native objects such as 
`HTMLElement` *all* DOM extensions on the element are available by
default without ever having to call `Element.extend()`, dollar function or
anything! This will then work in those browsers:

    var my_div = document.createElement('div');
    my_div.addClassName('pending').hide();
    document.body.appendChild(my_div);

Because the prototype of the native browser object is extended, all DOM elements
have Prototype extension methods built-in. This, however, isn't true for IE
which doesn't let anyone touch `HTMLElement.prototype`. To make the previous
example work in IE you would have to extend the element with `Element.extend()`.
Don't worry, the method is smart enough not to extend an element more than once.

Because of browsers that don't support this you must take care to use DOM
extensions only on elements that have been extended. For instance, the example
above works in Firefox and Opera, but add `Element.extend(my_div)` after
creating the element to make the script really solid. You can use the dollar
function as a shortcut like in the following example:

    // this will error out in IE: 
    $('someElement').parentNode.hide()
    // to make it cross-browser:
    $($('someElement').parentNode).hide()

Don't forget about this! Always test in all the browsers you plan to support.
