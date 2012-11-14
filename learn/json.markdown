---
layout: default
title_header: true
title: "Introduction to JSON"
---


JSON (JavaScript Object Notation) is a lightweight data-interchange format (if you are new to JSON, you can read more about it on the [JSON website](http://www.json.org)). It is notably used by APIs all over the web and is a fast alternative to XML in Ajax requests. Prototype 1.5.1 finally features JSON encoding and parsing support.

Prototype's JSON implementation is largely based on the work of [Douglas Crockford](http://www.crockford.com/) which will most likely be natively included in future versions of the main browsers. [Crockford's implementation](http://json.org/json.js) is unfortunately unsuitable for use with Prototype because of the way it extends `Object.prototype`. (Note that this will no longer be an issue once it is natively implemented.)

### Encoding

Prototype's JSON encoding slightly differs from Crockford's implementation as it does not extend `Object.prototype`. The following methods are available: `Number#toJSON`, String#toJSON, Array#toJSON, [`Hash#toJSON`](http://api.prototypejs.org/language/Hash/prototype/toJSON/), [`Date#toJSON`](http://api.prototypejs.org/language/Date/prototype/toJSON/) and [`Object.toJSON`](http://api.prototypejs.org/language/Object/toJSON/).

If you are unsure of what type the data you need to encode is, your best bet is to use `Object.toJSON` like so:

{% highlight js %}
var data = {name: 'Violet', occupation: 'character', age: 25 };
Object.toJSON(data);
//-> '{"name": "Violet", "occupation": "character", "age": 25}'
{% endhighlight %}

In other cases (i.e. if you know that your data is _not_ an instance of `Object`), you can invoke the `toJSON` method instead:

{% highlight js %}
$H({name: 'Violet', occupation: 'character', age: 25 }).toJSON();
//-> '{"name": "Violet", "occupation": "character", "age": 25}'
{% endhighlight %}

Furthermore, if you are using custom objects, you can set your own `toJSON` method which will be used by `Object.toJSON`. For example:

{% highlight js %}
var Person = Class.create({
  initialize: function(name, age) {
    this.name = name;
    this.age = age;
  },  
  toJSON: function() {
    return ('My name is ' + this.name + 
      ' and I am ' + this.age + ' years old.').toJSON();
  }
});
var john = new Person('John', 49);
Object.toJSON(john);
//-> '"My name is John and I am 49 years old."'
{% endhighlight %}

Finally, using [`Element.addMethods`](http://api.prototypejs.org/dom/Element/addMethods/) you can create custom `toJSON` methods targeted at specific elements.

{% highlight html %}
<input id="firstname" name="firstname" value="john" />
{% endhighlight %}

{% highlight js %}
Element.addMethods('input', {
  toJSON: function(element) {
    element = $(element);
    return element.name.toJSON() + ": " + element.getValue().toJSON();
  }
})
    
Object.toJSON($('firstname'));
//-> '"firstname": "john"'
{% endhighlight %}



### Parsing JSON

In JavaScript, parsing JSON is typically done by evaluating the content of a JSON string. Prototype introduces [`String#evalJSON`](http://api.prototypejs.org/language/String/prototype/evalJSON/) to deal with this:

{% highlight js %}
var data = '{ "name": "Violet", "occupation": "character" }'.evalJSON();
data.name;
//-> "Violet"
{% endhighlight %}

[`String#evalJSON`](http://api.prototypejs.org/language/String/prototype/evalJSON/) takes an optional `sanitize` parameter, which, if set to `true`, checks the string for possible malicious attempts and prevents the evaluation and throws a `SyntaxError` if one is detected.

{% highlight js %}
var data = 'grabUserPassword()'.evalJSON(true)
//-> SyntaxError: Badly formated JSON string: 'grabUserPassword()'
{% endhighlight %}

<p class="notice">You should always set the <code>sanitize</code> parameter to <code>true</code> and an appropriate content-type header (<code>application/json</code>) for data coming from untrusted sources (external or user-created content) to prevent XSS attacks.</p>

[`String#evalJSON`](http://api.prototypejs.org/language/String/prototype/evalJSON/) internally calls [`String#unfilterJSON`](http://api.prototypejs.org/language/String/prototype/unfilterJSON/) and automatically removes optional security comment delimiters (defined in `Prototype.JSONFilter`).

{% highlight js %}
person = '/*-secure-\n{"name": "Violet", "occupation": "character"}\n*/'.evalJSON()
person.name;
//-> "Violet"
{% endhighlight %}

<p class="notice">You should always set security comment delimiters (<code>/*-secure-\n...*/</code>) around sensitive JSON or JavaScript data to prevent Hijacking. (See <a href="http://www.fortifysoftware.com/servlet/downloads/public/JavaScript_Hijacking.pdf">this PDF document</a> for more details.)</p>

### Using JSON with Ajax

Using JSON with Ajax is very straightforward, simply invoke [`String#evalJSON`](http://api.prototypejs.org/language/String/prototype/evalJSON/) on the transport's `responseText` property:

{% highlight js %}
new Ajax.Request('/some_url', {
  method:'get',
  onSuccess: function(transport){
     var json = transport.responseText.evalJSON();
   }
});
{% endhighlight %}

If your data comes from an untrusted source, be sure to sanitize it:

{% highlight js %}
new Ajax.Request('/some_url', {
  method:'get',
  requestHeaders: {Accept: 'application/json'},
  onSuccess: function(transport){
    var json = transport.responseText.evalJSON(true);
  }
});
{% endhighlight %}
