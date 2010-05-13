---
layout: default
---

Introduction to Ajax
====================

Prototype framework enables you to deal with Ajax calls in a very easy and fun way that is also safe (cross-browser). Besides simple requests, this module also deals in a smart way with JavaScript code returned from a server and provides helper classes for polling.

Ajax functionality is contained in the global `Ajax` object. The transport
for Ajax requests is _xmlHttpRequest_, with browser differences safely
abstracted from the user. Actual requests are made by creating instances
of the [`Ajax.Request`](/api/ajax/request) object.

    new Ajax.Request('/some_url', { method:'get' });

The first parameter is the URL of the request; the second is the options
hash. The _method_ option refers to the HTTP method to
be used; default method is POST.

<p class="notice">Remember that for security reasons (that is preventing cross-site scripting attacks) <em>Ajax requests can only be made to URLs of the same protocol, host and port of the page containing the Ajax request</em>. Some browsers might allow arbitrary URLs, but you shouldn't rely on support for this.</p>

### Ajax response callbacks

Ajax requests are by default _asynchronous_, which means you must have
callbacks that will handle the data from a response. Callback methods are
passed in the options hash when making a request:

    new Ajax.Request('/some_url',
      {
        method:'get',
        onSuccess: function(transport){
          var response = transport.responseText || "no response text";
          alert("Success! \n\n" + response);
        },
        onFailure: function(){ alert('Something went wrong...') }
      });

Here, two callbacks are passed in the hash that alert of either success or
failure; `onSuccess` and `onFailure` are called accordingly based on the
status of the response. The first parameter passed to both is the native
_xmlHttpRequest_ object from which you can use its `responseText` and
`responseXML` properties, respectively.

You can specify both callbacks, one or none - it's up to you. Other
available callbacks are:

* `onUninitialized`,
* `onLoading`,
* `onLoaded`,
* `onInteractive`,
* `onComplete` and
* `onException`.

They all match a certain state of the _xmlHttpRequest_ transport, except for
`onException` which fires when there was an exception while dispatching
other callbacks.

Also available are `onXXX` callbacks, where XXX is the HTTP response status
like 200 or 404. Be aware that, when using those, your `onSuccess` and
`onFailure` won't fire because `onXXX` takes precedence, therefore using these
means you know what you're doing.

<p class="notice">The <code>onUninitialized</code>, <code>onLoading</code>, <code>onLoaded</code>, and <code>onInteractive</code> callbacks are not implemented consistently by all browsers. In general, it's best to avoid using these.</p>


### Parameters and the HTTP method

You can pass the parameters for the request as the `parameters` property in
options:

    new Ajax.Request('/some_url', {
      method: 'get',
      parameters: {company: 'example', limit: 12}
      });

Parameters are passed in as a hash (preferred) or a string of key-value pairs
separated by ampersands (like `company=example&limit=12`).

You can use parameters with both GET and POST requests. Keep in mind,
however, that GET requests to your application should never cause data
to be changed. Also, browsers are less likely to cache a response to a POST
request, but more likely to do so with GET.

One of the primary applications for the parameters property is sending the contents
of a FORM with an Ajax request, and Prototype gives you a helper method for this, called [`Form.serialize`](/api/form/serialize):

    new Ajax.Request('/some_url', {
      parameters: $('id_of_form_element').serialize(true)
      });

If you need to push custom HTTP request headers, you can do so with the
`requestHeaders` option. Just pass name-value pairs as a hash or in a flattened
array, like: `['X-Custom-1', 'value', 'X-Custom-2', 'other value']`.

If, for some reason, you have to POST a request with a custom post body
(not parameters from the `parameters` option), there is a `postBody` option
exactly for that. Be aware that when using `postBody`, parameters passed
will never be posted because `postBody` takes precedence as a body - using the
option means you know what you're doing.


### Evaluating a JavaScript response

Sometimes the application is designed to send JavaScript code as a response. If
the content type of the response matches the MIME type of JavaScript then this
is true and Prototype will automatically `eval()` returned code. You don't need
to handle the response explicitly if you don't need to.

Alternatively, if the response holds a X-JSON header, its content will be
parsed, saved as an object and sent to the callbacks as the second argument:

    new Ajax.Request('/some_url', { method:'get',
      onSuccess: function(transport, json){
          alert(json ? Object.inspect(json) : "no JSON object");
        }
      });

