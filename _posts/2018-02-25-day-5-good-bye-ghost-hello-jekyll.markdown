---
layout: post
title:  "Day 5: Good Bye Ghost, Hello Jekyll!"
date:   2018-02-25 20:08:00 +0100
---
<img src="https://s3-eu-central-1.amazonaws.com/coderdays/2018/02/coderdays-ghost-jekyll.jpg" class="main-image">

Some of you, who read the [Prologue][1], might remember that back in 2016 I started this new blog mainly because I wanted to use Ghost, the markdown based lightweight blog engine. It's 2018 now, and I had to make some decisions.
<!--more-->
### Good bye Ghost and Heroku

First of all, Ghost went 1.0, which is great, but this new version doesn't really play nice with Heroku. So either I'm stuck with 0.11 LTS or find some other place to host the blog.

Second, to keep the blog up and running 24/7 I had to use a Hobby dyno on Heroku, which costs money. Paying $7/month for a blog that I don't even use is more than luxury, it's throwing money out the window. That had to change.

So I did the math and started looking for a free alternative.

### Hello Jekyll and GitHub Pages

Last year I finally shut down the droplet I was using for [aorcsik.com][2]. It was kind of the same story: I was paying for the virtual machine that had a less than secure MySQL and my static homepage on it. The database moved to Amazon RDS and the homepage... to GitHub Pages.

[GitHub Pages][6] is great for static websites. All you have to do is build the site on the `gh-pages` branch, commit, push and it's out there. You can even configure it to use your own domain. With CloudFlare it's only a few steps and you get HTTPS as well.

So that's nice, but I have a blog, which is dynamic... right? Wrong! Sort of... well, it's like how you look at it. From the reader's perspective a blog is basically a collection of articles, which is pretty much static. Now for the writer, it's a CMS, which is a complex application, but do I need a CMS? Let me introduce Mr Jekyll.  

[Jekyll][7] is a static site generator, written in Ruby. It takes some configuration files, some liquid templates and your content, written in markdown, textile or good old HTML and builds the whole static site for you. Basically it *compiles* a blog from source. Even better, GitHub Pages can run the Jekyll build when publishing your site. The result: super fast static content, no moving parts, pretty urls hosted for free. **Where do I have to sign!?**

### Let's move!

Doing the groundwork is not so complicated, only a few things to be careful with. First, to use my domain name, the GitHub repository has to have the same name. So I created it with no initial content. Next, I installed Jekyll and created a new blog. GitHub has a [pretty good walkthrough][3], so I recommend using that. Then I set the custom domain in GitHub settings, configured CloudFlare ([with a little more help from GitHub][4]) and finally I had it online.

Moving the content was easy. Ghost stored them in markdown, the images are on S3, but still it was a bit off first, so I had to understand how a Jekyll blog works: [RTFM][5], seriously, there was everything I had to know. Trust me, I created some ugly *javascript-redirecting-page-to-post-nonsense* just to preserve the URLs of my old posts, because in Jekyll posts have the date in the path... right... unless you change that in the config, which was there in the **MANUAL!** So read it.

And that was it. I'm here writing this post in Atom, and I'm planning to do this more often for a change.

*Cheers!*

[1]: {{ site.baseurl }}{% link _posts/2016-09-02-prologue-ghost-on-heroku-how-this-blog-was-born.markdown %}
[2]: https://aorcsik.com
[3]: https://help.github.com/articles/setting-up-your-github-pages-site-locally-with-jekyll/
[4]: https://help.github.com/articles/setting-up-an-apex-domain/
[5]: https://jekyllrb.com/docs/home/
[6]: https://pages.github.com/
[7]: https://jekyllrb.com/
