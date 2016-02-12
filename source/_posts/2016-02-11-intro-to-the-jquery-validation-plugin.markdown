---
layout: post
title: "Intro to the jQuery Validation Plugin"
date: 2016-02-11 22:18:44 -0500
comments: true
categories:
---
So I’m a little bit in love with the [jQuery Validation Plugin](http://jqueryvalidation.org/). It provides easy, immediate validation on forms, analyzing user input as they type and providing instant feedback on validation failures in addition to preventing a form from submitting if it contains any invalid data. However, I found the [documentation](http://jqueryvalidation.org/documentation/) a little lacking, especially for a JavaScript/jQuery beginner like myself. So I wanted to share a step-by-step implementation guide with a focus on customization. Here’s an example of it in action on [Romp](https://romp.herokuapp.com/), a social networking application I built with [@eaud](https://github.com/eaud/) and [@japplaba](https://github.com/jappleba/) in Rails along with jQuery and AJAX:

<iframe width="420" height="315" src="https://www.youtube.com/embed/96TPLC96DGw" frameborder="0" allowfullscreen></iframe>


##DOWNLOAD & INCLUDE IT##

Okay, this probably seems ridiculously basic, but this was the first jQuery plugin I’d ever installed! It’s available on the [jQuery Validation website](http://jqueryvalidation.org/). Note that the zip file you download includes a demo and a whole mess of other files, but the one you actually need is `jquery.validate.js` inside of the `dist` folder (you may need some of the other files in this folder depending on how complex you’re getting, but this was all I used). If you’re not using Rails, be sure you’ve also included jQuery in your app. If you are using Rails, the manifest includes it for you.

In Rails, all you have to do is pop the `jquery.validate.js` into `/app/assets/javascripts` and the manifest file will require it for you. In case you aren’t familiar with it, the manifest file is at `application.js` and under NO condition should you delete the comments!! The special `//=` notation is requiring those files for you, and the `//= require_tree .` is what requires everything in the folder, grabbing you jQuery validation plugin you just included.

You can include your validation code in any file in the `javascripts` folder, just be sure to change the extensions from `.coffee` to `.js`. Rails will automatically compile of your scripts into a single file to be uploaded into your application.

##FIND YOUR FORM##

If you’re like me you’re going to customize everything about your validations, so that’s how I’m going to walk through this. There are shorter, less customizable options available, and I would recommend checking out [this video](https://youtu.be/yaxUV3Ib4vM) if you’re interested in that option.

The first thing to do is select the form you’ll be validating using a [jQuery object](https://kaylee42.github.io/blog/2016/01/27/understanding-jquery-and-the-dom/), and make sure it’ll be called when the document is ready, for instance:

```javascript
$(document).ready(function(){
    $("#activityForm").validate()
});
```

The `.validate()` method comes with the plugin and we’ll flesh it out in just a second. Note that if you are using Rails form helpers, you can customize the `id` attribute in the html hash like this:

```ruby
<%= form_for(@activity, html: {id: 'activityForm'}) do |f| %>
```

`.validate()` takes an argument of a hash which will dictate what validations are run and what messages are provided, and allow for the customization of error messages and more. Let’s take a look.

##WRITE YOUR RULES##

First and most importantly we need to decide what we’ll be validating. Take a look at this example validation hash:

```javascript
$("#activityForm").validate({
    rules: {
      title: “required”,
      "activity[name]": {
        required: true,
        minlength: 2,
        maxlength: 60
      },
      "activity[guest_min]": {
        required: true,
        digits: true
      },
      "activity[guest_max]": {
        required: true,
        digits: true,
        greaterThanMin: true
      },
    },
});
```
Here we can see a few different options for declaring our validations. Note that validations are called on the `name` attribute for your form input. If you are using Rails, you should NOT customize the names of your form inputs (even though you can) - it will completely throw off ActiveRecord and prevent you from utilizing mass assignment from the `params` hash. If you aren’t sure what `form_for` or `form_tag` generated as a name attribute for you, I’d recommend using the [Chrome Developer Tools](https://developer.chrome.com/devtools) to inspect each element and copy the name attribute directly into your validations hash. Note above that non-nested names can simply be written as symbols, while nested names need to be wrapped in quotes like so: `"activity[tags][]"`

If all you need to do is check that any sort of input was entered for a given field, you can simply write `field_name: “required”`. Easy!! But if you want to do anything more than that, you include a hash of different requirements as demonstrated in the example above. There are way more examples than provided here, and you can find some of them [here](http://jqueryvalidation.org/category/methods/). (I’ve yet to find a comprehensive list of every method available, so if you have one please let me know! If you feel like it’s a validation method that should exist, just Google it and it’ll probably come up.) The ones I’ve included should all be pretty self-explanatory, with the exception of the last one, `greaterThanMin`.

This is a custom method we wrote to ensure that the maximum number would be greater than or equal to the minimum. Here’s what it looks like:

```javascript
jQuery.validator.addMethod("greaterThanMin", function(value, element) {
    return this.optional(element) || (parseFloat(value) >= $('#activity_guest_min').val());
}, "* Amount must be greater than min guests");
```

Let’s break down what’s happening here. `jQuery.validator.addMethod()` comes with the plugin and allows us to write our own custom methods. Yay! We then pass in three arguments: the name of our method (as a string), the validation method (a function), and a default error message (again a string). Both the name and error message are simply strings (and the error message is only a default - we’ll go over how to override them in the next section). But let’s talk more about what’s going on in the function.

The function should take two arguments - `value` and `element`. `value` is the same as what you’d get if you called `.val()` on the form input (it’s whatever the user typed). `element` is something that you won’t need to manipulate, but you DO need to include it in order to have the first half of the conditional: `this.optional(element)`. You should DEFINITELY maintain this conditional format when writing your own methods. Basically, `this.optional(element)` will evaluate to true if `element == undefined`, so the custom validation will never be run if nothing is entered. This is good as it allows you to make the input required separately, both making this method more flexible and providing more accurate error messages to users. So you just write your custom method in the bit after the `||` - in our case checking to ensure that `value` is greater than the minimum value entered.

##WRITE YOUR ERRORS##

All validation methods come with default error messages - however they tend to be pretty bland. All required fields will automatically return the error “This field is required.” More specific rules do provide more specific information, like “Please enter at least 2 characters,” but you may want something fancier than that. Luckily custom error messages are SUPER EASY to write. In your `.validate()` hash, immediately below all of the rules, just include something like this (rules cut out for brevity):

```javascript
$("#activityForm").validate({

    rules: {
   ...
    },
    errors: {
      title: “Hey, you need a title!”,
      "activity[name]": {
        required: "Your activity must have a name, silly!",
        minlength: "One letter isn't a name...",
        maxlength: "Keep it short & sweet."
      },
      "activity[guest_min]": {
       required: "Let us know how many people can come!",
        digits: "It's gotta be a number"
      },
      "activity[guest_max]": {
        required: "Let us know how many people can come!",
        digits: "It's gotta be a number",
        greaterThanMin: "Must be greater than or equal to minimum guest number"
      },
    },
});
```
So, so easy. Just create a hash that’s identical to your rules hash, and replace the constraints with the error message you would like to appear. If you skip any no worries, the default messages will still show up.

By default error messages will show up immediately after the input, but you can change this! (There are other customizations available which you can check out [here](http://jqueryvalidation.org/validate/) - this is the only one I’ll cover). We ran into an issue where the placement of the error message with our checkboxes was really messy. However, customizing it was pretty easy. Here’s what we did to move the error message from the bottom to the top:

```javascript
  $("#activityForm").validate({
    errorPlacement: function(error,element) {
      if (element.attr("name") == "activity[tag_ids][]"){
        error.insertAfter($(".tag-checkboxes"));
      }else{
        error.insertAfter(element);
      }
    },
  rules: {...},
  errors: {...}
});
```
As you can see, the custom `errorPlacement` is just popped into the beginning of our validation hash (rules and errors cut for brevity). It takes two arguments: `error` and `element`. It’s important to use an if statement in your function call unless you’d like to change the placement of all of your error messages. In the if statement, use `element.attr("name")` to find the specific input element that you would like to adjust error placement for. Then you can use jQuery’s `.insertAfter()` or `.insertBefore()` (or really whatever you’d like) to customize the error placement. Be sure to include an else statement containing `error.insertAfter(element);` in order to maintain the default error placement for all other error messages.

##BONUS ROUND##
jQuery Validate plays super well with HTML5 form validation, which means that validation on new input types like URL and email are automatically run, saving you the headache of writing long obnoxious RegExps. Note that in order for this to happen jQuery Validate automatically adds `novalidate="novalidate"` to your form tag - don’t freak out, this just prevents the HTML5 validations from being run in lieu of the jQuery Validate functions.

Customizing the way errors appears is also incredibly easy. They’ll automatically just be standard text, but you can customize the text’s appearance by adding CSS to the `error` class. Here’s what we used to create the errors shown above:

```css
.error {
  color: #E4003D;
  font-style: italic;
}
```
Simple, right? Hope this was a helpful guide to getting started with jQuery Validate. Please remember that even though these validations prevent a form from submitting if it contains bad data, it’s still best practice to include validations on your database as well to ensure that no bad data gets in! If you’d like to see the complete code from the example at the top, you can check it out on our [GitHub repo](https://github.com/eaud/BookItList/blob/master/app/assets/javascripts/xform_validations.js).
