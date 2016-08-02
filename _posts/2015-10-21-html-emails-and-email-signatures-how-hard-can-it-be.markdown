---
layout: post
title:  "HTML emails & email signatures, how hard can it be?!"
description: "Maybe it's about time to fix HTML email templates. Can we make writing CSS easier? What about ditching templates? Are we doomed forever?"
date:   2015-10-21 02:23:32+02:00
size: 12
sizeDesktop: 4
textColor: grey-50
metaColor: grey-600
comments: true
author: dan
---

## TL;DR

I put together some gulp stuff to generate simple HTML email signatures / templates & hope others will find it useful. Check out [the repo][repo]. There's plenty to do to improve it, so fork away! The current samples look like this:

![Light Signature][light-img]

![Dark Signature][dark-img]


## Intro

A few weeks ago I was listening to the the JavaScript Jabber [episode 177][ep-177] with [Oren Rubin][shexman]. For those that know nothing about Oren, he's been developing stuff for the past 20 years. In his career he worked with machine learning, distributed computing and compilers, just to name a few. He moved on to web development afterwards, which he admits in the podcast was 'the really hard stuff'. I know many of you would be tempted to close this article, run outside and burst into tears of joy... but there's actually more to this post!

Web development **is hard** indeed. It also seems like it's not going to get easier anytime soon. Hell, the next generation of web developers won't be able to work on a so called 'web application' without using some sort of transpiler. You will be the laughing stock of the web if you are still writing in ES5 (or ES6 in about a year or so). The days of writing code in Notepad++ and refreshing a page in a browser are gone and they are not coming back.

So... you need to know all the new stuff, but that's not enough. You also need to know all the old stuff, because the web is not your colleague with a MacBook Pro using the latest version of Chrome. The web is any device with an internet connection, so we have to develop accordingly. I'll say it again:

> Web development is hard.
> Damn, web development is hard!

I believe the web today is not much different from the web 10 years ago (in a philosophical sense at least), however the 'what' and 'how' of web development have changed quite a lot. I am not talking about JS frameworks here or smartphone apps, but the stuff 'under the hood': the way we think of progressive enhancement (after all, we should learn from past mistakes), mobile-first designs, responsive layouts, etc.

You see, a lot of these revolutionary changes are strongly related to layout. In fact, web layout is changing again as we speak (web-time, not human-time). We moved from tables, wacky flash sites knew how to navigate, grids, now (almost) flexbox, tomorrow CSS shapes / multi columns layouts and who knows... as VR enters mainstream - some kind of 3D layout that we can't even imagine yet.

(I get it, I'm bad at intros but I'll make my point super soon, promise)

Before we look ahead, we need to take a good look back. We need to focus on a place forgotten by the web... a dark and scary box that gives me shivers when I think about it. I am of course talking about `email`, the last medium on the web were table layouts are the norm.

We need to take a good look at it and face the truth - we messed up.

Don't get me wrong, there has been progress, but in general email clients still suck. If there were some 'html email standards' the story might have been similar to the browser one, but there aren't. Everybody is free to do whatever they want. <br/> I am not thinking about crazy things here. It would be enough if we could ditch table layouts and inline styles. I would also be happy if email clients would have some basic support for media queries. Sure, it would be even nicer if we were able to do CSS3 animations or whatnot, but that's not what I really care about.

It's hard enough to work with nested tables as it is. When you throw a sausage of inline CSS in the mix, various techniques to punch your computer will suddenly become appealing to you.

## The bad news

There's no way to get rid of tables / nested tables. There is just hope and tears. I'll point out a very unlikely solution later in the article.

## The good news - In your face, inline CSS

As you probably figured out from the intro, I got fed up with all of this. That inline s\*\*\* is impossible to maintain. So I started on a very basic (I thought) gulp build system to reduce my increasingly high blood pressure. About 10 npm modules later, I realized this entire business could take a couple of hours.

My attempt to destroy inline styles and attributes was fairly successful. I used [gulp-inline-css][gulp-inline-css], thanks to [@jonkemp][jonkemp].

<small>By the way, inlining CSS is the new black (performance benefits), check out [this cool lib][critical]</small>.

Unfortunately, you can't get rid of all of them. I learned that if you wish to be on the safe side, you should use attributes whenever you can (progressive enhancement, remember?). Here are some of the things that don't work:

- **width** will be added as an attribute only if it's provided in pixels
- **height** won't be added as an attribute at all
- **background-color** won't become **bgcolor**
- **text-align** won't become **align**
- **vertical-align** won't become **valign**
- etc. See [this issue][inline-css-issue-8] for more

On the flip side, **border**, **cellpadding** & **cellspacing** will work just fine.

## Re-using templates

It's common that footers or headers are redundant across templates. I figured that the ability to include templates would come in handy, so I installed the awesome [gulp-preprocess][gulp-preprocess] plugin.

I could then include files by writing up some HTML comment directives (e.g. <em>&lsaquo;!-- @include head.inc.html --&rsaquo;</em> and that's about it. There are other useful directives, for displaying stuff from a configuration file you can use <em>&lsaquo;!-- @echo foo --&rsaquo;</em>.

If pre-process was a dependency anyway, I figured might as well go deeper and do a configuration-based setup. Fast-forward a few npm modules and I ended up with a build system that can take multiple templates & can handle contact info for a team (if you want it too). Here's an overview of how it works:

![Diagram: overview of what the build process can do][diagram-img]

## Going forward

If gulp was already there I thought I should also add the classical pre-build clean-up & watch the HTML, CSS & configuration files for changes. I went further and minified HTML & inline styles (<em>&lsaquo;style&rsaquo;</em>), embedded images in the template (base64) for local assets and probably some other smart thing that I can't remember at the moment.

But why stop here? Let's throw everything we have at it... hell, can we even make a HTML email 'transpiler', which converts semantic HTML into a table layout & adds schema.org attributes automagically? Let's generate screenshots of tests on the popular email clients, all at the convenience of our terminal. Let's add pre-processor support... How do **you** think we can make writing email templates easier?

[Let's get this over with, once and for all!][repo-issues]

[critical]: https://github.com/addyosmani/critical
[dark-img]: {{ site.baseurl }}/assets/html-emails-and-email-signatures-how-hard-can-it-be/html-responsive-email-template-signature-dark.png
[diagram-img]: {{ site.baseurl }}/assets/html-emails-and-email-signatures-how-hard-can-it-be/html-responsive-email-template-build-diagram.png
[ep-177]: https://devchat.tv/js-jabber/177-jsj-ui-validation-with-oren-rubin/mini_player
[gulp-inline-css]: https://github.com/jonkemp/gulp-inline-css
[gulp-preprocess]: https://github.com/jas/gulp-preprocess
[inline-css-issue-8]: https://github.com/jonkemp/inline-css/issues/8
[jonkemp]: https://twitter.com/jonkemp
[light-img]: {{ site.baseurl }}/assets/html-emails-and-email-signatures-how-hard-can-it-be/html-responsive-email-template-signature-light.png
[repo-issues]: https://github.com/fadeit/responsive-html-email-signature/issues
[repo]: https://github.com/fadeit/responsive-html-email-signature
[shexman]: https://twitter.com/shexman
