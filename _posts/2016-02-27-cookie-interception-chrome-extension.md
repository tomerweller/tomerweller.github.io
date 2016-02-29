---
layout: post
title:  "Intercepting Cookies in a Chrome extension"
draft: true
date:   2016-02-27 23:59:59 -0500
---

Last week I built a Chrome extension that was supposed to solve privacy on the Web. It didn't - but that's a different post. 

I did, however, found my self in need of intercepting all cookie read/writes in a Chrome extension. This proved to be interesting and non-trivial so here's a brain dump of the process - hopefully this helps someones somewhere. (and by someone, I'm looking at you, future self!) 

### Intercepting cookies?
Yup. Some potential use cases: 

- Managing multiple Cookie stores
- Using alternative Cookie stores
- Namespacing Cookies
- Exposing Cookies on a conditional basis like time, url, etc. 
- Cookie Art

Generally speaking, an extension will intercept cookies either to save your privacy or to completely violate it. *With great power comes great responsibility*.

### Isn't there a Cookie API for Chrome extensions?
True. And it's a [fine API][chrome.cookies]. However, it does not allow real-time interception. Meaning, you can read and write cookies and even get notified when cookies change, but not manipulate them *WHILE* they are being read / written. 
 
### OOGI
For the sake of this post I will build **OOGI** (*shorthand for Oogifletset - the Hebrew translation of "The Cookie Monster"*) a simple extension that statically namespaces all cookies with the prefix **`oogi$`**. When cookies are set they will be prefixed before being written into the cookie store, and when being queried the prefix will be removed - making it completely transparent to remote servers. 

The following assumes you have basic knowledge in building a chrome extension and understand what a cookie is. The full source code is available [here][oogi-source].

There are two ways to get and set cookies. I'll tackle each one of them separately:  

- **HTTP Headers**
- **document.cookie**

### HTTP Headers

The HTTP Request "Cookie" header will contain all cookies currently set and the HTTP Response's "Set-Cookie" header writes/modifies cookies. 

A Request's "Cookie" header is a concatenation of all cookie names and values for the current domain and path:

{% highlight html %}
Cookie:my_int_cookie=16091; my_double_cookie=0.2545873837079853
{% endhighlight %}

A Response can contain multiple "Set-Cookie" headers, one per cookie. These also contain a Path and optionally a timeout: 

{% highlight html %}
Set-Cookie:my_int_cookie=5540; Path=/
Set-Cookie:my_double_cookie=0.5358087662607431; Path=/
{% endhighlight %}

Luckily, Chrome extensions have a [Webrequest API][chrome.webRequest] for manipulating HTTP requests and responses in a blocking manner. 

Declaring stuff in `manifest.json`: 

{% highlight json %}
{
    ...
    
    "permissions": [
        "webRequest",
        "webRequestBlocking",
        "*://*/*"
    ],
    "background": {
        "scripts": [
            "background.js"
        ],
        "persistent": true
    },
    
    ...
}
{% endhighlight %}

The [WebRequest API][chrome.webRequest] allows us to intercept different stages in the lifecycle of an HTTP Request/Response loop. 

Here, in `background.js`, we plug into `onBeforeSendHeaders` to modify sent cookies and to `onHeadersReceived` to modify the setting of cookies:

{% highlight javascript %}
chrome.webRequest.onBeforeSendHeaders.addListener(
    function (details) {
        details.requestHeaders.forEach(function(requestHeader){
            if (requestHeader.name.toLowerCase() === "cookie") {
                requestHeader.value = processCookieStr(requestHeader.value);
            }
        });
        return {
            requestHeaders: details.requestHeaders
        };
    }, {
        urls: [ "*://*/*" ]
    }, ['blocking', 'requestHeaders']
);

chrome.webRequest.onHeadersReceived.addListener(
    function (details) {
        details.responseHeaders.forEach(function(responseHeader){
            if (responseHeader.name.toLowerCase() === "set-cookie") {
                responseHeader.value = processSetCookieStr(responseHeader.value);
            }
        });
        return {
            responseHeaders: details.responseHeaders
        };
    }, {
        urls: ["*://*/*"]
    }, ['blocking','responseHeaders']
);
{% endhighlight %}

*The actual cookie string manipulation happens in a separate file. See [full code][oogi-source] for specifics.*

### document.cookie

This is the fun part. 

Cookies can be directly manipulated with Javascript via the [document.cookie][document.cookie] property which is a javascript [accessor property][defineProperty]. If you've never encountered accessor properties (like me until a week ago), you're in for some surprises. 

Let's examine `document.cookie` in the Chrome dev tools console:

{% highlight javascript %}
> document.cookie
"my_int_cookie=25760; my_double_cookie=0.563346192939207"
{% endhighlight %}

Looks like a String property. right? Let's try to write a different value.

