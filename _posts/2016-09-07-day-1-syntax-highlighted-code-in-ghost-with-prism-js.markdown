---
layout: post
title:  "Day 1: Syntax Highlighted Code in Ghost with Prism.js"
date:   2016-09-07 08:36:00 +0100
---
For a coding blog, code snippets are an essential requirement. Ghost's editor is powered by Markdown, where one can create code blocks multiple ways.
<!--more-->
With indentation

        This is an indented code block.

or with triple backticks:

    ```
    This is a triple backtick code block.
    ```

These blocks come without syntax highlighting by default, so some extra is needed to make it work.

## Prism.js

[Prism.js](http://prismjs.com/) is the [recommended](http://support.ghost.org/faq/syntax-highlighting/) library to highlight code in ghost. To do that you have to specify the language in a triple backtick code block, like this:

    ```javascript
    This is a JavaScript code block.
    ```

Also, you have to add `prism.js` and a theme stylesheet to the page. For that I had multiple options:

1. Link it from a public CDN
2. Upload it to my S3 bucket and link it from there
3. Add it to the blog repository
4. Add it as a dependency of the blog

My problem with the first two options is that the site would rely on some outside dependency, that is not managed in the blog's GitHub repository. Why am I so picky about this, since I store images for the posts in S3. My reasoning here is that those images are *content* and syntax highlighting is *logic*, which should be tightly coupled with the blog.

So I was left with the latter two options. Why I didn't choose 3 is not complicated. The actual code of Prism would end up in the repository, which is something I absolutely don't want. It is a dependency, so:

```bash
$ npm install prismjs --save
```

## Linking a dependency

The above command puts Prism in node_modules, but it's unreachable by the express server powering Ghost.

### Serving static files in Ghost

For this, I needed access to the express server behind Ghost. The [recommended](https://github.com/TryGhost/Ghost/wiki/Using-Ghost-as-an-npm-module#mount-ghost-on-a-subdirectory) way is to provide my own parent express app, which I first had to add as a dependency:

```bash
$ npm install express --save
```

Then I edited my `server.js` to require and create the parent express app:

```javascript
var path = require('path'),
    ghost = require('ghost'),
    express = require('express'),
    parentApp = express();

ghost({
    config: path.join(__dirname, 'config.js')
}).then(function (ghostServer) {
```

Configured it to serve static files from the `/public` folder:

```javascript
    parentApp.use('/static', express.static(__dirname + '/public'));
```

Finally, I mounted Ghost to the configured path and started it with the parent express app:

```javascript
    parentApp.use(ghostServer.config.paths.subdir, ghostServer.rootApp);

    ghostServer.start(parentApp);
});
```

Now anything in `/public` is available in `/static` from the browser, but Prism is still in `node_modules`.

### Serve Prism

To make Prism available I would have to manually copy it to `/public`, but that's as good as option 3. The solution was to ignore `public/prism.js` and `public/prism.css` in `.gitignore` and use a post-install script to copy these files to the `/public` folder.

I already had a post-install script, that moves the Casper theme to the `/content/themes` folder of Ghost, so I moved that along with the new copy commands to a `postintall.sh` file:

```bash
#!/bin/sh

ncp node_modules/Casper content/themes/casper

ncp node_modules/prismjs/prism.js public/prism.js
ncp node_modules/prismjs/themes/prism.css public/prism.css
```

## The final piece of the puzzle

Now that Prism is available, the last thing I had to do is add the theme stylesheet to `{{ghost_head}}`:

```html
<link href="/static/prism.css" rel="stylesheet">
```

and the script to the `{{ghost_foot}}`:

```html
<script href="/static/prism.js"></script>
```

in Code Injection.

You can see the result in the code blocks of this post :)
