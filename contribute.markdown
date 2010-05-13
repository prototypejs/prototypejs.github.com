---
layout: default
---

Contribute
==========

Think you found a bug? Got some handy code you think would fit awesomely in the framework? Or do you have what it takes to optimize some important method &mdash; or even a whole module &mdash; making it more flexible, powerful or speedy? Then prove your JavaScript-fu by contributing to Prototype!

We welcome any (and all!) of these:

* bug reports;
* failing tests proving some feature is broken in certain cases;
* code patches;
* enhancement requests and patches;
* additional tests for code that you believe hasn't got enough coverage;
* API documentation contributions;
* ideas and other interesting discussions on the mailing lists or in the IRC channel.

<h3 id="tickets">Submitting tickets</h3>

The Prototype core team uses [Lighthouse as its issue tracker](http://prototype.lighthouseapp.com/projects/8886/home). You have to [register first](http://prototype.lighthouseapp.com/users/new?to=%2Fprojects%2F8886-prototype%2Ftickets%2Fnew) to [open a new ticket](http://prototype.lighthouseapp.com/projects/8886-prototype/tickets/new), but it only takes a second (and it's free).

Try to keep the title sef-descriptive and concise, but write your heart out in full description box. If you need to include a snippet of JavaScript code in your text, [mark it up appropriately](http://lighthouseapp.com/help/text-formatting), like so:

    This line opens a popup every time, which scares me!
    
    @@@ javascript
    var scary = 'BOO!'
    window.alert(scary)
    @@@
    
    Can Prototype be used to make this less scary? Thank you.

Just don't use this feature to paste actual patches; upload them to your ticket as attachments.

<h3 id="patches">Writing code</h3>
  
To contribute patches and tests, in most cases you will need:

1. the latest version of Prototype (ideally using a [Git](http://git.or.cz/) client, but [tarballs are also available](http://github.com/sstephenson/prototype/tree/master));
2. your favorite code editor;
3. all the modern web browsers your operating system can handle (to test in, of course);
4. A recent version of [Ruby](http://ruby-lang.org) and its Rake library. Ruby is installed by default on OS X, but Windows users will need to use the [One-Click Ruby Installer](http://rubyinstaller.rubyforge.org/wiki/wiki.pl). Rake is used to "build" the consolidated JavaScript file and to run our automated test suite across supported browsers.
  
**Start by cloning the project from our repository:**

    git clone git://github.com/sstephenson/prototype.git
    
This will create a new directory named `prototype`. The most important directories in there are `src/`, in which all the code resides; and `test/`, which provides a testing framework as well as all the unit tests.

While in the root directory of the project, write

    rake dist

This predefined Rake task **merges all the source files** from `src/` to a single file: `dist/prototype.js`. Unit tests use this file. To see what unit tests look like, open up any of the HTML documents found in `test/unit/` in your browser. They run automatically, so you should see a flurry of green-colored rows after a few moments.

Usually there should be *no test failures* if you are using a [supported browser](http://prototypejs.org/download).

To see tests fail, **try to break something on purpose.** Open `src/array.js` and find the method named `first` near the top:

    first: function() {
      return this[0];
    },
  
Just for the fun of it, change the return line to return a string (like "foo" or "hello!") and save the file. Build the `prototype.js` file again by issuing the `rake dist` command. Then open `test/unit/array.html` in a browser. A single test with a couple of assertions [should turn red](/assets/2007/1/15/array-fail.png). Congratulations, you have just broken Prototype! ;)

From here you're on your own. **Make changes to source files in `src/`, add your methods, go crazy.** When you're done, create a new ticket on Lighthouse. Don't tell us which lines to change and don't paste changed source; you should *attach* a patch (sometimes called "a diff file") to your ticket. If you're using Git, you can create a patch like this:

    # first be sure to commit your changes locally
    git commit -a -m "fixed a bug regarding the DOM"

    # make the patch file(s)
    git format-patch origin

This will make one patch file (prefixed with numbers) for each commit that you did locally. That's fine. Upload the files to your ticket. (If you're not using Git, there are other ways to generate diffs. They vary depending on your platform. Google is your friend.)

**Alternatively to all this,** if you are a GitHub user as well, you can fork our project, push your changes there and send Sam (`sstephenson`) a [pull request](http://github.com/guides/pull-requests) via the GitHub interface.

Some coding guidelines:

  * two spaces, no tabs;
  * keep variable names nice and descriptive;
  * don't forget to pick low-hanging performance fruit in common constructs such as loops;
  * follow the conventions already laid out in the code.


<h3 id="testing">The importance of testing</h3>

With unit testing we ensure adding new features will not break [old ones](/assets/2007/1/15/array-assertions-light.png). We also ensure that the library behaves the same way across all supported browsers. When you're submitting a patch you really **should** submit tests, also. That way we know you mean business and that we can safely roll it into core. With rare exception, **patches won't get applied if they don't have accompanying tests**; if you write the tests yourself (instead of waiting for someone else to write them for you) you vastly increase your patch's chance of acceptance.

Patches that _introduce new features_ should have test cases that demonstrate the new functionality.

Patches that _fix bugs_ should include test cases in order to document what has been fixed. For this reason, your test cases should **fail** without your patch and **pass** after it has been applied.

Before you test your changes, first be sure you've got the latest version of Prototype:

    git pull
    
Git will pull in any changes made to Prototype since you last grabbed the source. Then, run `rake dist` so that the changes you made in `src/` get built into the distributable.

Finally, run the tests:

    rake test

This will run Prototype's entire test suite (all unit tests) across all installed browsers: Safari, Firefox, IE, Opera, and Konqueror (depending of your platform, of course). The tests run, page by page, and pass their results back to the terminal.

If any of the tests fail, you'll need to figure out why, make the changes, and run the tests again. To avoid re-running the whole test suite, you can limit the testing task to certain browsers and/or certain groups of tests:

    rake test BROWSERS=firefox TESTS=ajax,string

*Note*: Most tests can be run outside of `rake` simply by opening the corresponding HTML file in a browser. But some tests (in particular, those in `ajax.js`) rely on client-server communication. Rake faciliates these tests by spawning a local web server.

The unit testing library itself bears similarity's to Ruby's `Test::Unit` and allows for a broad range of different types of assertions. Here's a [presentation about it](http://mir.aculo.us/2006/9/16/adventures-in-javascript-testing).

<h3 id="mistakes">Common mistakes</h3>

We've seen enough of these:

* Very *application-specific* requests/patches: frameworks don't try to cover every scenario ever imagined, so try to concentrate on more commonly used stuff;
* bug reports that don't describe how to reproduce the issue (give us detailed steps and *tell us which browsers exhibit the bug!*);
* tickets that claim things are broken when, in fact, the reporter has simply forgotten to `git pull` or `rake dist`;
* patches made without regard to execution speed: don't slow down important code to add a feature of marginal utility;
* patches that aren't cross-browser: test in at least 2 browsers, Firefox and another one;
* duplicate requests or patches: search Lighthouse to find out if someone has beat you to it.

<h3 id="compress">Prototype.js doesn't compress well! Can I submit a patch to fix that?</h3>

There is nothing wrong with our code, but with your JS compression software. You can change your local copy of the framework to your liking and compress as much as you want, but we don't support it ourselves. What we recommend is good gzip settings on your webserver; the framework is less than 30 kB gzipped.

If gzip isn't an option for you, several people maintain unofficial compressed versions of Prototype and/or [script.aculo.us](http://script.aculo.us). Google will point you in the right direction.

We also don't accept patches that change our code simply because it generated warnings from JSLint or Firefox's strict checking mode. Code linters check for idioms and constructs that _might_ be problematic; as a result, they generate scads of false positives.

<h3 id="docs">API documentation enhancements</h3>

You are welcome to contribute API documentation snippets if you think we missed something. **We write the documentation in [Markdown](http://daringfireball.net/projects/markdown/) format.** As a template you might want to use this source for the [`Element#getElementsBySelector`](/api/element/getElementsBySelector) method:

    ## getElementsBySelector

    language: ebnf
    getElementsBySelector(element, selector...) -> [HTMLElement...]
        

    Takes an arbitrary number of CSS selectors (strings) and returns a
    document-order array of [extended](/api/element/extend) children of `element`
    that match any of them.

    ----------------------------------------------------------------------------

    This method is very similar to [$$()](/api/utility/dollar-dollar) but can be
    used within the context of one element, rather than the whole document. The
    supported CSS syntax is identical, so please refer to the
    [`$$()` docs](/api/utility/dollar-dollar) for details.

    ### Examples

    $('apples').getElementsBySelector('[title="yummy!"]');
        // -> [h3, li#golden-delicious, li#mutsu]
    
        $('apples').getElementsBySelector( 'p#saying', 'li[title="yummy!"]');
        // -> [h3, li#golden-delicious, li#mutsu,  p#saying]
    
        $('apples').getElementsBySelector('[title="disgusting!"]');
        // -> []
        

    ### Tip

    `Element#getElementsBySelector` can be used as a pleasant alternative to
    `getElementsByTagName`.
