---
layout: default
title: "Upgrading to the latest stable release"
title_header: true
site_section: documentation
---

If you've put off the task of upgrading your old code to the latest stable release of Prototype, now you're out of excuses. Since version 1.6, Prototype comes with with an <a href="/assets/2008/4/22/prototype_update_helper.js">update helper</a> that will warn you of any deprecations or API changes.

The script is meant to be used with Firebug, so it's Firefox-only for the moment — but when you're done, your code will be ready for use alongside the latest version of Prototype <em>in all browsers</em>.

<img src="/assets/2008/2/12/console1_1.png" alt="deprecation.js screenshot" />

Using the script is easy. To migrate a page to the current version of Prototype:

1. Find the <code>script</code> tag that references <code>prototype.js</code>. Change the path to point to the latest version of Prototype (or else overwrite the existing <code>prototype.js</code> with the new version).
2. <em>On the very next line</em>, add a <code>script</code> tag that references <code>prototype\_update\_helper.js</code>. 
3. Develop your app as normal.

When your code calls a method that's been deprecated, replaced, or modified, the script will log a warning or error to your Firebug console. Clicking its hyperlink will take you to the update helper script itself, which isn't all that helpful; but the message <em>itself</em> will contain a stack trace that points to the source of the error.

Naturally, the console errors are the most important to address, since they represent things that will <em>no longer work</em> in the latest version of Prototype. The warnings represent deprecations — things that still work in the latest version, but are not guarateed to work in <em>future</em> versions of Prototype. If you'd like to see only removal notices, you can set a property in your code to turn off deprecations:

    prototypeUpdateHelper.logLevel = UpdateHelper.Warn;

As you address these errors and warnings, they'll go away. When there are no more errors, your code is compatible with the current version of Prototype. When there are no more warnings, your code is nimble and future-proof.

<a href="/assets/2008/4/22/prototype_update_helper.js">Download the deprecation script</a>.
