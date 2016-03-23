---
layout: post
title:  "Parse is dying - all hail the benevolent dictator"
description: why Portability is a major design consideration
date:   2016-01-28 00:00 -0500
---

Facebook are [closing down Parse][parse_death] and it seems that the developer community is thrilled. After all, they're letting it die slowly and open-sourcing a parse compatible server. And what if they hadn’t been so nice? Have we gotten to a point where our applications depend on the generosity of 3rd party service providers? Yes. We use gmaps for mapping, Facebook for authentication, Amazon S3 for storage and much more. Some choices are almost unavoidable and some are worth the risk and/or saved resources. However, depending on 3rd party services has become the norm rather than a conscious decision.

Sometimes services get shut down, sometimes pricing changes and sometimes it just doesn’t work as well as it used to / you expected. Dealing with your own bugs and errors is no fun. Dealing with someone else's is horrific. What can we do? design for portability.

Portability is a major design consideration that is often overlooked. When faced with a technology decision we should give credit to options that allow portability. For example, a hosted mongodb solution (compose, mongolab) allows you to seamlessly move your data between services or to a private mongo installation. Whereas proprietary data stores like the Google App Engine datastore or Amazon’s DynamoDB have you locked in for life or until you manually (and painfully) migrate away.

Sometimes it's worth it. Some services, such as Amazon's S3, are so ridiculously good and the portable alternatives are so bad that other decisions just don't make sense. But it's still important that we understand the price we're paying.

On the flip side, and with all due credit, by creating the new open source [parse-server][parse_server], Facebook have just turned Parse into a pretty portable piece of technology. I believe we will soon see multiple Parse-as-a-Service providers that will standardize the young Backend-as-a-Service  industry and make it a better option for those of us who still insist of thinking about portability. 

[parse_death]: http://blog.parse.com/announcements/moving-on/
[parse_server]: https://github.com/ParsePlatform/parse-server