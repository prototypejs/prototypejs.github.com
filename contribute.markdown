---
layout: default
---

Contribute
==========

Prototype welcomes any (and all) of these:

* bug reports;
* failing tests proving some feature is broken in certain cases;
* patches/pull requests;
* enhancement requests;
* additional tests for code that you believe hasn’t got enough coverage;
* API documentation contributions;
* ideas and other interesting discussions.

<h3 id="tickets">Submitting tickets</h3>

The Prototype core team uses [GitHub issues](https://github.com/sstephenson/prototype/issues) to track tickets. If you don't have an account, it’s easy to sign up.

Try to keep the title concise, but write your heart out in the full description box. If you need to include a snippet of JavaScript code in your text, [mark it up appropriately](https://help.github.com/articles/github-flavored-markdown), like so:

    This line opens a popup every time, which scares me!
    
    ```javascript
    var scary = 'BOO!'
    window.alert(scary)
    ```
    
    Can Prototype be used to make this less scary? Thank you.
    
If you have a patch for the issue you’re describing, instead of filing a new issue you should probably make a pull request.

<h3 id="patches">Writing code</h3>

Prototype is on GitHub and strongly prefers that patches arrive in the form of [pull requests](https://help.github.com/articles/using-pull-requests). We use the fork-and-pull model ([as described here](https://help.github.com/articles/using-pull-requests#fork--pull)), thus, when you want to contribute a patch, you should:

  1. fork the main Prototype repository;
  2. create a [topic branch](http://stackoverflow.com/questions/284514/what-is-a-git-topic-branch) on your fork and commit some changes to that branch;
  3. make a pull request from that branch to the main repo’s master branch.
  
To work on Prototype development, you’ll need:

  1. the latest version of the Prototype source tree;
  2. your favorite code editor;
  3. all the modern web browsers your operating system can handle (to test in, of course);
  4. a recent version of [Ruby](http://ruby-lang.org) and its Rake library. Ruby is installed by default on OS X, but Windows users will need to use the [One-Click Ruby Installer](http://rubyinstaller.org). Rake is used to “build” the consolidated JavaScript file and to run our automated test suite across supported browsers.

To get started, clone the project to your disk, either from the main repo or from your fork. This will create a new directory named `prototype`. The most important directories in there are `src/`, in which all the code resides; and `test/`, which provides a testing framework as well as all the unit tests.

(Prototype relies on a few submodules, but it will try to fetch those submodules on demand. Or, instead, you can run `git submodule init` and `git submodule update`.)

While in the root directory of the project, run:

    rake dist

This predefined Rake task **merges all the source files** from `src/` to a single file: `dist/prototype.js`. Unit tests use this file.

To see what unit tests look like, open up any of the HTML documents found in `test/unit/` in your browser. They run automatically, so you should see a flurry of green-colored rows after a few moments.

Usually there should be *no test failures* if you are using a [supported browser](http://prototypejs.org/download).

To run specific tests in an automated fashion, you can use `rake test` to run the whole suite; or you can include a `TESTS` variable to run only certain test files.

To see tests fail, **try to break something on purpose.** Open `src/array.js` and find the method named `first` near the top:

    first: function() {
      return this[0];
    },
  
Just for the fun of it, change the return line to return a string (like "foo" or "hello!") and save the file. Then run…

    rake test TESTS=array BROWSERS=chrome
    
…to run _just_ the array test file in just a single browser (Chrome in this case, but you could specify `ie` or `firefox` or `safari` instead). A single test with a couple of assertions [should turn red](/assets/2007/1/15/array-fail.png). Congratulations, you have just broken Prototype! ;)

From here you're on your own. **Make changes to source files in `src/`, add your methods, go crazy.** When you're done, create a pull request on GitHub.

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

This will run Prototype's entire test suite (all unit tests) across all installed browsers: Safari, Firefox, IE, Opera, and Chrome (depending of your platform, of course). The tests run, page by page, and pass their results back to the terminal.

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

<h3 id="documentation">Writing documentation</h3>

Prototype uses [PDoc](http://pdoc.org) for documentation. PDoc looks at specially-formatted comments in the source code and extracts narrative documentation, then builds HTML around it. This is how we generate [the official documentation](http://api.prototypejs.org), but you can use this same process to generate similar documentation on your hard drive.

From the project root, run…

    rake doc
    
…and PDoc should build documentation for all of Prototype’s classes and methods. The documentation is placed in the `doc` subfolder, so you can open `doc/index.html` to view it.

This is useful if you’d like to have local documentation, or if you want to generate documentation for an older version of Prototype (just check out the tag you want, then run `rake doc`). It’s also useful if you’d like to help write documentation, since you‘re able to preview the changes you’ve made before you submit a pull request.