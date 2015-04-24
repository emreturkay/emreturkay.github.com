---
layout: post
title: "Web Components"
description: ""
category: 
tags: 
- web components
---

Recently, I have come across with "Web Components" and the phrase caught my attention. Even though the subject is a few years old, since I am mostly developing embedded code and had little experience with the web frontend development, it was new for me.

<img src="/images/webcomponents-logo.svg"
     alt="Web Components" align="right" width="123" height="79"
     style="width: 123; height: 79">

"Web Components" is the common name of a set of W3C specifications, focusing on increasing the level of reusability and encapsulation for the web development. Look at the following HTML code, which benefits from the Web Components.

{% highlight html %}
<img-slider>
    <img src="images/sunset.jpg" alt="a dramatic sunset">
    <img src="images/arch.jpg" alt="a rock arch">
    <img src="images/grooves.jpg" alt="some neat grooves">
    <img src="images/rock.jpg" alt="an interesting rock">
</img-slider>
{% endhighlight %}

I have stumbled upon the above code in [a blog post](http://css-tricks.com/modular-future-web-components/). It is short, descriptive, and absolutely impressive. The `<img-slider>` tag (called a custom element) is encapsulating data (HTML DOM) and behavior (JavaScript) under the hood. Even better, thanks to the *HTML Imports*, the code describing the `<img-slider>` appearance and behavior is loaded from a separate HTML file by using the `<link>` tag with the `rel='import'` attribute:

{% highlight html %}
<link rel="import" href="/path/img-slider.html">
{% endhighlight %}

The beauty of HTML imports is that allows you to separate all your HTML, JS, and CSS code related to implement specific functions into their own HTML files. Similar to "import", "using" or "include" directives you can find in pretty much every programming language.

Another Web Components specification called *Shadow DOM* will also help you to encapsulate your CSS styles and protect it from the rest of the document (and also protects the style of the rest of the document from your inner style definitions). Shadow DOM also provides a scope for element IDs and classes, as any sane software developer would expect. Combining all of these features it surely presents a revolutionary methodology for the frontend web development.

Personally, I like JavaScript; even though it is a very simple language, it handles modern software development techniques in very smart ways. The OOP infrastructure is straightforward and also open to functional programming concepts. Even though it lacks a detailed encapsulation one can get used to it. It is fun to write pure JavaScript applications, as in node.js. The DOM, on the other hand, is very limiting and honestly I think it is outdated. Web components seem like a great improvement addressing this problem with DOM.

## Current Status

Although it is really exciting stuff, there are still some disadvantages. Web Components is a new technology, the browser support by now is limited with Chrome (and also Opera since both are using the Blink engine). You can see the latest status of the browser support for the Web Components on the page dedicated just for this purpose; ["Are We Componentized Yet?"](http://jonrimmer.github.io/are-we-componentized-yet/) On the rescue comes the [Web Components Polyfills](http://webcomponents.org/polyfills/), they provide most of the functionality defined in the Web Components specification as a JavaScript library. I am not sure if you want to put the Web Components in the production yet since even with the polyfills you cannot get all the functionality defined in the specification and also you will get a performance penalty. However using the polyfills is a great way to start and play around the concept.

Currently, there are 3 popular projects focused on building libraries for the Web Components; [Polymer](https://www.polymer-project.org) from Google, [X-Tag](https://developer.mozilla.org/en-US/Apps/Tools_and_frameworks/x-tags) from Mozilla, and [Bosonic](http://bosonic.github.io). You can find more information about them in their respective web sites.

<img src="/images/polymer-logo.svg"
     alt="Web Components" align="left" width="123"
     style="width: 123; padding-right: 10px">

The Polymer project seems like the most interesting one, it brings lots of web components, which you can use by importing their respective HTML files. Since it is a Google product, at the heart of the visual appearance lays the [material design](http://www.google.com/design/spec/material-design/introduction.html). Maybe the most interesting part of the Polymer is; it provides a declarative component building. The Polymer style definition of `<img-slider>` element would be something like below;

{% highlight html %}
<!-- Define element -->
<polymer-element name="img-slider" attributes="orientation">
    <template>
        <style> /*... Style Information */ </style>
        <div id="container"> <!--... DOM Template --> </div>
    </template>
    <script>
        Polymer({
            /*... JavaScript Code */
        });
    </script>
</polymer-element>
{% endhighlight %}

You can find more information about the Web Components in the WebComponents.org site and in the Polymer web site. I am planning to work more on Polymer and as I do I will share my experience here on my blog.
