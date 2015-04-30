---
layout: post
title: "Chat Application with Polymer - I"
description: ""
category: 
tags: 
- web components
- polymer
- socket.io
- real time web
---

As I have mentioned in my [last post]({% post_url 2015-04-15-web-components %}), I have been experimenting with Web Components and in particular Google’s Polymer library. There is another concept I want to learn and play around; ["Real Time Web"](http://en.wikipedia.org/wiki/Real-time_web). Real Time Web is the term used to describe continuous bidirectional web client/server communication as in web based chat applications and continuously streaming feeds in a social media application without requiring a "Refresh" button. There are various techniques used to provide continuous communication such as ["long polling"](http://en.wikipedia.org/wiki/Push_technology#Long_polling) and ["web sockets"](http://en.wikipedia.org/wiki/WebSocket). A node.js JavaScript library called [Socket.IO](http://socket.io) provides a high level client/server API for real time functionality while hiding the low level protocols. Instead of focusing any one of the low level protocols, it seems like a good idea to use the Socket.IO if you need real-time behavior in your application.

## Socket.IO Example

In the Socket.IO home page, there is [a quick and nice tutorial](http://socket.io/get-started/chat/) demonstrating the basics of server and client side application development with Socket.IO. The tutorial describes how to write a very simple web based chat application. If you don’t know anything about Socket.IO, as I did, you can follow the tutorial and get a basic understanding in about 15 minutes. At the end of the tutorial there is a homework section listing a few suggestions that could be added to the tutorial application. By trying to do the homework you can test your freshly gathered knowledge.

Since learning by example is the best way to master a subject; I decided to play around the “Chat Example Application” tutorial and add Polymer constructs to it. I have also decided to do the homework given at the end of the tutorial using Polymer, Web Components, and similar techniques.

I have followed the tutorial to build my own chat example but before starting the Polymer Chat Application project I decided to fork the original node.js chat application [rauchg/chat-example](https://github.com/rauchg/chat-example). This way we can keep the whole history and provide a semantically complete reference to any future explorers. _Forking_ in GitHub terminology is simply making a `git clone` on the server side. There is a "Fork" button on the upper right side of the github page of chat-example, when I clicked it, the github forked the project and created a copy of it prepending my user name to its original name. Since it is mostly a demonstration for Polymer, I have decided to change its name to "polymer-chat-example", hence the name [emreturkay/polymer-chat-example](https://github.com/emreturkay/polymer-chat-example) is produced. Then I have cloned the sources to my laptop to fiddle with them.

You will need node.js on your computer to follow the rest of this document. You can download the binaries for your Operating System from the [node.js download page](https://nodejs.org/download). When you are ready, you can simply clone the [rauchg/chat-example](https://github.com/rauchg/chat-example) repo on your computer and give it a try. Once you download the sources you need to first install the dependencies by executing `npm install`. It creates a "node_modules" directory and downloads the dependency files in it. In our example the dependencies are "express" and "socket.io" libraries. You can then execute your server side application by typing `node index.js` in the command line and connect to your chat application by entering "localhost:3000" into the browser address bar. 

    $ npm install
    $ node index.js

The very first change I made to the source code was to remove the "jquery" dependency. Since we will experiment with Polymer, we don’t need another level of complexity. The Socket.IO example is pretty basic so simply replacing jquery `$()` function with `document.getElementById()` calls was enough for the migration.

## Setting Up Polymer

Polymer uses [bower](http://bower.io) as the package management system. Similar to npm’s "package.json" file, bower has a "bower.json" file to track the dependencies. We can create it by executing `bower init` in the source directory. It asks a bunch of questions, you can simply choose the default answers. Next we need to install the polymer dependencies. We can accomplish it by using `bower install` command:

    $ bower init
    $ bower install --save Polymer/polymer#~0.5
    $ bower install --save Polymer/core-elements#~0.5
    $ bower install --save Polymer/paper-elements#~0.5

As you can guess "Polymer/polymer" is the base dependency, it also installs the web components polyfills "webcomponentsjs".  Polymer has two different sets of elements;

* _core elements_: a set of useful  utility elements and
* _paper elements_: a set of visual elements based on Google’s material design.

As you can see above we are installing both of these libraries. The `—-save` option lets the bower to modify the "bower.json" file and add the specified package as a dependency. Our generated bower.json looks like this:

{% highlight javascript %}
{
  "name": "polymer-chat-example",
  "main": "index.js",
  "version": "0.0.1",
  "homepage": "https://github.com/emreturkay/polymer-chat-example",
  "authors": [
    "Emre Turkay <emreturkay@gmail.com>"
  ],
  "keywords": [
    "polymer",
    "web components",
    "socketio"
  ],
  "license": "MIT",
  "ignore": [
    "**/.*",
    "node_modules",
    "bower_components",
    "test",
    "tests"
  ],
  "dependencies": {
    "polymer": "Polymer/polymer#~0.5",
    "core-elements": "Polymer/core-elements#~0.5",
    "paper-elements": "Polymer/paper-elements#~0.5"
  }
}
{% endhighlight %}

## Using Custom Elements

Now we are ready for some Web Components action. Since we are using Polymer, we can easily give it a better look with the paper elements. I will wrap the `<input>` within a [`<paper-input-decorator>`](https://www.polymer-project.org/0.5/docs/elements/paper-input-decorator.html) element, replace the default button with a [`<paper-icon-button>`](https://www.polymer-project.org/0.5/docs/elements/paper-icon-button.html) with a "send" icon, and give some shadow to the messages with the [`<paper-shadow>`](https://www.polymer-project.org/0.5/docs/elements/paper-shadow.html) element.

As we have seen in my previous post, first we need to import the HTML file, which defines the custom component we want to use. However, currently, not all the browsers support Web Components specification including the "HTML Imports", however we can use Web Components Polyfills to partially present the functionality. The polyfills are defined within a JavaScript file; let’s start by including it:

{% highlight html %}
<script src="/bower_components/webcomponentsjs/webcomponents.js">
</script>
{% endhighlight %}

Now we can import the HTML files defining the `<paper-input-decorator>`, `<paper-icon-button>`, and `<paper-shadow>` custom elements:

{% highlight html %}
<link rel="import" href="/bower_components/polymer/polymer.html">
<link rel="import" href="/bower_components/core-icon/core-icon.html">
<link rel="import" href="/bower_components/paper-input/paper-input-decorator.html">
<link rel="import" href="/bower_components/paper-icon-button/paper-icon-button.html">
<link rel="import" href="/bower_components/paper-shadow/paper-shadow.html">
{% endhighlight %}

The last missing piece is that our node.js application currently serves only one file, "index.html", however the browser will request the JavaScript file "webcomponents.js" given in the `<script>` element and all the HTML files given in the `<link>` elements from our server. Since all these files are located in the "bower_components" directory we will configure the express so that it serves them to the browser whenever requested:

{% highlight javascript %}
var express = require('express');
var app = express();
var http = require('http').Server(app);
var io = require('socket.io')(http);

app.get('/', function(req, res) {
    res.sendFile(__dirname + '/index.html');
});

app.use("/bower_components", express.static(__dirname + "/bower_components"));
{% endhighlight %}

Finally we can use the custom elements provided by Polymer exactly as if we were using the built-in HTML elements:

{% highlight html %}
<body onLoad="onLoad()">
    <div id="messages"></div>
    <form id="textForm" action="">
        <paper-input-decorator id="decorator"
                               label="Enter message to send">
            <input id="text" autocomplete="off" autofocus/>
        </paper-input-decorator>
        <paper-icon-button id="button" icon="send"></paper-icon-button>
    </form>
</body>
{% endhighlight %}

Here we have wrapped the `<input>` element within a `<paper-input-decorator>`, as its name suggests decorating it to have a better look and functionality. Replaced the `<button>` with `<paper-icon-button>`. We let the button to have the "send" icon from Polymer core-icons library. We can also use the custom element as a parameter to the `document.createElement()` call. We have wrapped the messages in a `<paper-shadow>` element as you can see in the `<body> onLoad` event handler below:

{% highlight javascript %}
function onLoad() {
    var messagesList = document.getElementById("messages");
    // ...
    socket.on('chat message', function(msg) {
        var shadow = document.createElement('paper-shadow');
        shadow.innerHTML = msg;
        messagesList.appendChild(shadow);
    });
}
{% endhighlight %}

Yes, I could make the shadow with CSS, but in that case we wouldn't have a demonstration for the `document.createElement()` usage and with my aesthetic ability it surely wouldn't be as nice looking as it is now. We have used some attributes with the custom elements, such as `label` for the `<paper-input-decorator>` and `icon` for the `<paper-icon-button>`. You can find their description and more attributes to specify in the [Polymer elements documentation](https://www.polymer-project.org/0.5/docs/elements/).

Similar to using the custom elements, styling them is also the same as the built-in elements. In the CSS section below you can see the styling of `paper-input-decorator`, `paper-icon-button`, and `paper-shadow` elements:

{% highlight html %}
<style>
/* ... */
form paper-input-decorator {
    position: fixed;
    bottom: 0;
    padding: 10px;
    width: 90%;
    margin-right: .5%;
}
form paper-icon-button {
    position: fixed;
    bottom: 0;
    right: 0;
    margin: 0 20px 10px 0;
}
paper-shadow {
    margin: 5px;
    padding: 10px;
    float: left;
    align-self: flex-start;
}
</style>
{% endhighlight %}

Now we have a fancy looking chat application, using exactly the same logic with the node.js tutorial example. The best part is we need minimal information to use the custom elements. We can use our current HTML, CSS, and JavaScript knowledge mostly as they are.

You can fetch the result from the [GitHub repo](https://github.com/emreturkay/polymer-chat-example/releases/tag/0.0.1). If you don't have time to try it yourself, this is how it looks:

<img src="/images/polymer-chat-capture-01.png"
     alt="Polymer Chat Screen Capture" align="right">

