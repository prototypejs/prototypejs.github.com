---
layout: blog_archive
author: Andrew Dupont
author_url: http://andrewdupont.net/
sections: blog
title: "Documentation: not just new, but also improved"
digest: "When we officially released 1.6.1 last week, we also published new documentation, the first official docs generated with [PDoc](http://pdoc.org)."
---

When we officially released 1.6.1 last week, we also published new documentation, the first official docs generated with [PDoc](http://pdoc.org).

Tobie, ear to the ground, brought to my attention what many of you were saying (on the blog and [on Twitter](https://twitter.com/#!/search/@prototypejs?q=%40prototypejs)): the new docs were harder to navigate and, therefore, harder to browse. Though I had eventual plans to re-do the navigation, the instant feedback showed it was a more critical issue than I’d guessed. So I spent the last week making some changes to the template we use to generate the docs.

You can see the results at [api.prototypejs.org](http://api.prototypejs.org). The biggest change is obvious: a fixed, always-visible sidebar that makes it easier to move from section to section. Typing in the search box replaces the hierarchical navigation with a list of matching results. Clearing the search box (use the ESC key as a shortcut) switches back to the ordinary navigation. The sidebar will preserve state from page to page — it’ll remember your search term and the scrollbar position.

The docs aren’t perfect yet, but they’re good enough to use. I’ve tested them on Firefox 3.5, Safari 4.0, and IE 7–8. If there are glitches in these browsers or others, please [open issues on the GitHub project](https://github.com/savetheclocktower/prototype-pdoc-template/issues). (If you, as a JavaScript developer, are still using IE 6, I’d like to take you out for a beer and ask you why.)

We intend for this to be default template included with PDoc, albeit without the Prototype branding. And now that we’ve accomplished the most pressing goal — getting PDoc to generate comprehensive and canonical docs for Prototype — we can focus on the big ideas we’ve got for the next version of our inline documentation tool.