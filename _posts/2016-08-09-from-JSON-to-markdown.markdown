---
layout: post
title:  "We've gone static (again)"
description: "We went from JSON (Angular) to markdown (Jekyll). Is this going to boost our blogging adventures though?"
date:   2016-08-09 08:22:30+01:00
size: 12
sizeDesktop: 6
cover: "assets/from-JSON-to-markdown/cover.gif"
textColor: grey-50
metaColor: grey-600
comments: true
author: fadeit
tags: [jekyll, angular, static site generator, json, performance, amp, blog]
---

It's a big day today: we've gone Jekyll - the state of the union static site generator. 

Yes folks, the fadeit blog has been dismembered. It is not more but a puny appendage, isolated from the rest of the fadeit universe.
Call it the great fadeit schism if you will. But why did we do it?<br/>
More on that in a second.


Let me tell you something about JSON first. It was created for humans, you know, like you and me.
When specified by [Mr. Douglas Crockford](https://en.wikipedia.org/wiki/Douglas_Crockford), it was supposed to be:

- easy to read & write
- easy for machines to parse & generate

All the more so we thought we can write blog posts in it & get on with our lives. Oh how naive we were!<br/>
Easy to read & write? - not if you write a complicated blog post with tables, diagrams and code blocks.

It wasn't obvious to us from the start though, so this is why we'll share this little story with you. Fortunately, you won't have to go through the same struggle as we did. Sit back, relax and take a deep breath. You're safe now.


## Our little static site that could
For various reasons, we had (and still have) an Angular site running on [fadeit.dk](https://fadeit.dk).
Without getting into details, our decision was to write a small Angular blog module that we'd feed JSON to.
We all do our fair share of console cowboying, so we weren't afraid of JSON. Doing so we got many advantages out of the box:

- design / UX consistency with the rest of the site
- complete control over rendering / plugins (i.e. code highlighting)
- decent performance
- some SEO 
- a JSON linter!
- i18n (which we didn't really use)

> Most of all, we saved time. Design aside, the actual implementation took a couple of days.

We were happy, so we wrote a few blog posts.
After a while, the writing frequency went down.
Then, we started to forget how the JSON should be structured.
We had to refresh our memory with each blogpost.
When we did want to go all in & create some crazy custom layout we'd add piles of code that we'd forget how to use later.
Let's face it, they weren't properly documented either. 
Why would they be? We just wanted to write a blog post, not document how elements are rendered.

To avoid going full mental jacket here, let's just say it became a burden to write each blog post. 
None of us were motivated enough to start on a new post, a mental block was in place and we slowly stopped writing anything.

> It wasn't strict enough in the right places, but too strict in the wrong places.
We couldn't just write a blog post anymore.


## Why use a static site generator anyway?

For us here's what made it attractive:

- progressive enhancement: no need for JavaScript
- better performance: easier to handle traffic spikes, no db queries, server-side rendering & the list goes on
- super easy to host & run, inexpensive too (you can even do Github pages!)
- version control for content
- it's safer, too!

We were happy users of a static site generator (kinda) in the first place, but that was mostly client-side.
While we did get an UX bonus, it was rather slow on mobile devices. That's partly because it required downloading large scripts at first load. We weren't taking advantage of server-side rendering. <br/>
It still made a lot of sense to switch to a proper static site generator - and so we did.


## Why Jekyll

There are plenty site generators to choose from (more on that later), but Jekyll caught our eye because of:

- fewer scripts, no Angular, fast first load
- markdown! Easy to write, easy to migrate
- community, maturity & (plugins)[https://jekyllrb.com/docs/plugins/]
- easy to get started
- [AMP](https://www.ampproject.org/) support

Granted, the first 2 items in the list are true for many static site generators.
However, the size and dedication of the community just blew our mind. 


## To wrap it up
Don't take our word for it, give it a try for your next project. We're happy now, but we were also happy with our Angular-JSON static site.
We're also running Wordpress, Joomla, Squarespace and even Medium blogs. There are many, many others you can try too. How many? About [437 of them](https://staticsitegenerators.net/) to be precise (at the time of this post). 

There's no silver bullet, but Jekyll is a damn good solution for techies to produce content.