Use this functionality when you want to fetch non-trivial data with Ajax but
want to avoid the overhead of parsing XML responses. JSON is much faster
(and lighter) than XML.


### Global responders

There is an object that is informed about every Ajax request:
`Ajax.Responders`. With it, you can register callbacks that will fire on a
certain state of any `Ajax.Request` issued:

    Ajax.Responders.register({
      onCreate: function(){
        alert('a request has been initialized!');
      }, 
      onComplete: function(){
        alert('a request completed');
      }
    });

Every callback matching an _xmlHttpRequest_ transport state is allowed here,
with an addition of `onCreate`. Globally tracking requests like this can be
useful in many ways: you can log them for debugging purposes using a
JavaScript logger of your choice or make a global exception handler that
informs the users of a possible connection problem.


### Updating your page dynamically with `Ajax.Updater`

Developers often want to make Ajax requests to receive HTML fragments that
update parts of the document. With `Ajax.Request` with an `onComplete`
callback this is fairly easy, but with [`Ajax.Updater`](/api/ajax/updater) it's even easier!

Suppose you have this code in your HTML document:

    language: html
    &lt;h2&gt;Our fantastic products&lt;/h2&gt;
    &lt;div id=&quot;products&quot;&gt;(fetching product list ...)&lt;/div&gt;

The 'products' container is empty and you want to fill it with HTML returned
from an Ajax response. No problem:

    new Ajax.Updater('products', '/some_url', { method: 'get' });

That's all, no more work. The arguments are the same of `Ajax.Request`,
except there is the receiver element in the first place. Prototype
will automagically update the container with the response using the
`Element.update()` method.

If your HTML comes with inline scripts, they will be stripped by default.
You'll have to pass `true` as the `evalScripts` option in order to see
your scripts being executed.

But what if an error occurs, and the server returns an error message instead of
HTML? Often you don't want insert errors in places where users expected content.
Fortunately, Prototype provides a convenient solution: instead of the actual
container as the first argument you can pass in a hash of 2 different
containers in this form: `{ success:'products', failure:'errors' }`. Content
will be injected in the _success_ container if all went well, but errors will be
written to the _failure_ container. By using this feature your interfaces can
become much more user-friendly.

You might also choose not to overwrite the current container contents, but
insert new HTML on top or bottom like you would do with `Insertion.Top` or
`Insertion.Bottom`. Well, you can. Just pass the insertion object as the
`insertion` parameter to Ajax:

    new Ajax.Updater('products', '/some_url', {
      method: 'get',
      insertion: Insertion.Top
      });

`Ajax.Updater` will use the given object to make the insertion of returned
HTML in the container ('products') element. Nifty.


### Automate requests with the `Ajax.PeriodicalUpdater`

You find the `Ajax.Updater` cool, but want to run it in periodical intervals
to repeatedly fetch content from the server? Prototype framework has that,
too - it's called [`Ajax.PeriodicalUpdater `](/api/ajax/periodicalupdater), and basically it's running
`Ajax.Updater` at regular intervals.

    new Ajax.PeriodicalUpdater('products', '/some_url',
      {
        method: 'get',
        insertion: Insertion.Top,
        frequency: 1,
        decay: 2
      });

Two new options here are `frequency` and `decay`. Frequency is the interval in
seconds at which the requests are to be made. Here, it's 1 second, which means
we have an Ajax request every second. The default frequency is 2 seconds. Our
users might be happy because the responsiveness of the application, but our
servers might be taking quite a load if enough people leave their browsers open
on the page for quite some time. That's why we have *the decay option* - it is
the factor by which the frequency is multiplied every time when current
response body is the same as previous one. First Ajax request will then be made
in 1 second, second in 2, third in 4 seconds, fourth in 8 and so on. Of course,
if the server always returns different content, decay will never take effect;
this factor only makes sense when your content doesn't change so rapidly and
your application tends to return the same content over and over.

Having frequency falloff can take the load off the servers considerably because
the overall number of requests is reduced. You can experiment with this factor
while monitoring server load, or you can turn it off completely by passing
1 (which is default) or simply omitting it.


### Move along

Learn more about [`Ajax.Request`](/api/ajax/request), [`Ajax.Updater`](/api/ajax/updater) and [`Ajax options`](/api/ajax/options).

