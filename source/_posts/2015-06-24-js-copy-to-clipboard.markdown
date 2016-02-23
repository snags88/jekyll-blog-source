---
layout: post
title:  "JS: Copy to Clipboard Button"
permalink:  "js-copy-to-clipboard"
date:   2015-06-24 11:28:00 -0500
categories: web-development
---
A while ago (as in a month) I was doing a code challenge to create a [bitly](href="https://bitly.com/"){:target="_blank"} clone. One of the features I wanted to include on my website was the ability to press a button and have the shortened URL copied to your computer's clipboard. I thought "Easy, no problem. Just have to use JavaScript magic." WRONG! The user's clipboard is not accessible from the web browser due to security concerns.

Well not entirely wrong. One of the workaround for this issue was found by Jeffery Larson and it involves using JavaScript. What you can do is embed an Adobe Flash object into the DOM, send it the text to be copied, and voilà! Although this hack has been patched in the latest versions of Adobe Flash, there is a JavaScript library called [ZeroClipboard](href="https://github.com/zeroclipboard/zeroclipboard"){:target="_blank"} that gets around the patch. From what I've read, it was actually created by staff members at Github so users can copy URLs and SHA codes easily.

 Anyways, enough talking. Let's actually go through how to implement ZeroClipboard in a Rails app.
<p style="text-align: center;">♦ ♦ ♦</p>
First things first, we have to require the ZeroClipboard library. Thankfully there is a gem called __zeroclipboard-rails__ that we can add to our project's __Gemfile__. Afterwards, we just need to require __zeroclipboard.js__ in our assets file:

{% highlight ruby %}
# Gemfile
gem 'zeroclipboard-rails'
{% endhighlight %}
{% highlight javascript %}
// app/assets/javascripts/application.js
//= require zeroclipboard
{% endhighlight %}

Next, we'll go ahead and set up the HTML so that we have a "Copy" button with a specific ID and some text for us to copy.
{% highlight html %}
<p id="text-to-copy">This text will be copied.</p>
<button id="copy-to-clipboard">Copy!</button>
{% endhighlight %}

Now we get to jump into JavaScript. For this example I'm using the jQuery library for simplicity sake. However, this can be accomplished with vanilla JavaScript. The first thing we need to do is instantiate a ZeroClipboard object while passing in the DOM element that represents the "Copy" button. Then after the document loads, we'll create an event listener to listen for a __copy__ event and pass it a function to trigger when the event occurs.
{% highlight javascript %}
$(function(){
  var client = new ZeroClipboard($("#copy-to-clipboard"));
  client.on("copy", copyToClipboard);
});
{% endhighlight %}
We haven't defined the __copyToClipboard__ function yet, but we'll do that next. This particular one is hard coded to get the text in the &lt;p&gt; tag, but you can configure this function to be more dynamic.
{% highlight javascript %}
function copyToClipboard(event) {
  var clipboard = event.clipboardData; //access system clipboard
  var text = $("#text-to-copy").text(); //set text to copy variable
  clipboard.setData("text/plain", text) //write new data into system clipboard
}
{% endhighlight %}
Now, if you put all that together, you get a fully functioning "Copy" button that copies stuff onto your system clipboard. How cool is that? If you want to see this in action, go check out my <a href="https://shorty-app.herokuapp.com/" target="_blank">Shorty App</a> and the <a href="https://github.com/snags88/url-shortener" target="_blank">public repo</a>.  Until next time, happy coding!
