---
layout: default
title:  "Tomer Weller / A Blob"
---

<H3>Posts</H3>

{% for post in site.posts %}{% if post.draft != true %}
- [{{post.title}}]({{ post.url | prepend: site.baseurl }}) ({{ post.date | date: "%b %-d, %Y" }}) {% endif %}{% endfor %} 



<H3>Links</H3>

- [ESP32 - Toolchain setup on OSX](http://www.pubpub.org/pub/esp32-osx-setup) (Jan 2016)
- ["How to Make Something that Makes (almost) Anything](http://fab.cba.mit.edu/classes/865.15/people/tomer.weller/index.html) (Spring 2015)
- ["How to Make (almost) Anything"](http://fab.cba.mit.edu/classes/863.14/people/tomer_weller/index.html) (Fall 2014)
- [My old portfolio website](http://www.tomerweller.com) (2014)