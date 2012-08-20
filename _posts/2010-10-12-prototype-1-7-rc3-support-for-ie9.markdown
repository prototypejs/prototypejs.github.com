---
layout: blog_archive
author: Andrew Dupont
author_url: http://andrewdupont.net/
sections: Fearued, Releases, blog
title: "Prototype 1.7 RC3: Support for IE9"
digest: "Release Candidate 3 of Prototype 1.7 is now out. This long-delayed version includes full support for Internet Explorer 9."
---

It was five months ago that we released RC2, and it was true that we were planning to release 1.7 final soon after. Two problems intervened:

### Element.getDimensions

To minimize code duplication, we had redefined `Element#getDimensions` (and the related methods `getWidth` and `getHeight`) to use the new and preferred `Element.Layout` methods for measuring dimensions. This logic, while more robust, produced slightly different results in a few cases, so we decided to revert to the previous implementation of `Element#getDimensions`.

If you're writing new code, we encourage you to use `Element.Layout` — e.g., `someNode.measure('border-box-width')` instead of `someNode.getWidth()`. You'll get more accurate results in general. But we're leaving `Element#getDimensions` the way it is to ensure that people don't have to rewrite existing code.

### Internet Explorer 9

The far more important reason for putting the brakes on 1.7 was the impending release of Internet Explorer 9. In short, we didn't want to put out something that would break in large ways when IE9 was released. That meant waiting for a preview release of IE9 that was stable enough (and representative enough of the final product) to test against.

The IE9 beta passes all but one of our unit tests. The one minor failure is the result of an issue on Microsoft's end; we hear it will be fixed before the final release of IE9.

### Other changes

To learn about new features in version 1.7, refer to the [blog post about the 1.7 RC1 release](http://prototypejs.org/2010/4/5/prototype-1-7-rc1-sizzle-layout-dimensions-api-event-delegation-and-more).

As always, this release includes an assortment of bug fixes. Consult the CHANGELOG for further details, or [see the full diff between this release and the previous release candidate](http://github.com/sstephenson/prototype/compare/1.7_rc2...1.7_rc3) (RC2 to RC3).

### What now?

Now that we've got these two issues solved, we're eager to get the final release out the door. It'll free me up to lend more time to [script.aculo.us 2.0](http://scripty2.com) (which just saw a beta release) and its nascent UI components.

### Download, report bugs, and get help

* [Download Prototype 1.7 RC3](http://prototypejs.org/assets/2010/10/12/prototype.js)
* [View the API documentation](http://api.prototypejs.org)
* [Check out the Prototype source code](http://github.com/sstephenson/prototype/) on GitHub
* [Submit bug reports](https://prototype.lighthouseapp.com/projects/8886-prototype/overview) to Lighthouse
* [Get Prototype help](/discuss) on the mailing list or #prototype IRC channel
* [Talk to the core team](http://groups.google.com/group/prototype-core) on the prototype-core mailing list

Thanks to the many contributors who made this release possible!
