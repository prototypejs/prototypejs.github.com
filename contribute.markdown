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

<h4 id="unit_tests">Unit Tests</h4>

To see what unit tests look like, open up the `test/unit/tests` directory and look at any of the `*.test.js` files.

To start up a server for testing, run:

    rake test:start
    
and open a browser window to the instructed URL, likely `http://127.0.0.1:4567/test/`. The tests will take a few moments to run, but when they're done, you _should_ see a bunch of passed tests and no failed tests — if you're using a [supported browser](http://prototypejs.org/download), that is.

If you want to run only some of the files you see in `test/unit/tests`, you can put them in the URL — `http://127.0.0.1:4567/test/ajax,string` will run only the tests in `ajax.test.js` and `string.test.js`.

If you want to run tests in an automated fashion, you can start up the server with `rake test:start`, then run

    rake test:run
    
in a second terminal window. The `test:run` task should run the tests against all the browsers which Prototype supports and which are installed on your system.

You can tell the runner to use only specific browsers or to run only specific tests. For instance:

    rake test:run BROWSERS=firefox,chrome TESTS=ajax,string
    
will run the tests from `ajax.test.js` and `string.test.js` in both Firefox and Chrome and report the results back to you.

To see tests fail, **try to break something on purpose.** Open `src/lang/array.js` and find the method named `first` near the top:

    first: function() {
      return this[0];
    },
  
Just for the fun of it, change the return line to return a string (like "foo" or "hello!") and save the file, then run `rake dist` to build a new distributable.

Now start the test server and open `http://127.0.0.1:4567/test/array` in a browser of your choice. You should see that the `#first` test is now failing.

From here you're on your own. **Make changes to source files in `src/`, add your methods, go crazy.** When you're done, create a pull request on GitHub.

Some coding guidelines:

  * two spaces, no tabs;
  * keep variable names nice and descriptive;
  * don't forget to pick low-hanging performance fruit in common constructs such as loops;
  * follow the conventions already laid out in the code.
  
We use [Mocha](http://mochajs.org) for testing along with [Proclaim](https://github.com/rowanmanning/proclaim)'s TDD-style assertions. (We add some of our own assertions for convenience; check out `test/unit/static/js/test_helpers.js` to see what's been added.)


<h3 id="testing">The importance of testing</h3>

With unit testing we ensure adding new features will not break old ones. We also ensure that the library behaves the same way across all supported browsers.

When you're submitting a pull request, please also submit tests to cover the code you've written. With rare exception, **patches won't get applied if they don't have accompanying tests**; if you write the tests yourself (instead of waiting for someone else to write them for you) you vastly increase your patch's chance of acceptance.

Patches that _introduce new features_ should have test cases that demonstrate the new functionality.

Patches that _fix bugs_ should include test cases in order to document what has been fixed. For this reason, your test cases should **fail** without your patch and **pass** after it has been applied.


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