{% highlight javascript %}
> document.cookie="A=1"
"A=1"
> document.cookie
"my_int_cookie=25760; my_double_cookie=0.563346192939207; A=1"
{% endhighlight %}

So writing to this property actually appends the written value? 

{% highlight javascript %}
> document.cookie="A=2"
"A=2"
> document.cookie
"my_int_cookie=25760; my_double_cookie=0.563346192939207; A=2"
{% endhighlight %}

Nope. It seems that the String is only a representation of an actual collection behind the scenes.

So what's going on here? Accessor properties can be set with custom getter and setter functions. That means that by invoking the property we're actually calling a getter function and by assigning value, a setter function. These are defined using the [Object.defineProperty()][defineProperty] method.

Let's try to use Object.defineProperty() function to override document.cookie: 

{% highlight javascript %}
> Object.defineProperty(document, 'cookie', {
      get: function() {
          return "My amazing getter";
      },
    
      set: function(value) {
          console.log("My amazing setter");
      }
  });
> document.cookie
"My amazing getter"
> document.cookie = 1
My amazing setter
1

{% endhighlight %}

Great success! This might suffice if our extension completely ignores Chrome's internal cookie store. However, if access to the cookie store is still required, which is case for Oogi, we need to save a reference to the original getter and setter functions of document.cookie.

Theoretically, the [Object.getOwnPropertyDescriptor()][getOwnPropertyDescriptor] function should help with that. 

{% highlight javascript %}
> Object.getOwnPropertyDescriptor(document, "cookie")
undefined
{% endhighlight %}

It seems, however, that Chrome doesn't want you to access that specific descriptor. 

However, We can still use the [\_\_lookupGetter\_\_()][lookupGetter] and [\_\_lookupSetter\_\_()][lookupGetter] functions which are **deprecated-do-not-use-or-the-universe-will-implode**.

{% highlight javascript %}
> var cookieGetter = document.__lookupGetter__("cookie").bind(document)
> var cookieSetter = document.__lookupSetter__("cookie").bind(document)
> cookieGetter()
"my_int_cookie=14356; my_double_cookie=0.22253736178390682"
> cookieSetter("A=1")
> cookieGetter()
"my_int_cookie=14356; my_double_cookie=0.22253736178390682; A=1"
{% endhighlight %}

Bingo! 

However, all of this this experimentation was done in the Chrome dev tools console. How do we port this to our extension?

Running custom javascript code in every webpage is done using [content scripts][chrome.content]. However, the document object in a content script is actually a replica. Modifying it is not visible to the rest of the Javascript code in the page. Luckily, this is a [well discussed][so.injectjs] issue and the simplest solutions is to create a custom script element and inject it into every page.

Declaring our content script in  `manifest.json`:

{% highlight json %}
{
    ...
    "permissions": [
        "contentSettings",
        "*://*/*"
    ],
    "content_scripts": [
        {
            "matches": ["*://*/*"],
            "js": ["content.js"],
            "run_at": "document_start"
        }
    ],
    "web_accessible_resources": ["inject.js"],
    ...
}
{% endhighlight %}

Creating a `content.js` script that injects an external js file: 

{% highlight javascript %}
var headElement = (document.head||document.documentElement);
var injectJs = function(fileName) {
    var s = document.createElement('script');
    s.src = chrome.extension.getURL(fileName);
    headElement.insertBefore(s, headElement.firstElementChild);
};
injectJs("inject.js");
{% endhighlight %}

And finally, overriding `document.cookie` with our injected javascript: 

{% highlight javascript %}
var cookieGetter = document.__lookupGetter__("cookie").bind(document);
var cookieSetter = document.__lookupSetter__("cookie").bind(document);

Object.defineProperty(document, 'cookie', {
    get: function() {
        var storedCookieStr = cookieGetter();
        return processCookieStr(storedCookieStr);
    },

    set: function(cookieString) {
        var newValue = processSetCookieStr(cookieString);
        return cookieSetter(newValue);
    }
});
{% endhighlight %}

That's it! 

I left out some of the glue code to keep this post relatively short. You can checkout the complete example [here][oogi-source]. Feel free to submit issues and pull requests if I missed anything.

##### Huge thanks to Thariq Shihipar for helping out with the JS hacking!

[oogi-source]: http://www.github.com/tomerweller/oogi
[chrome.webRequest]: https://developer.chrome.com/extensions/webRequest
[chrome.cookies]: https://developer.chrome.com/extensions/cookies
[chrome.content]: https://developer.chrome.com/extensions/content_scripts
[document.cookie]: https://developer.mozilla.org/en-US/docs/Web/API/Document/cookie
[defineProperty]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty
[accessor_property]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty
[getOwnPropertyDescriptor]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyDescriptor
[lookupGetter]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/__lookupGetter__
[lookupSetter]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/__lookupSetter__
[so.injectjs]: http://stackoverflow.com/questions/9515704/building-a-chrome-extension-inject-code-in-a-page-using-a-content-script