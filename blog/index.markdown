---
layout: default
title: "Blog"
---

{% for post in site.posts %}
  * {{ post.date | date: '%B %e, %Y'}} – [{{ post.title }}]({{ post.url }})
{% endfor %}