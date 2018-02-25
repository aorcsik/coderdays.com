---
layout: post
title:  "Prologue: Ghost on Heroku – how this blog was born"
date:   2016-09-02 23:37:00 +0100
---
Two days ago I was going over the newsletters in my inbox when I saw that [Ghost 0.10.0 was released](https://dev.ghost.org/ghost-0-10-0/). I heard about this new CMS a while back, but moving aways from WordPress and PHP seemed like a bigger jump I intended to make back then.
<!--more-->
Since then things have changed and I thought it was the perfect opportunity to jump into something new. It turned out to be a challenging one.

## Click the button...

Deploying Ghost on Heroku is easier than you think. All you have to do is:

1. Read the README on [cobyism/ghost-on-heroku](https://github.com/cobyism/ghost-on-heroku/)
2. Then click this button  
[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy?template=https://github.com/cobyism/ghost-on-heroku)

This deployed Ghost 0.9.0 with PostgresSQL, Mailgun and Amazon S3 storage on Heroku. But... I wanted *Ghost 0.10.0*. How hard could it be to upgrade?

## Upgrade to 0.10.0

First of all, the above installation uses ghost as an npm dependency. So upgrade goes by bumping `ghost: "0.9.0"` to `"0.10.0"` in `package.json`, right?

Yes, and no. Because the currently available version in npm is `"0.10.0-rc1"`. I used this exact version first but after playing around a bit with the [npm semver calculator](https://semver.npmjs.com/) I figured `"~0.10.0-rc"` would cover the stable version when Ghost Team publishes it.

Now that I had the new version I just pushed the change to Heroku and the server did... *not* start. Ghost 0.10.0 among many new features and fixes has some breaking changes too. Third party storage adapters now have to extend `BaseStore` and implement the `save`, `serve`, `exists` and `delete` methods. So I had to find an S3 storage adapter that does these, but there wasn't any, so I forked [spanishdict/ghost-s3-compat](https://github.com/spanishdict/ghost-s3-compat), which is a recommended adapter by the Ghost Team, made the necessary changes and added the repo as an npm dependency.

Ghost finally started. You can check the changes in my [pull request](https://github.com/cobyism/ghost-on-heroku/pull/77).

## coderdays.com

Next step was the domain. To my luck, it was available and registering it took less than 30 minutes.

Setting up a [custom domain for a Heroku app](https://devcenter.heroku.com/articles/custom-domains) is easy... if you have the right DNS provider. Since dynos (the virtual containers in Heroku) have dynamic IP the only source of truth is the app's domain name, which ends with `.herokuapp.com`.

## CNAME Flattening

The domain root is configured by an `A` record, which accepts an IP as value. No domain name. Forget it. What I needed was a DNS provider who supports some non-standard records like `ANAME`/`ALIAS`, or `CNAME` Flattening, which was [introduced by CloudFlare](https://blog.cloudflare.com/introducing-cname-flattening-rfc-compliant-cnames-at-a-domains-root/) who also provide free DNS so they were my obvious choice. Setup was fast, I changed the settings of my Heroku app, and voilá:

![Bad credentials](https://s3-eu-central-1.amazonaws.com/coderdays/2016/09/Screen_Shot_2016_08_31_at_20_25_46-1472846772269.png)

## HTTPS for everyone

Bad credentials are bad. I needed a valid SSL certificate. Until recently it was something you had to pay for but thanks to [Let's Encrypt](https://letsencrypt.org/) nowadays you just install certbot:

    $ brew install certbot

and request one:

    $ sudo certbot certonly --manual

When asked I provided my domain name, then came the critical part, the verification. In this step, I had to serve a certain file with a `LONG_FILENAME` and an `EVEN_LONGER_CONTENT` on the domain. The easiest way was to install express and create a simple server that serves the required file:

```javascript
var express = require('express');
var app = express();

app.get('/.well-known/acme-challenge/LONG_FILENAME', function (req, res) {
    res.send('EVEN_LONGER_CONTENT');
});

app.listen(process.env.PORT, function () {
    console.log('Example app listening!');
});
```

After successful verification, the certificates were placed to `/etc/letsencrypt/live/coderdays.com/`, then I followed the instructions in [Heroku SSL (Beta)](https://devcenter.heroku.com/articles/ssl-beta) and uploaded my certificate and private key:

```
$ heroku _certs:add fullchain.pem privkey.pem
```

As a result, I received a new domain name for my app, which I had to change in CloudFlare, then finally:

![Good credentials](https://s3-eu-central-1.amazonaws.com/coderdays/2016/09/Screen_Shot_2016_08_31_at_20_25_39-1472848865006.png)

## Mailgun

To use the custom domain for sending emails I had to configure Mailgun too. Their Starter plan includes one custom domain, so after activating the account I added coderdays.com. They provide a nice walkthrough to configure the DNS server, which was a breeze with CloudFlare. In a few minutes, the records were verified and I could successfully send a test email from Ghost's Labs section.

## Final Thoughts

I have to admit I completed the above process in two days. I didn't go into detail on how much pain I went through to set up the certbot verification before settling with the solution I presented above, or how I started over the upgrade to 0.10.0 three times because it just didn't want to work. These are also part of the story, so they deserved to be mentioned.
