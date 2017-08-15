---
layout: default
title:  "Tomer Weller / A Blob"
---

Hi, I'm **Tomer Weller** and I make things. Usually, with computers and code. I recently graduated from the MIT Media Lab where, amongst other things, I worked on fostering collaboration between makers and non-for-profit organizations. I'm interested in how people do the things that they do. And LISP.


<H3>Posts</H3>

{% for post in site.posts %}{% if post.draft != true %}
- [{{post.title}}]({{ post.url | prepend: site.baseurl }}) ({{ post.date | date: "%b %-d, %Y" }}) {% endif %}{% endfor %} 



<H3>Links</H3>
- [Master's Thesis : "This is how : story centric distributed brainstorming and collaboration"](https://dspace.mit.edu/handle/1721.1/107549) (Aug 2016)
- [ESP32 - Toolchain setup on OSX](http://www.pubpub.org/pub/esp32-osx-setup) (Jan 2016)
- ["How to Make Something that Makes (almost) Anything](http://fab.cba.mit.edu/classes/865.15/people/tomer.weller/index.html) (Spring 2015)
- ["How to Make (almost) Anything"](http://fab.cba.mit.edu/classes/863.14/people/tomer_weller/index.html) (Fall 2014)
- [My old portfolio website](http://www.tomerweller.com) (2014)
