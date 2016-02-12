---
layout: post
title: "Understanding jQuery &amp; the DOM"
date: 2016-01-27 09:29:41 -0500
comments: true
categories:
---
As someone whose first programming language was Ruby, the transition into JavaScript was rocky, to say the least. But after a week of near-constant rage fueled by missing the 16th closing curly brace and forgetting that my functions won’t do shit unless I call them, I think I’m starting to come to terms with JavaScript, and maybe even… like it a little? And that’s all thanks to jQuery, because jQuery just makes everything in JS so much easier and better, especially when interacting with the DOM.

But I definitely didn’t feel that way at first. No, at first the way I felt about jQuery was more or less: WTF are all these frigging $ about? Why did all these things I learned in regular ol’ JavaScript suddenly stop working? Why aren’t these effing jQuery methods working when I think they should? And WTF are DOM elements anyway?!?!?

... I think I get it now. So the following is the first of two posts documenting how I found the answer to what may seem (at least to those more familiar with JavaScript than I) a fairly basic question: What the hell does it mean when someone tells you that a jQuery object is an array-like collection of DOM elements? This post assumes you have basic experience with HTML, CSS, and JavaScript, but not much else.

##THE DOM##

Let’s start at the beginning: What on earth is the DOM? I had a vague understanding of the DOM as like… something in the browser… that uses HTML? But that didn’t really seem good enough. So I found the official definition from the [World Wide Web Consortium](https://www.w3.org/DOM/), the group that maintains standards for the DOM:

>The Document Object Model is a platform- and language-neutral interface that will allow programs and scripts to dynamically access and update the content, structure and style of documents. The document can be further processed and the results of that processing can be incorporated back into the presented page.

Well, thanks for nothing, W3C. Luckily, other definitions of the DOM exist out there, and have led me to this understanding of a few keys points:

+ The DOM is a **standard model**, so it is interpreted the same regardless of browser or language.
+ The DOM allows us to access and interact with **HTML elements as objects** (it is object-oriented).
+ The DOM connects web pages to our scripts by providing a **structured representation** of those objects.

In other words, the DOM facilitates our access to HTML elements by representing them as objects, and allowing us to use JavaScript (or another language) to easily access and manipulate that HTML. Let’s take a closer look at what that means for us as programmers.

*Note: The DOM isn’t only for HTML (it also has models for XML and the Core DOM) but those aren’t important for the purposes of this post.*


##DOM ELEMENTS AND JAVASCRIPT##

Here’s a visual representation of what the hell is going on in the DOM:
![DOM tree](/images/jquerydom/pic_htmltree.gif)

Source: [W3 Schools](http://www.w3schools.com/js/js_htmldom.asp)

All of those boxes represent different kinds of [nodes](https://www.w3.org/TR/DOM-Level-2-Core/core.html#ID-1950641247), which are the primary datatypes used by the DOM. In other words, nodes are the friendly little objects that the DOM turns our HTML into! There are a bunch of different node types, only a few of which are represented here, but we’re primarily interested in the `element` type, because this is primarily what we’ll be calling on in both JavaScript and jQuery methods. (There is limited support for other nodes like `text` and `comment` in SOME methods, but their use is not encouraged.)

What is a DOM element exactly? Easy: if you write it in an HTML tag, it’s an element. Here’s some elements:
```html
<body></body> <!--woah it's an element!-->
<h1>RAWR</h1> <!--omg another one!-->
<div class="magic">OMG MAGIX</div> <!--elements for everyone!-->
```
The DOM standard allows us to use JavaScript to grab these elements and operate on them.

In vanilla JavaScript, we access the `document` object (the node which encapsulates every other node) in order to find a specific DOM element. So, for instance, if we were looking for a `div` with an id of `pterodactyl`, we could use the following:

```javascript
var awesome = document.getElementById("pterodactyl");
```

and `awesome` now contains the pterodactyl `div` we were looking for. There are a [bunch of other JavaScript methods](https://developer.mozilla.org/en-US/docs/Web/API/Document#Methods) like this for working directly with DOM elements, but honestly you should probably never use them because jQuery.

##JQUERY OBJECTS##

jQuery is just a JavaScript library that makes your life way better. jQuery objects are like DOM elements only way better. No really, they are pretty much exactly the same (only better). jQuery objects are infinitely easier to work with than DOM elements in their pure form and give us access to a plethora of [super handy jQuery methods](https://api.jquery.com/category/manipulation/) that do things like allowing us to create animation and alter HTML at runtime. jQuery objects just provide a wrapper object that go around either a single DOM element or a collection of DOM elements, allowing us to use those methods.

There are way too many jQuery methods to go into here, but what I do want to explore are the ways available to get ahold of the jQuery object. The simplest is just to take a pre-existing DOM element and wrap it. That syntax looks like this:

```javascript
var element = "<p>Hey there!</p>"
$(element)
```
So that’s what all the $$$ is about! The `$()` syntax is just sugar for `jQuery()` - the jQuery wrapper object. As soon as we wrap the DOM element (or an array of DOM elements), we have access to all of the jQuery methods. The most common way you’ll see this implementation of getting a jQuery object is wrapping `$(this)` (demonstrated below), because usually you’ll want to get hold of jQuery objects using selectors.

Luckily, turns out this is also SUPER EASY. Just grab [your favorite CSS selectors cheatsheet](http://www.w3schools.com/cssref/css_selectors.asp) and dive on in! Here are a few examples of how this would be implemented:

```javascript
$('h1') // selects all h1 elements
$('#rainbows') // selects the element with an ID of rainbow
$('a[href^="https"]') // selects all a elements with an href attribute beginning with "https"
```
You can create any combination and use any CSS selectors you like. If you check it out in the console, the jQuery object you get back will look like this:
![jQuery objects](/images/jquerydom/objects.png)
In the first example I’ve selected all the elements with a class of `list`. I got three objects back, and we can see that it’s not just the tags, but all of the element’s attributes and children. The console allows you to expand and contract your view of the children.

In the second example I selected the object with an ID of `wrapper`. Note that even though it’s only a single object, is still comes with brackets around it - this array-like notation is an indication that the DOM element is wrapped in a jQuery object. (BUT NEVER FORGET THIS IS NOT AN ACTUAL ARRAY - see part two.)

A third, less common way to create a jQuery object is by wrapping an entire HTML string that does not yet exist on the page. This is less common largely because (to my understanding) it has a much more limited use case. You would want to do this primarily if you needed to call another method or act on the new object after you add it to the page. Here is an example:

```javascript
$('<li>dance party</li>').appendTo($('ul#funtimes')).on('click', function(){
  $(this).hide()
})
```
Here we actually get so see ALL THREE of the jQuery object method syntaxes we’ve gone over. Let’s take it step by step. On line one we make a jQuery object by wrapping an HTML string that will add the item "dancy party" to a list. My making this HTML string into a jQuery object we gain access to the `appendTo()` method. As an argument to this method we use a jQuery object that we create through the use of CSS selectors (it’s the `<ul>` with an id of funtimes).

Great! Our list item is now appended. But we want to be able to hide it on a click (though like, why would you want to hide a dance party?) Luckily we can call the `on()` method and, thanks to some jQuery magic, have access to a `this` that isn’t the goddamn window. On line two we see the first method we talked about, directly wrapping up a DOM element (in this case `this`), which is the first jQuery object - the new string. And this is also why we constructed our first object directly from a string rather than doing the reverse formation:

```javascript
$('ul#funtimes').append('<li>dance party</li>').on('click', function(){
  $(this).hide() // this isn’t what you think it is
})
```
If we had formulated it this way, when we referenced `this` we would have been accessing the `ul`, not the new list item we appended, and clicking would have hid the entirety of the list rather than the specific list item.

##FEELING GOOD?##
Hopefully by now you’ve got a grasp on what it means for something to be a jQuery object! But there’s plenty of quirks still to explore, so check out (the hopefully shorter) part two where we explore such fascinating topics as:
Why the hell isn’t this acting like an array?
Dammit, now I need to get the plain DOM element back.
What’s going on with `this` anyway?

##RESOURCES##

###THE DOM###
+ [W3 Schools: JavaScript HTML DOM](http://www.w3schools.com/js/js_htmldom.asp)
+ [MDN: Introduction to the DOM](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model/Introduction)

###JQUERY###
+ [jQuery API: jQuery()](https://api.jquery.com/jquery/)
+ [The jQuery Object](https://learn.jquery.com/using-jquery-core/jquery-object/)
+ [The Mystery of the jQuery Object: A Basic Introduction](https://www.smashingmagazine.com/2014/05/mystery-jquery-object-syntax-basic-introduction/)
+ [Working with JavaScript DOM Objects vs. jQuery Objects](http://howtodoinjava.com/scripting/jquery/javascript-dom-objects-vs-jquery-objects/)
