---
layout: post
title:  "Clojurescript/Reagent : importing React components from NPM"
description: How to import a React components from npm, via webpack, to a Clojurescript/Reagent app.
draft: false
date:   2016-06-24 23:59:59 -0500
--- 

This post describes, step by step, how to setup a workflow for importing [React][React] components from [NPM][npm], using [Webpack][webpack], and incorporate them in your [Reagent][reagent] views.

### Motivation
Any Javascript developer using a modern build tool can easily test and incorporate React components from 3rd party developers in their app. It's usually just a matter of declaring dependencies, building and importing.

Moving to Clojurescript and Reagent has been amazing in many ways but I just couldn't wrap my head around a comparable import flow.

The workflow I will describe employs NPM as a dependency manager and Webpack as a build tool. The build artifact is added as an external Javascript dependency to an otherwise (pretty) standard Leinnnigen build process. 

This solution does not use Cljsjs in anyway.  

### Why not Cljsjs? 
[Cljsjs][cljsjs] is a great community effort to package common Javascript libraries in a Clojurescript friendly way. However, it has some shortcomings. 

1. Very few packages compared to NPM or Bower. Whenever I look for something it's usually not there.
2. Packages are mostly [out of date][pkgs.csv] compared to their NPM counterparts. This make sense because with NPM packages the author/maintainer of the library is the same person as the maintainer of the package. That's not the case with Cljsjs. 
3. Packaging a library is not trivial. The community's usual reply to the import issue is "just package it yourself". Sending someone who wants to fiddle around with a 3rd party react component, to package the dependency themselves is, in my opinion, cumbersome. This also leads back to problem #2.

### Zefstyle

In this example we'll start with a simple frontend-only Reagant app and setup a workflow that will allow us to add a [react-player][react-player] component to it. Let's call this project **Zefstyle**. You can also skip this altogether and just jump to the finished result [here][zefstyle-github]. 

Starting with a template: 

{% highlight bash %}
$ lein new reagent-frontend zefstyle && cd zefstyle
{% endhighlight %}

For sanity - Let's see that it's working

{% highlight bash %}
$ lein figwheel
{% endhighlight %}

While that's running, open public/index.html in your favorite browser. You should see a page that says "Welcome to Reagent"

Since we're running figwheel, we can change that live. In `public/index.html` change the header to "Zef Style" - You should see it updating within seconds. You can then close fihghweel for now.

### NPM Setup
Let's create a minimal `package.json` file. We can either do it interactively with `npm init` or just manually create a `package.json` file: 

{% highlight json %}
{
  "name": "zefstyle",
  "version": "0.0.1",
  "author": "tomer.weller@gmail.com",
  "scripts": {
    "watch": "webpack -d --watch",
    "build": "webpack -p"
  },
  "dependencies": {
    "react-player": "0.7.3",
    "react": "15.1.0",
    "react-dom": "15.1.0",
    "webpack": "1.13.1"
  }
}
{% endhighlight %}

#### package.json remarks: 
- We import `react` and `react-dom` although they are not a hard dependency of `react-player` and more than that - we already have a React instance in our code, the one that Reagant depends on. Why? There is an order of execution issue when relying on Reagant's React dependency. This also means that we will later need to exclude Reagant's React dependency so that we don't have two React instances. 
- I've included some shortcuts for webpack scripts. They are not necessary but will come in handy.

Install dependencies: 

{% highlight bash %}
$ npm install
{% endhighlight %}

All of our dependency tree should now be in the `node_modules` directory. 

An alternative approach to this step would be to use the [lein-npm][lein-npm] plugin. I'm pretty comfortable with npm so I decided to do it myself. 

### Webpack setup
Our webpack setup will be pretty minimal - no fancy loaders. It consists of a definition file `webpack.config.js` and an entry script that bootstraps our imported libraries to the `window` object. 

`webpack.config.js` :

{% highlight javascript %}
const webpack = require('webpack');
const path = require('path');

const BUILD_DIR = path.resolve(__dirname, 'public', 'js');
const APP_DIR = path.resolve(__dirname, 'src', 'js');

const config = {
  entry: `${APP_DIR}/main.js`,
  output: {
    path: BUILD_DIR,
    filename: 'bundle.js'
  },
};

module.exports = config;
{% endhighlight %}

We basically defined our entry point script as `src/js/main.js` and our output artifact as `public/js/bundle.js`.

Our entry `src/js/main.js` should look something like: 

{% highlight javascript %}
window.deps = {
    'react' : require('react'),
    'react-dom' : require('react-dom'),
    'react-player' : require('react-player'),
};

