---
layout: default
title:  "Tomer Weller / A Blob"
---

<H3>Drafts</H3>

{% for post in site.posts %}{% if post.draft == true %}
- [{{post.title}}]({{ post.url | prepend: site.baseurl }}) ({{ post.date | date: "%b %-d, %Y" }}) {% endif %}{% endfor %} 