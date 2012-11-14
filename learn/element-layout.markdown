---
layout: default
title_header: true
title: "Element.Layout: Your digital tape measure"
site_section: documentation
---

Why is it so hard to measure element dimensions accurately? For the same reason it's hard to do _anything_ in browser JavaScript: the APIs are inconsistent and ill-suited to our needs.

Prototype 1.7 introduced [`Element.Layout`](http://api.prototypejs.org/dom/Element/Layout/), an attempt to give element measurements a sane and consistent API that's suited to common tasks.

### The problem

There are two main ways to measure elements on a web page. One is to use the old Internet Explorer–style properties like `offsetWidth` and `clientHeight`:

{% highlight javascript %}
$('troz').offsetWidth;  //-> 50
$('troz').clientHeight; //-> 100
{% endhighlight %}

Reading from these properties is rather performant and gives us actual pixel values in numbers. But it's awfully hard to remember which properties correspond to which dimensions ("offset"? "client"?) and they're of no use if you need to measure something other than width or height.

The other common way is to read computed CSS styles:

{% highlight js %}
$('troz').getStyle('width');  //-> '50px'
$('troz').getStyle('height'); //-> '100px'
{% endhighlight %}

Computed styles are excellent for measuring oddball things like borders or padding. But they're much more annoying to work with. They return strings with units, so you've got to call `parseInt` to turn them into numbers. And older versions of Internet Explorer don't support _computed styles_, whose measurements are always in pixel units. They only support `currentStyle`, which will report dimensions in whatever units are specified in the applicable CSS for the element. That means we're not guaranteed to get pixel values back.

Enough of this. Let's fix it in code.

### The solution

We want a system with the upsides of both approaches. That means it should:

1. give us results in numbers representing pixel values;
2. allow us to measure any aspect of an element's box;
3. work identically no matter which browser we're using.

Enter [`Element.Layout`](http://api.prototypejs.org/dom/Element/Layout/).

`Element.Layout` is a subclass of [`Hash`](http://api.prototypejs.org/language/Hash/). Imagine a hash with predefined keys, things like `height` and `width` and `padding-bottom` and `border-left` and `top` and `left`. Imagine that all of these keys have pixel values, and that the actual measuring is done the very first time you try to read a given property.

Got it? Let's jump to some use cases before I explain more.

#### The simple case

If you want a one-off measurement of an element, use [`Element#measure`](http://api.prototypejs.org/dom/Element/prototype/measure/):

{% highlight js %}
$('troz').measure('width'); //-> 150
$('troz').measure('border-box-width'); //-> 180
$('troz').measure('border-top'); //-> 5

// Offsets, too:
$('troz').measure('top'); //-> 226
{% endhighlight %}

The measurement names are intuitive; most of them are derived from their CSS equivalents. So `width` means the width of the content box, without margin and padding, just like in CSS. But `border-box-width` means the width of the element from the left edge of its left border to the right edge of its right border. This approach gives you far more granularity than the old `offsetWidth`-style DHTML properties.

These measurements are **guaranteed to be in pixels, even in Internet Explorer**.

#### The complex case

If you need to measure several things at once, though, `Element#measure` is not the most efficient way to do it. Behind the scenes, elements sometimes need a bit of manipulation before they'll report accurate dimensions, so it's best to try to minimize that manipulation cost.

First, use [`Element#getLayout`](http://api.prototypejs.org/dom/Element/prototype/getLayout/) to obtain an instance of `Element.Layout`:

{% highlight js %}
var layout = $('troz').getLayout();
{% endhighlight %}

Now use [`Element.Layout#get`](http://api.prototypejs.org/dom/Element/Layout/prototype/get/) to retrieve values, using the same property names you used for `Element#measure`:

{% highlight js %}
layout.get('width');  //-> 150
layout.get('height'); //-> 500

layout.get('padding-left');  //-> 10
layout.get('margin-left');   //-> 25
layout.get('padding-top');    //-> 10
layout.get('padding-bottom'); //-> 10

layout.get('border-box-width'); //-> 180
layout.get('padding-box-height'); //-> 520

layout.get('width');  //-> 150
{% endhighlight %}

Here's where the remembered values (or _memoization_, if you prefer) come in. When I ask for `width`, Prototype measures the element – which, as we discussed, is a costly operation — and returns a value. A few lines later, I ask for `width` again, and I get the same value. But this time it didn't do any measuring. It remembered the value from last time.

There's more. When I ask for `padding-box-height`, Prototype knows that's just `height` plus `padding-top` plus `padding-bottom`. All three of those properties are already memoized, since I asked for them earlier, so it skips the measurement phase and just gives me the sum.

You can also get an instance of `Element.Layout` with all properties pre-measured by passing a boolean `true` as the first argument to `getLayout`:

{% highlight js %}
var layout = $('troz').getLayout(true);
{% endhighlight %}

How does it know when an element's dimensions change? It doesn't. Don't hang onto an instance of `Element.Layout` for too long; it's meant for short-term efficiency, not long-term caching. You can grab a new instance by calling `Element#getLayout` again.

#### Measuring hidden elements

There are several common use cases for being able to measure elements that are hidden with `display: none`. So `Element.Layout` supports this: you can measure elements that are hidden, _as long as their parents are visible_. (Like when you want to animate an element from a hidden state and need to know how tall it will be.)

This is difficult because browsers don't typically report accurate dimensions for hidden elements. To get around this, Prototype does some extra magic: it switches `display: none` to `visibility: hidden`, applies `position: absolute` to remove the element from the ordinary layout flow, then removes `display: none` so that the browser will give us the numbers we want. Then it reverses the process.

Thus the constraint that the element's parent must be visible. If it weren't, we'd have to do all this magic for the element's parent as well as the element itself, and so on up the tree until we got to a visible ancestor. The cost of this approach, both in performance and complexity, is prohibitive.

### Further reading

For more, visit the documentation pages for [`Element.Layout`](http://api.prototypejs.org/dom/Element/Layout/) and [`Element#measure`](http://api.prototypejs.org/dom/Element/prototype/measure).

