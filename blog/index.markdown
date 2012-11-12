---
layout: default
title: "Blog Archive"
body_id: "blog"
title_header: true
site_section: "blog"
---

<ul class="posts-list">
{% for post in site.posts %}
  <li>
    <h4 class="post-title"><a href="{{ post.url }}">{{ post.title }}</a></h4>
    <small class="post-meta">by {{ post.author }} on {{ post.date | date: '%B %e, %Y') }}</small>
    {% if post.digest %}
    <div class="post-digest">
      {{ post.digest | markdownify }}
    </div>
    {% endif %}
  </li>
{% endfor %}
</ul>