window.React = window.deps['react'];
window.ReactDOM = window.deps['react-dom'];
{% endhighlight %}

I usually prefer not to bind too many objects to the global window context, that's why I push whatever I can into `window.deps`. With that said, `React` and `ReactDOM` must be on the global window context because that's where components expect them to be. 

Now we can either run

{% highlight bash %}
$ npm run build 
{% endhighlight %}

For a onetime build, or: 
{% highlight bash %}
$ npm run watch
{% endhighlight %}

For a continuous build that watches for changes in our code. Given that the Javascript code should remain pretty static the watch might be redundant.
 
To finalize our webpack setup we need to add the `bundle.js` in a script tag to the main html page.

`public/index.html` should look something like this

{% highlight html %}
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta content="width=device-width, initial-scale=1" name="viewport">
    <link href="css/site.css" rel="stylesheet" type="text/css">
  </head>
  <body>
    <div id="app">
      <h3>ClojureScript has not been compiled!</h3>
      <p>please run <b>lein figwheel</b> in order to start the compiler</p>
    </div>
    <script src="js/bundle.js" type="text/javascript"></script>
    <script src="js/app.js" type="text/javascript"></script>
  </body>
</html>

{% endhighlight %}



### Excluding cljsjs.react
Like I mentioned before, we're counting on webpack to bring React into the picture. That means that we need to get rid of the `cljsjs.react` & `cljsjs.react.dom` packages that reagant depends on. 

In `project.clj` let's change our dependencies to be:

{% highlight clojure %}
:dependencies [[org.clojure/clojure "1.8.0" :scope "provided"]
               [org.clojure/clojurescript "1.9.36" :scope "provided"]
               [reagent "0.6.0-rc" :exclusions [cljsjs/react cljsjs/react-dom]]]
{% endhighlight %}

Now, to fool Reagant into thinking that these packages already exist - we're going to create empty namespaces.

`src/cljsjs/react.cljs`:
{% highlight clojure %}
(ns cljsjs.react)
{% endhighlight %}

`src/cljsjs/react/dom.cljs`:
{% highlight clojure %}
(ns cljsjs.react.dom)
{% endhighlight %}

At this point it's a good idea to cleanup and re-run figwheel.

{% highlight bash %}
$ lein clean && lein figwheel
{% endhighlight %}

### Grand finale

Now that we have everything in place we can add the react-player to our main Reagant view.

in `src/zefstyle/core.cljs` let's change our `home-page` function to be:

{% highlight clojure %}
(defn home-page []
  (let [react-player (aget js/window "deps" "react-player")]
    [:div
     [:h2 "Zef Style"]
     [:> react-player {:url "https://youtu.be/uMK0prafzw0"}]]))
{% endhighlight %}

Notice the special `:>` syntax for using pure React components.

opening `public/index.html` should yield something like this: 
![yolandi](/assets/zefstyle.png)
	
**Voila!** We brought the pure JS react-player component into our Reagent view. 

To bring in new components we just need to declare them in `package.json`, `require()` them in main.js and rebuild.

### Caveats:

1. JS dependencies are not processed by the Closure compiler so they do not get optimized at all. 
2. Setting up this process is not trivial. However, it only needs to be done once and from then on it's a smooth sail. Also, automating the process or part of it sounds like a feasible task as part of a lein plugin. I'll look into that when some time frees up. 

Let me know if this was helpful, requires some fixups or if it's complete rubbish.
 
### Thanks to: 
- [Thariq Shihipar][thariq] for suggesting this approach although condemning the use of anything other than Javascript and React.
- [/r/clojurescript][clojurescript-reddit], [Clojurians on Slack][clojurians] and especially @sbmitchell for helping me troubleshoot.  

[zefstyle-github]:https://github.com/tomerweller/zefstyle
[react-player]:https://www.npmjs.com/package/react-player
[npm]:https://www.npmjs.com/
[reagent]:https://reagent-project.github.io/
[webpack]:https://webpack.github.io/
[cljsjs]:http://cljsjs.github.io/
[react]:https://facebook.github.io/react/
[pkgs.csv]:https://docs.google.com/spreadsheets/d/1t61hPgEfb1ukzezb6dn38z8kp07TsEFgkaVwF01-acE/pubhtml
[lein-npm]:https://github.com/RyanMcG/lein-npm
[thariq]:https://twitter.com/trq__
[clojurians]:https://clojurians.slack.com
[clojurescript-reddit]:https://www.reddit.com/r/clojurescript

