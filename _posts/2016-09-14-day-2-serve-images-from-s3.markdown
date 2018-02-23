---
layout: post
title:  "Day 2: Serve Images from S3"
date:   2016-09-14 16:57:00 +0100
---
I started [Mozipremierek.hu](https://mozipremierek.hu), my Hungarian movie premieres calendar more than two years ago. It went through a lot of changes lately, one of which was moving the image content to S3.

## From hosted to IaaS to PaaS

Mozipremierek was born on a hosted service. back then it was fine, but after a while, the environment started feeling more and more restricting, and I also needed space to store the increasing number of image content, so I moved it to a DigitalOcean droplet.

After another 8 months, I realized that managing the virtual server and building my own deployment flow is too much work. I'm not a hardcore sysop, and I have no plans to become one. Fine tuning and optimizing the server to handle secure connections efficiently was a pain and I didn't even try to set up my own mail server. I wanted to focus on development.

[Heroku](https://heroku.com/) gave me all the above and more out of the box (or with one click)... along with its [ephemeral file system](https://devcenter.heroku.com/articles/dynos#ephemeral-filesystem). You're not supposed to rely on storing files in a Dyno, so with all the good things came a problem. I had to move my files to a CDN. My choice was Amazon Simple Storage Service, a.k.a. [Amazon S3](https://aws.amazon.com/s3/).

## Baseline

So, how it worked before the move was really simple. I created a routing for `/poster/[movie_id]-[movie_slug].[media_id].jpg` in the server logic. When a client accessed this endpoint a file was generated with the same path, so later the web server served the generated file. It worked with uploaded files too, the only difference was that the source was a file on the server, not an external URL.

## Uploaded files to S3

The first step was to move the uploads to S3. I created a command, that uploaded the existing files. Next, I changed the movie manager to save new files to S3 instead of the server and modified the poster routing to search for the uploaded files on S3 instead of the upload folder. With that my biggest problem, storage space was gone.

## Generated files to S3

My next thought was, that I could even move the generated files, thumbnails, blurred images to the CDN too. It was also really simple, I uploaded the generated files to S3 and changed routing to search for them there, and if found, redirect to the S3 resource URL.

I learned two really important options here. First, if I want to make my files stored on S3 public, I have to set the `ACL` (Access Control Level) to `public-read`. Second, S3 only serves headers that I set, so to enable client side caching I had to set `CacheControl` to `max-age=290304000, public`.

Everything seemed fine until I saw this on [NewRelic](https://newrelic.com/):

![](https://s3-eu-central-1.amazonaws.com/coderdays/2016/09/Screen_Shot_2016_09_14_at_13_00_09-1473850828796.png)

The light green stuff... that's request queueing. The single Hobby Dyno was having issues serving the many many image requests, even though they are mostly redirects to S3. Should I upgrade to a higher tier with bigger request pool... or offload traffic?

## Offloading traffic to S3

What does it help if you have images in S3 if you proxy them? It still takes up one request for the server. What I had to do is directly link the S3 resources. Now this wasn't that easy.

Every movie has a generated poster image, a thumbnail and a blurred poster. I created a field for each of these in the database and changed the poster routing to store the S3 URLs of the generated files. I also had to change the handling of these images in the whole application to use the stored URLs if they are set, or else the routing path which will generate them. This is essentially still the same cache logic I described in the baseline.

![](https://s3-eu-central-1.amazonaws.com/coderdays/2016/09/Screen_Shot_2016_09_14_at_13_16_10-1473851798524.png)

You can see what cold cache did after I deployed this change. Apdex index hit rock bottom, but after a few minutes things were back to normal... or were they?

## Slower than before

After the cold cache peak, you can see that there is less light green stuff, but there are much more blue, and the average page load is several times longer than before, usually more that 1000ms. That's because I broke page caching.

I cache the index page HTML since it takes a while to generate it. What I didn't mention before is that every time I save a generated poster URL I reset this cache so the page could render with the generated S3 URLs. So every time a poster was loaded that had no S3 URLs the cache was reset.

Now you could ask: hey, but the during the cold cache peak all images for the index page were generated, right? Yes, and no. The index page only requests the thumbnails and the blurred posters, so opening a movie page that uses the full-size poster reset cache. Old movie pages, that don't show up on the index are frequently visited too. When their posters are generated, they also reset the cache. (Figuring out when not to reset the cache would have added more overhead.) Add some nasty bugs to the mix and you're cooking crap, big time.

## Final steps

I created a console command that generated posters for every movie in the database (right now it's around 800). Squashed the bugs, made some optimizations and finally reached the point where there was no frequent cache reset.

![](https://s3-eu-central-1.amazonaws.com/coderdays/2016/09/Screen_Shot_2016_09_14_at_13_37_09-1473853045244.png)

The average transaction time is now back to the below 500ms range, it even stays around 250ms for most of the time, which is a clear improvement from where I started. There is still room for improvement, but I don't feel the need to upgrade from Hobby tier for a hobby project... for now.
