---
layout: post
title:  "CSS backdrop filters"
description: "The backdrop-filter will (hopefully) become mainstream in the near future. Here's why I am excited about it."
date: 2016-09-27 15:22:00+01:00
size: 12
sizeDesktop: 6
cover: "assets/backdrop-(filter)-it-like-its-hot/css-backdrop-filter-text-magic.gif"
textColor: blue-grey-800
metaColor: blue-grey-800
shadow: true
comments: true
author: dan
tags: [css, front-end, filter, backdrop-filter, tips-and-tricks]
---

There are many new CSS features to get excited about ([awesome list here](http://css4.rocks/)?).
Tons of selectors & structural properties that I can't wait to see implemented in all browsers. <br/>

<small>Side note: OMG CSS grid 😱, amirite? Did you even see this:</small>

<div style="width: 100%">
  <blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">This is a very basic type of layout CSS Grid will let you create without any additional markup <a href="https://t.co/HGOd0RGnQ1">pic.twitter.com/HGOd0RGnQ1</a></p>&mdash; Wes Bos (@wesbos) <a href="https://twitter.com/wesbos/status/777955345146777600">September 19, 2016</a></blockquote>
</div>

<blockquote><p lang="en" dir="ltr">Here is was the CSS behind that looks like: <a href="https://t.co/BNiJnZ7zaX">pic.twitter.com/BNiJnZ7zaX</a></p>&mdash; Wes Bos (@wesbos) <a href="https://twitter.com/wesbos/status/777957127558082561">September 19, 2016</a></blockquote>

However, I'm the kind of person that gets excited about the small things, `backdrop-filter` included. As humble as it seems, it can enable a ton of design stunts for those that can wield it.


## What you can do with backdrop-filter
For centuries, designers and developers have been trying to figure out how to adjust text color when placed on top of a background. If the background is light, the text has to be dark and vice-versa. <br/>
Similar to centering elements before flexbox, many gave up and looked into other professions. Others hacked their way out of it with cheap tricks, such as text-shadow / box-shadow (I'm looking at you, Facebook!).

It doesn't have to be that way. Here's what we can achieve with `backdrop-filter: inverse(1)`:

<span style="text-align: center; width: 100%; display: inline-block;">
![Backdrop filter trick][backdrop-image]
</span>

Contrary to other solutions, this one is independent from the text itself. It's also flexible IMO & it can be used for other use cases too.<br/>
There's a small trick to it. To apply the `inverse` filter only on the text and prevent messing with the background, the background can be inverted again (`filter` can be used for that). Here's what I mean:

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
Unless you are on **Safari 9+**, you'll see the inverted background (pink) and nothing else. <br/>
Browser support [is limited](http://caniuse.com/#feat=css-backdrop-filter) at the moment.

But that's not all! Like `filter`, we can use a bunch of effects, such as `brightness`, `contrast`, `grayscale`, `hue-rotate` (awesome!), `saturate` and more. <br/>
So many possibilities, so little time! What will you build?


## Why does it exist
But wait, we already have `filter`? What's the difference? <br/>
Simply put, `filter` applies a certain (duh) filter on an element. On the other hand, `backdrop-filter` applies the effect to the layers (elements) underneath it.

<small>Don't confuse setting opacity < 1 to an element that has a `filter` with applying that filter to the elements underneath it. It's completely different.</small>

The origin of `backdrop-filter` is tied to Apple's frosted glass design on iOS (and macOS). It's fairly popular these days, with many devs trying to reproduce it one way or another. In recent Webkit builds, creating the frosted glass effect is as easy as:

![Apple backdrop filter][apple-image]
If you want to read more about it there's plenty of info in [this blog post](https://webkit.org/blog/3632/introducing-backdrop-filters/).


## Why I don't care about the frost effect
It's just a trend. It's going to disappear when the next trend comes out... use it with caution (damn, I should remove it from [mindrudan.com](http://mindrudan.com)).


## Wrap up
Convincing browsers vendors to implement this experimental feature won't be easy.
As we all know, the best way to do it is with a good song. Here goes:

<i>
...<br/>
When the browser pimp's in the crib ma<br/>
(Back)drop it like it's hot<br/>
(Back)drop it like it's hot<br/>
I got the rolly on my DOM and I'm pouring Chandon<br/>
And I roll the best stylesheet cause I got it going on<br/>
...
</i>

<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

[apple-image]: {{ site.baseurl }}/assets/backdrop-(filter)-it-like-its-hot/css-backdrop-filter-apple.png
[backdrop-image]: {{ site.baseurl }}/assets/backdrop-(filter)-it-like-its-hot/css-backdrop-trick.gif