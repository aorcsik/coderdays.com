---
layout: post
title:  "Day 3: Ghost and DISQUS Comments"
date:   2016-09-21 18:22:00 +0100
---
The Ghost CMS doesn't have comments by default. Some might consider this a missing feature, but I don't. Why would you want to complicate your CMS with comments, if there are really nice solutions available, like [DISQUS](https://disqus.com/).

## Comments as a Service
DISQUS focuses on one thing, comments, and they do a great job. You sign up for an account, create a site, install their widget, sit back and wait for comments to arrive.

Installing is quite easy. For popular platforms, like WordPress, they have plugins, but it can be embedded on any page following their simple tutorials. [They even have one for Ghost.](https://help.disqus.com/customer/portal/articles/1454924-ghost-installation-instructions) But... it tells you to make changes to the theme you're using. I don't like that. So, how should I do this without touching the theme?

## Do it with JavaScript!
I took a closer look at the code in the tutorial. It's basically embedding a script in the page header, which looks for global variables and DOM elements on the page. OK, then I simply set the variables and place the DOM elements before embedding the script asynchronously. Well... almost, let's see how it is done.

### The comment widget
For the comment widget, there needs to be an empty container with the id `disqus_thread`:

```html
<div id="disqus_thread"></div>
```

and two global variables:

```javascript
var disqus_shortname = 'coderdays';
var disqus_identifier = '???';
```

The first is the short name of the site, in my case `coderdays`. The second one is a unique identifier for the post. It is optional, but recommended, because if it's missing DISQUS uses the page URL as the identifier, which could change over time. The post id is usually a good identifier, so I looked at the generated HTML code of a post to find it... but it's not there. Not unless I make it! So I added a hidden input field at the end of the post with the post id.

```html
<input type="hidden" id="disqus_identifier" value="6">
```

By the way, this is particularly handy, because only pages with the hidden id would have a comment widget.

For the magic to happen, I had to have the above little pieces in place and load the embed code from DISQUS:

```javascript
$(function() {
    window.disqus_shortname = "coderdays";
    if ($("input#disqus_identifier").length > 0) {
```

After page load, I set the global `disqus_shortname` and search for the `disqus_identifier` hidden field.

```javascript
        window.disqus_identifier = $("input#disqus_identifier").val();
        var $disqus_thread = $("<div id=\"disqus_thread\"></div>").appendTo($("#disqus_identifier").closest("article"));
```

If there is any, I set its value for the other global variable with the same name. Then I create the empty `disqus_thread` container and place it just at the end of the `<article>` tag, that contains the hidden field.

Here comes the last step, loading the embed script:

```javascript
        var disqus_loaded = false;
        var load_disqus = function() {
            if (!disqus_loaded && $disqus_thread.offset().top > $(window).scrollTop() + $(window).height()) {
                disqus_loaded = true;
                $.getScript('//' + window.disqus_shortname + '.disqus.com/embed.js');
           }
           return disqus_loaded;
       };
       if (!load_disqus()) {
           $(window).on("scroll.disqus", function() {
               if (load_disqus()) $(window).off("scroll.disqus");
           });
        }
    }
});
```

It might seem a bit complicated, but for good reason. I want to load the script once and only if it's scrolled in view. To achieve this, `load_disqus()` checks if the container is visible and loads the script. If the initial load fails, then I bind a scroll event handler to keep trying loading the widget. If successful I unbind the handler.

### Comment counters

The other nice thing DISQUS provides is comment counters. This is also available through an embed script, with some DOM preparation. It converts links like this one:

```html
<a href="{{url}}#disqus_thread">Comments</a>
```

into counters. The `{{url}}` should be the URL of the post. These are best to be placed on list pages, around the meta section.

I used JavaScript here too to add the required HTML tags to the page but had to take a slightly different approach. First I tried the method I used for the widget: add tags, embed script, but for some reason, this didn't work as expected.

What worked, is embedding the script first:

```javascript
$(function() {
    window.disqus_shortname = "coderdays";
    if ($("article footer time").length > 0) {
        $.getScript('//' + window.disqus_shortname + '.disqus.com/count.js').done(function() {
```

then adding the links to the page:

```javascript
            $("article").each(function() {
                var url = $(this).find(".post-title a").attr("href");
                $("<a class=\"post-date\" href=\"https://" + window.location.hostname + url + "#disqus_thread\">Comments</a>").insertAfter($(this).find("footer time"));
            });
```

and finally calling the below function, which renders counters asynchronously:

```javascript
            DISQUSWIDGETS.getCount({reset: true});
        });
    }
});
```

I put both the widget and the counter snippets to the `{{ghost_foot}}` in Code Injection, and that's it. I managed to add comments to my blog without touching the theme itself. Mission accomplished!

You can check the result here on Coder Days, and feel free to leave a comment! :)
