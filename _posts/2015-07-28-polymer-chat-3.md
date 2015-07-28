---
layout: post
title: "Chat Application with Polymer - III"
description: ""
category: 
tags: 
- web components
- polymer
- socket.io
- real time web
- data binding
---

This is the 3rd post of the series of posts on [an example chat application with Polymer](https://github.com/emreturkay/polymer-chat-example). In the previous posts, we have built a chat application with node.js, replaced standard HTML elements with Polymer components and built our own custom components.

In this post we will focus on the Polymer's way of data binding.

## Data Binding

Similar to other JavaScript frameworks, Polymer provides a mechanism called _data binding_ to provide a better Model/View separation. Actually, in the [last post]({% post_url 2015-06-11-polymer-chat-2 %}) we have used data binding when we were defining the event handlers for the `<paper-input>` and `<paper-icon-button>` elements;

{% highlight html %}
{% raw %}
<template>
    <!-- ... -->
    <paper-input
        id="input"
        label="Enter message to send"
        on-keypress="{{keyPressed}}"
        flex>
    </paper-input>
    <paper-icon-button
        id="button"
        icon="send"
        on-tap="{{buttonTapped}}">
    </paper-icon-button>
    <!-- ... -->
</template>
<script>
    Polymer({
        // ...
        keyPressed: function(event, detail, sender) { /* ... */ },
        buttonTapped: function(event, detail, sender) { /* ... */ },
        // ...
    });
</script>
{% endraw %}
{% endhighlight %}

In the code above the `on-keypress` and `on-tap` events are bound to the `keyPressed()` and `buttonTapped()` methods of the `<chat-message-editor>` component, respectively. In Polymer, the data (model) is the element itself, we can make an experiment to see this. First load the chat application with Chrome Browser and invoke "Inspect Element" from the right click menu, make sure the `<paper-input-decorator>` element is selected and type `$0.label="Type here"` in the JavaScript Console. You will see the placeholder text will change from "Enter message to send" to "Type here". Here `$0` represents the selected `<paper-input-decorator>` element.

We can use the double mustache notation to bind both function and data properties of the element, as seen in the code above. For example, we could have bound the `label` property of the `<paper-input>` element to a `placeholder` property of the element as below:

{% highlight html %}
{% raw %}
<paper-input
    id="input"
    label="{{placeholder}}"
    on-keypress="{{keyPressed}}"
    flex>
</paper-input>
{% endraw %}
{% endhighlight %}

and in the index.html, we could change it by setting the property:

{% highlight html %}
{% raw %}
document.getElementById("editor").placeholder = "Type here";
{% endraw %}
{% endhighlight %}

### The `<template>` tag
The Web Components specifications include a new `<template>` element. With the `<template>` tag you can define a DOM sub tree, similar to a `<div>` element. The good part is the contents of the `<template>` will not be rendered on the screen (the image, audio and video resources are not downloaded, the content is not displayed, etc) until you instantiate it.

Here is an example taken from [Eric Bidelman](http://ericbidelman.com)'s article [HTML's New Template Tag](http://www.html5rocks.com/en/tutorials/webcomponents/template/);

{% highlight html %}
{% raw %}

<template id="mytemplate">
    <img src="" alt="great image">
    <div class="comment"></div>
</template>

{% endraw %}
{% endhighlight %}

And this is how you instantiate it;

{% highlight javascript %}
{% raw %}

var t = document.querySelector('#mytemplate');
// Populate the src at runtime.
t.content.querySelector('img').src = 'logo.png';

var clone = document.importNode(t.content, true);
document.body.appendChild(clone);

{% endraw %}
{% endhighlight %}

As you can see in the examples above, the `<template>` element is a great tool for code reuse and it is at the core of the Web Components.

As we have seen in the previous post, Polymer uses the `<template>` tag in the `<polymer-element>` definition. Polymer also extends the `<template>` element but before demonstrating this, let us first build a new custom element, `<chat-message-view>`.

{% highlight html %}
{% raw %}

<link rel="import" href="/bower_components/paper-shadow/paper-shadow.html">

<polymer-element name="chat-message-view" constructor="ChatMessageView">
    <template>
        <style>
            paper-shadow {
                margin: 5px;
                padding: 10px;
                float: left;
            }
        </style>
        <div id="messages" vertical layout start></div>
    </template>
    <script>
        Polymer({
            addMessage: function(msg) {
                var shadow = document.createElement('paper-shadow');
                shadow.innerHTML = msg;
                this.$.messages.appendChild(shadow);
            }
        });
    </script>
</polymer-element>

{% endraw %}
{% endhighlight %}
[See the full source at GitHub](https://github.com/emreturkay/polymer-chat-example/blob/908e744b4556be7763e668b3e1cde7a43bbbf254/elements/chat-message-view.html)

In the code above, we are manually creating a `<paper-shadow>` element and setting the contents in the `addMessage()` function of `<chat-message-view>` element every time we receive a message. 

The Polymer adds a `repeat` attribute to the standard `<template>` tag for easily binding array data. With the help of Polymer's extensions for the `<template>` element, we can define the above process in a more descriptive way. 

First let us define a model to keep the messages, basically an array of strings. We need to initialize it in the `ready()` [lifecycle callback](https://www.polymer-project.org/0.5/docs/polymer/polymer.html#lifecyclemethods).

{% highlight javascript %}
Polymer({
    ready: function() {
        this.messages = [];
    }
});
{% endhighlight %}

Now, we can re-write our `#messages` div by using the `messages` array and `<template>` tag as below:

{% highlight html %}
{% raw %}
<div vertical layout start>
    <template repeat="{{message in messages}}">
        <paper-shadow> {{message}} </paper-shadow>
    </template>
</div>
{% endraw %}
{% endhighlight %}

Here the template tag makes sure the `<div>` contained in it is repeated for all the messages in the `messages` array and each item of the array is placed as text content of the `<paper-shadow>` element.

By using this method we can remove the `addMessage()` function. The last missing part is that we need to change our index.html file so it doesn't refer to the now obsolete `addMessage()` function. Instead of calling it, we will add an entry to the `messages` array:

{% highlight javascript %}
socket.on('chat message', function(msg) {
    view.messages.push(msg);
});
{% endhighlight %}

You can find the source of the final example at [Github](https://github.com/emreturkay/polymer-chat-example/tree/0e6148c9adff5e1d227f60ac17b9b4d6092a20fe).

## Application as a Web Component
We have already created two components, the `<chat-message-editor>` and `<chat-message-view>` and while building them we have used some Polymer elements. Similar to that, we can use our own custom components to build bigger and better composite components. To demonstrate it I have created a `<chat-app>` component containing the whole application wrapping all the visual and behavioral aspects of our chat application.

See the the `chat-app` definition below:

{% highlight html %}
{% raw %}
<polymer-element name="chat-app" constructor="ChatApp">
    <template>
        <style>
            :host {
                height: 100%;
            }
            #container {
                position: relative;
                width: 100%;
                height: 100%;
            }
        </style>
        <div id="container" layout vertical>
            <chat-message-view id="view" model="{{model}}" flex></chat-message-view>
            <chat-message-editor id="editor"></chat-message-editor>
        </div>
    </template>
    <script>
        Polymer({
            model: [],
            ready: function() {
                this.socket = io();
                var self = this;
                function sendMessage(ev) {
                    self.socket.emit('chat message', ev.detail.message);
                }
                this.$.editor.addEventListener('message', sendMessage);
                this.socket.on('chat message', function(msg) {
                    self.model.push(msg);
                });
            }
        });
    </script>
</polymer-element>
{% endraw %}
{% endhighlight %}
[See the full source at GitHub](https://github.com/emreturkay/polymer-chat-example/blob/fc2caf0b06205ff092ab2d6d58c8ceed833787d1/elements/chat-app.html)

As you can see above we have moved the `<chat-message-view>` and `<chat-message-editor>` definitions from the `index.html` file into the `chat-app.html` file.

There is one other difference from the previous code, we have moved the `model` from the `chat-message-view` component into the `chat-app` component. To do that we have defined an attribute for the `chat-message-view` component:

{% highlight html %}
{% raw %}
<polymer-element name="chat-message-view" attributes="model" constructor="ChatMessageView">
    <!-- ... -->
    <div vertical layout start>
        <template repeat="{{message in model}}">
            <paper-shadow>{{message}}</paper-shadow>
        </template>
    </div>
    <!-- ... -->
</polymer-element>
{% endraw %}
{% endhighlight %}

By moving the data model out of the `chat-message-view` component we have provided more flexibility. For example now it is easier to implement many of the homework items from the "[Getting Starting Guide](http://socket.io/get-started/chat/)" of socket.io documentation.

There is a lot more to the Polymer's way of _data binding_, but I will not go into details. You can visit the [Polymer Data Binding Overview](https://www.polymer-project.org/0.5/docs/polymer/databinding.html) for more information.
