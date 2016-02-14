---
layout: post
title:  "A Blog named Blob"
date:   2016-02-10 23:59:59 -0500
---

**Blog?** A couple of weeks ago, following the [death of Parse.com][parse_death], I wrote a [rant]({% post_url 2016-01-28-parse-is-dying-all-hail-the-benevolent-dictator %}) about the virtues of portability and how they are too often overlooked. Posting that rant on a system that inherently does not allow portability (facebook, medium, etc.) felt a bit wrong. So, following a New Year's resolution to better document my work, a blog seemed in order. 

**Blob?** Developer blogs usually have a wise ass name. This Blob is no different. Some rationalizations:

- "an indeterminate mass or shape" : I'm not committing to a specific theme. Tech rant will probably rule but an occasional hummus recipe or fabrication tutorial might appear. 
- "**b**inary **l**arge **ob**jects" : Same as above but in geek terminology. Anything can be represented as a binary object.
- Yes. Blob sounds like Blog. 

**[Jekyll][jekyll]** is a bullet proof, dead simple and open source static site generator. In comes markdown - out comes a static website. The output, being simple html, can be hosted virtually anywhere. As a bonus, [github pages][ghpages] will build and host your jekyll website free of charge. 

**Comments** are a bitch. They add dynamic content to an otherwise beautifully static website. The de-facto industry standard is disqus but using that means someone else owns your discussion. It also means user session data is shared and no anonymous comments are allowed. In comes **[isso][isso]** - an opensource, self hosted alternative to disqus. I love the idea of having sqlite as a datastore and, let's face it, the ammout of expected comments (~1 per year) does not justify a multi-connection database solution. I'm running isso on a VM in an undisclosed location in the world... 

**Design** - I'm slowly developing a hatred for beautiful web pages. Everything on the web is just too God damn pretty these days. I decided to get rid of all of Jekyll's styling and start from scratch. I like fixed width fonts and I'm taking cues from [Florent Verschelde's][fvsch] [monospace theme][monospace_theme] on tumblr and [remarkdow][remarkdown] css library.
 [My css][my_css] file is currently 10 lines long but will probably grow when richer content is added. I'm also using a [solarized bright][solarized_css] css for code highlighting and isso's default css with minor modifications. 

The code is available [here][here]. 

[parse_death]: http://blog.parse.com/announcements/moving-on/
[ghpages]: https://pages.github.com/
[jekyll]: http://jekyllrb.com/
[isso]: https://github.com/posativ/isso
[fvsch]: http://fvsch.com/
[monospace_theme]: https://monospace-theme.tumblr.com/
[remarkdown]: http://fvsch.com/code/remarkdown/
[my_css]: https://github.com/tomerweller/tomerweller.github.io/blob/master/css/custom.css
[here]: https://github.com/tomerweller/tomerweller.github.io
[solarized_css]: https://gist.github.com/edwardhotchkiss/2005058