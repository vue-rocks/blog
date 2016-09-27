---
layout: post
title:  "Backdrop filter"
description: "The backdrop-filter will (hopefully) become mainstream in the near future. Here's why I am excited about it."
date: 2016-08-25 15:10:30+01:00
size: 12
sizeDesktop: 4
cover: "assets/backdrop-(filter)-it-like-its-hot/css-backdrop-filter-text-magic.gif"
textColor: blue-grey-800
metaColor: blue-grey-800
shadow: true
comments: true
author: dan
tags: [css, front-end, filter, backdrop-filter, tips-and-tricks]
---

There are many new CSS features to get excited about (have you seen [this crazy list](http://css4.rocks/)?).
Tons of selectors & structural properties that I can't wait to see in all browsers. 
Have you see what you can do with CSS grid?!

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">This is a very basic type of layout CSS Grid will let you create without any additional markup <a href="https://t.co/HGOd0RGnQ1">pic.twitter.com/HGOd0RGnQ1</a></p>&mdash; Wes Bos (@wesbos) <a href="https://twitter.com/wesbos/status/777955345146777600">September 19, 2016</a></blockquote>

<blockquote><p lang="en" dir="ltr">Here is was the CSS behind that looks like: <a href="https://t.co/BNiJnZ7zaX">pic.twitter.com/BNiJnZ7zaX</a></p>&mdash; Wes Bos (@wesbos) <a href="https://twitter.com/wesbos/status/777957127558082561">September 19, 2016</a></blockquote>

However, I'm the kind of person that gets excited about the small things.
Meet `backdrop-filter`, a nifty little property that can enable a ton of great design stunts.


## Why does it exist
But wait, don't we already have `filter`? Which is kind of mainstream by now? What's the difference?<br/>
Simply put, `filter` applies a certain (duh) filter on an element. On the other hand, `backdrop-filter` applies the effect to the layers (element) underneath it too.
You can use opacity w/ `filter` to apply an effect on layers placed underneath, but it's completely different.

`backdrop-filter` is something that Apple pushed because of their frosted glass design on iOS (and macOS). It's fairly popular these days, with many devs trying to reproduce it one way or another. In recent webkit builds, it's as easy as:

![Apple backdrop filter][apple-image]
If you want to read more about it there's plenty of info in [this blog post](https://webkit.org/blog/3632/introducing-backdrop-filters/).


## Why I don't care about the frost effect
It's just a trend. It's going to disappear when the next trend comes out. Use it with caution... (damn, I should remove it from [mindrudan.com](http://mindrudan.com)).


## What I care about
For centuries, designers and developers have been looking to figure out how to adjust text color when placed on top of a background. If the background is light, the text has to be dark and vice-versa. 
Many gave up and started farming (like with centering text), while some hacked their way out of it with cheap tricks, such as text-shadow / box-shadow (I'm looking at you, Facebook!).

It doesn't have to be that way, so here's what we can achieve `backdrop-filter: inverse(1)`:

<span style="text-align: center; width: 100%; display: inline-block;">
![Backdrop filter trick][backdrop-image]
</span>

This solution has the power to fix the problem, but in a very flexible way. <br/>
There's a trick to it though. To prevent messing with the background & apply the `inverse` filter only on the text, we can invert the background again (in this case, `filter` can be used for that). Here's what I mean:


```css
/* has to have z-index > 'before' & < 'after' */
p { /* ... */ }

/* the background, has to have z-index < 'p' */
section:before{
  background-color: green;
  filter: invert(1);
  /* ... */
}

/* has to have z-index > 'p' */
section:after{
  backdrop-filter: invert(1);
  /* ... */
}
```

The full code for it is available below.

<div style="width: 100%;">
  <p data-height="265" data-theme-id="0" data-slug-hash="XjRoZz" data-default-tab="css,result" data-user="danmindru" data-embed-version="2" class="codepen">See the Pen <a href="https://codepen.io/danmindru/pen/XjRoZz/">CSS filter to invert text color</a> by Dan Mindru (<a href="http://codepen.io/danmindru">@danmindru</a>) on <a href="http://codepen.io">CodePen</a>.</p>
</div>

<br/>
**Be warned**, unless you are on Safari 9+ you'll see the inverted background (pink) and nothing else. <br/>
Browser support is [still limited](http://caniuse.com/#feat=css-backdrop-filter).

But that's not all! Like `filter`, we can use a bunch of effects, such as `brightness`, `contrast`, `grayscale`, `hue-rotate` (awesome!), `saturate` and more.
So many possibilities, so little time! What will you build?


## Wrap up
Convincing browsers vendors to implement this experimental feature won't be easy.
As we all know, the best way to do it is with a good song. Here goes:

...<br/>
When the browser pimp's in the crib ma<br/>
Backdrop it like it's hot<br/>
Backdrop it like it's hot<br/>
I got the rolly on my DOM and I'm pouring Chandon<br/>
And I roll the best stylesheet cause I got it going on<br/>
...

<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

[apple-image]: {{ site.baseurl }}/assets/backdrop-(filter)-it-like-its-hot/css-backdrop-filter-apple.png
[backdrop-image]: {{ site.baseurl }}/assets/backdrop-(filter)-it-like-its-hot/css-backdrop-trick.gif