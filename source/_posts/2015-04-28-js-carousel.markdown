---
layout: post
title: "JS: Carousel"
permalink: "js-carousel"
date: 2015-04-28 14:37:00 -0500
categories: web-development
---
Flatiron School ended last Friday and now most of my classmates and I are in a flurry of interviews. So far I've found that there's been a lot of JavaScript related questions so I figured I would practice a little bit on JSFiddle. I stumbled through pseudo-coding a carousel in a recent interview and got curious on how I would implement it. I am a developer after all, so I made one.

__tl;dr__ The main concept is that you have items that are hidden and you slide them in/out of view based on what button is clicked. Here's the <a href="https://jsfiddle.net/xosu6wLy/3/" target="_blank">JSFiddle</a>.

### HTML

The HTML structure is pretty straight forward. There's an outer container that will holds 2 div containers; the first contains the panels that rotate through the view and the second contains the buttons that the user can click through to get to a certain set of panels.

{% highlight html %}
<div class="outer-container">
    <div class="panel-container">
        <ul id="panels" class="clearfix">
            <li class="panel" data-index="0">1</li>
            <li class="panel" data-index="1">2</li>
            <li class="panel" data-index="2">3</li>
            <li class="panel" data-index="3">4</li>
            <li class="panel" data-index="4">5</li>
            <li class="panel" data-index="5">6</li>
        </ul>
    </div>
    <div class="button-container">
        <ul id="buttons" class="clearfix">
            <li class="button" data-panel="0"></li>
            <li class="button" data-panel="1"></li>
            <li class="button" data-panel="2"></li>
            <li class="button" data-panel="3"></li>
        </ul>
    </div>
</div>
{% endhighlight %}

Inside the panel and button containers, I created an unordered list with the panels/buttons as list items. The classes and the data attributes are used for styling and JQuery so make sure to include those.

### CSS
Half the battle is setting up the html elements to sit where you want. There are some key parts so I'll highlight them here.

{% highlight css %}
/* General Styling */
.outer-container {
    background: blue;
    overflow: hidden;
    width: 190px;
    margin: 0 auto;
}

ul {
    list-style-type: none;
    padding: 0;
    margin: 0;
}

ul li {
    float: left;
    display: block;
}

.clearfix:after {
    content:" ";
    visibility: hidden;
    display: block;
    height: 0;
    clear: both;
}
{% endhighlight %}

The `outer container` should have it's overflow set to hidden so that the panels that extend beyond what you want to show are hidden. It also should have a set width. I hard-coded mine to fit exactly 3 panels. The `ul` has some general styling like removing the bullet points. The `li` elements are set to float left and display block so that they line up horizontally. Lastly, we have the famous `clearfix` because we used float on the list item elements.

{% highlight css %}
/* Panel styling */
 .panel-container {
    width: 370px; /* should fit all panels horizontally*/
    position: relative;
    top: 0;
    left: 0;
}
.panel {
    background: red;
    height: 50px;
    width: 50px;
    margin: 10px 0 10px 10px;
    text-align: center;
}
{% endhighlight %}

For the panels, we need to style the `panel-container` to a set width so that all of the panels fit horizontally. Otherwise, the panels will move down to the next row. I also set the position to relative so that the JQuery will animate the container left and right relative to the parent container. The `panels` themselves can be formatted however they need to be. For the sake of the example, I set fixed heights on them.


{% highlight css %}
/* Button styling */
 .button-container {
    margin: 0 auto;
    width: 96px;
}
.button {
    background: green;
    height:20px;
    width:20px;
    border-radius: 50%;
    margin: 5px 2px;
    cursor: pointer;
}
{% endhighlight %}

The buttons don't really matter in terms of the styling, but to center the `button-container`, I gave it a fixed width and the good ol' margin auto trick. Additionally for the `button` I used cursor pointer so that they look like real clickable buttons.

### JavaScript/JQuery

Here's where the real fun is. There are plenty of other ways to implement this, but to make things easier, I created HTML data attributes on both the panels and buttons to figure out which buttons relate to which panels. First of all, on document load, we need to bind an event listener to the buttons using the following code.

{% highlight javascript %}
$(function () {
    $(".button").on("click", carousel);
})
{% endhighlight %}

The callback function called `carousel` will be triggered when a user clicks a button.
{% highlight javascript %}
function carousel() {
  var index = $(this).data("panel");
  var width = index * $("li[data-index=" + index + "]").outerWidth(true);
  scrollView(width);
}
{% endhighlight %}

The purpose of the `carousel` function is to determine which button was clicked and how many pixels it should move the `panel-container` to the left. After it determines that, it will call a function called `scrollView` providing the amount of pixels to scroll.
{% highlight javascript %}
function scrollView(pixels) {
  var $panelContainer = $(".panel-container");
  var shift = pixels * -1;
  $panelContainer.animate({
      "left": shift + "px"
  }, "slow");
}
{% endhighlight %}

Finally the `scrollView` function will take the given input and animate the scroll!

### Conclusion
There's a lot of libraries out there to do some front-end animation, but it's really important to know how to build it from scratch. It could be intimidating at first, but once you break down the concepts, it's pretty straight forward. Until next time, happy coding!
