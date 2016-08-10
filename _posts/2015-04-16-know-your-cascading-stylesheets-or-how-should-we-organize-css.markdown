---
layout: post
title:  "Know your cascading stylesheets OR how should we organize CSS?"
description: "How can we create maintainable, reusable, comprehensible CSS? Is there a formula that works for everybody, or is it just a matter of personal preference?"
date:   2015-04-16 14:10:00+02:00
size: 12
sizeDesktop: 12
textColor: grey-50
metaColor: grey-600
comments: true
author: dan
tags: [CSS, stylesheet, pre-processor, SASS, LESS, BEM, front-end]
---

Here's my experience: every time I start a new project I feel like re-thinking my CSS strategy. I never felt satisfied with the quality of the stylesheets at the end of a project.

The main reason is that there are two types of CSS: the original pretty CSS and 'after a few months' CSS. Most of us start out ambitious, carefully writing properties and actually documenting the process. When fixes or features need to be added after a few weeks, we just throw properties wherever to get it done, because deadlines. 

## The problem with CSS

Documentation, formatting, frameworks, pre-processors, specificity - they all are trouble makers. Any of these might force you to give up and sell your soul. I am looking at you, the so called developer that appended `!important` to a property. In the W3C spec, "important" is documented as "you gave up and now a scrambled mess of selectors that nobody understands took control of your stylesheet".

In all seriousness now, you should never have to write it. However, there are many other signs that show you lost your grip. For example, if you have the same selector in many places and you aren't sure what exactly you're changing when updating a property.

I am not writing this article because I have it all figured it out, but because I have some clues on how to improve the quality of CSS. I don't want it to feel like a set of rules, but rather the start of a constructive dialog.

## Some pre-processor context

We need to get one thing straight. I am using a CSS pre-processors (LESS, have been using SASS too) and I can't go back. It makes my stylesheets more consistent and enables me to write less (duh) code. It also makes it easy to change global properties that (ironically) truly cascade on all elements. This article is not about pre-processors and they are most certainly NOT a solution to CSS organization issues. While they might be helpful, they bring their own problems. I will touch the subject here and there, because many front-enders rely on pre-processors to keep their sanity.

## The solutions today

### Properties

Let's start with the easy part.

When it comes to properties, what works and what doesn't?

I've been going the 'random way' until now and everything turned out ok. I wouldn't say no to another approach. Here are the most popular options:

1. Group by type
2. Order by line length
3. Alphabetize

#### 1. Group by type

This method was once (and maybe still is) the most popular method of writing properties. Nicolas Gallager's [idiomatic CSS][idiomatic-css] recommends this style for smaller teams, but suggests alphabetising for larger teams.

Here's one how to do it:


```css
section {
  /* Display & Box Model */
  display: inline;
  box-sizing: border-box;
  width: 15px;
  padding: 5px 10px 15px 0;
  border: 1px dashed #000;
  margin: 10px 20px;

  /* Color */
  background: #FFF;
  color: #333;

  /* Text */
  font-family: Helvetica, Arial, sans-serif;
  font-size: 12px;
  line-height: 16px;
  text-align: center;

  /* Other */
  cursor: pointer;
}
```

We are free to add our own types, such as `Effects` (can be border-radius, box-shadow, etc), `Animations & Transitions` or `Transforms`. There aren't any rules, we can create our own as long as we are consistent. The downside is that the rest of the team has to learn these rules, which is not very efficient.

#### 2. Order by line length

This is the 'old school' method. Some developers like it because it decreases the amount of lines and gives them a better overview of the stylesheet. I don't think this method is realistic today, especially when we have to pile up a bunch of duplicate properties to support all browsers.

Here's why it worked before:


```css
section { display: inline; overflow-x: hidden; box-sizing: border-box; width: 15px; height: 15px; }
section ul { background-color:red; }
```

...and here's why it doesn't work today:


```css
section ul li {  animation:animationFrames linear 3s; animation-iteration-count: infinite; animation-fill-mode: forwards; -webkit-animation: animationFrames linear 3s; -webkit-animation-iteration-count: infinite; -webkit-animation-fill-mode:forwards; -moz-animation: animationFrames linear 3s; -moz-animation-iteration-count: infinite; -moz-animation-fill-mode:forwards; -o-animation: animationFrames linear 3s; -o-animation-iteration-count: infinite; -o-animation-fill-mode:forwards; -ms-animation: animationFrames linear 3s; -ms-animation-iteration-count: infinite; -ms-animation-fill-mode:forwards; }
```

To be fair about it, I should point out that there is a way to make it better. By using a task runner (such as [Grunt][grunt] or [Gulp][gulp]) we can automatically prefix the properties that require a vendor prefix. This way we'll get rid of many of the duplicate styles:


```css
section ul li {  animation:animationFrames linear 3s; animation-iteration-count: infinite; animation-fill-mode: forwards; }
```

It's better, but probably 1-2 properties away from being hard to read. Some developers used to argue that it also decreases the size of the CSS file, which is now a 'commodity'. Again, by using a task runner we can eliminate all spaces and line breaks. This gives us the freedom of writing CSS however we like, with plenty of spaces and comments. ([grunt-cssmin][grunt-cssmin] and [gulp-cssmin][gulp-cssmin])

I wonder if anybody is still following the order by line approach today. I wouldn't be surprised if today it's no more than a page in the history of web development.

By the way, what's the deal with the autoprefixer? It's very helpful. If you are not using one, head over to [grunt autoprefixer][grunt-autoprefixer] or [gulp autoprefixer][gulp-autoprefixer] and expect to get your mind blown. You'll drastically reduce the size of your CSS source and increase it's readability. If you are using a pre-processors you will also get rid of a huge chunk of mixins.

#### 3. Alphabetize

The last but not the least method is alphabetising properties. Many developers like the consistency that it brings, that's why it works well for bigger teams. Any other method you might choose is often adjusted to a developer's mindset, but this one always stays the same. However, CSS properties have a strong relationship - for example absolute positioning with z-index, left or top. It's hard to visualise these relationships when alphabetising.

There's more criticism: we certainly don't write CSS alphabetically, therefore developers have to invest extra time to alphabetize declarations after they wrote them. What about for those using a pre-processor, how will we alphabetize mixins?

Should we actually follow any rules while writing properties? In all honesty, I never felt the need to add rules, random works just fine. At the same time, I haven't worked in a large team of front-enders. Maybe there is a point to alphabetize, but it certainly doesn't feel wrong to randomly write the properties for me.

#### Shorthand (bonus)

To wrap up the properties, here's another interesting topic. Should we use shorthand or write it all down? The answer is: it depends. If you are talking about box-model properties, it sometimes makes sense to use shorthand. I use it in the following circumstances:


```css
section{
   /* I use shorthand for the following cases */
   margin: 15px 20px;
   margin: 10px 15px 20px 35px;
   margin: 30px auto;
}
```

The shorthand can force values to top, right, bottom and left. Sometimes we don't want all properties to be set. This can result in all kinds of conflicts, for example:


```css
section {
   margin-top: 20px;
}

@media only screen and (min-width:767px){
   section{
     margin: 0 20px 0 15px;
   }
}
```

In this case, I actually want to maintain the original margin-top, but the shorthand forces me to either repeat myself or even worse - set it to a different value without realizing. In this case I prefer not to use shorthand:


```css
section {
   margin-top: 20px;
}

@media only screen and (min-width:767px){
  section{
    margin-right: 20px;
    margin-left: 20px;
  }
}
```

For some properties it makes sense to generally use shorthand, like border-radius. There is no simple answer to 'should I use shorthand'. My opinion is we have to think about each property and decide on the spot. I might start with shorthand and change it later when I notice some redundant properties.

### Splitting CSS info files

This is were the 'pro' territory begins. It's not because it's hard to do (it used to be harder), but you have to know what you're doing. You need to have a clear idea of what CSS you'll write and how it's going to be connected. Call it 'CSS architecture' if you will.

One of the common ways to get started is to use the SMACSS principles. The idea is to split the rules into 5 categories:

- Base
- Layout
- Module
- State
- Theme


Of course, there's more to SMACSS than this - but I won't get into details. This approach will introduce a sort of modularity to CSS, which of course is what every developer dreams of. It's the holy grail: flexibility, maintainability, extensibility, comprehensibility - it has it all. Sounds good, let's all do it! Only...  I don't think it really works with CSS. It could work sure, but do you know any CSS architects which can design a modular stylesheet system? I believe that most modular systems work well because of the pre-analysis and thorough planning invested into them initially. I would like to think the same is possible with CSS, but do we ever do it?

I'll leave this one open. However, I have to say that this kind of structure has never worked for me. My team members weren't happy either. It's hard to find declarations and most importantly: it doesn't align with the cascade.

### The cascade / specificity

Oh boy, this is a big one.<br/>
Let's get the basics right first and then you'll see why specificity can be a bitch.<br/>
In a nutshell, CSS properties are applied based on the following model:

`inline style > ID > class / attributes / pseudo-class > element / pseudo-element`

Simply put, here's how CSS specificity is calculated:


```css
 *              {} /* -> specificity = 0,0,0,0 */
 p              {} /* -> specificity = 0,0,0,1 */
 p:first-child  {} /* -> specificity = 0,0,1,1 */
 p:first-letter {} /* -> specificity = 0,0,0,2 */
 p span         {} /* -> specificity = 0,0,0,2 */
 p span+i       {} /* -> specificity = 0,0,0,3 */
 p span i.red   {} /* -> specificity = 0,0,1,3 */
 p.red.active   {} /* -> specificity = 0,0,2,1 */
 #name          {} /* -> specificity = 0,1,0,0 */
 style=""          /* -> specificity = 1,0,0,0 */
```

By the way, if you understand this, you should never need to use important. Even if you completely mess up, you can still use tricks to avoid it, as shown in this example:


```css
/* Avoid */
p.red { color: red!important; }
p.red { color: blue; }

/* If you really have to, use a trick */
p.red.red { color: red; } /* -> specificity = 0,0,2,1 */
p.red { color: blue; }    /* -> specificity = 0,0,1,1 */

/* the color of p.red will be red in both cases */
```

Finally, let's focus on two of the declarations above.


```css
p:first-child  {}  /* -> specificity = 0,0,0,2 */
p span         {}  /* -> specificity = 0,0,0,2 */
```

The specificity is the same for both declarations. In this case, the selector that comes last in the file wins ('p span'). You can imagine how splitting CSS into many files can be problematic.

That's the least of the problems though. Specificity can single-handedly ruin CSS for us. It's a frequent problem with frameworks, content management systems, style guides and of course pre-processors. The real issue is that when CSS is specific, it forces us to keep writing more and more specific selectors. We then easily end up with redundant, complicated and disorganized stylesheets.

> Rule number one while writing CSS: be specific only when truly necessary.

### Element tree indentation

Specificity used to be a common problem with pre-processor users. The advertisement for both SASS and LESS that got a lot of developers to use them was: 'reflect your HTML (or DOM) structure in your stylesheet'.

Let's see what that means. For the follwing HTML: 


```html
<header>
  <section class=“navigation”>
    <a href=“home” class=“logo”>...</a>
  </section>
</header>
```

Instead of this CSS:


```css
header{
  background-color: blue;
}

header .navigation {
  width: 300px;
}

header .navigation .logo {
  max-height: 20px;
}
```

We could write the following LESS/SASS code:


```sass
header{
  background-color: blue;

 .navigation {
    width: 300px;

    .logo {
      max-height: 20px;
    }
  }
}
```

If that doesn't impress you, it should. It helps us easier implement something called OOCSS - you guessed it - object oriented CSS. It's not object oriented in a true sense, but it encourages the development of components that are independent, self-contained and highly reusable (there's also more to OOCSS, but I again won't go into details).

The downside of this is being specific all the time, which we should avoid. Be specific only when truly necessary, remember?<br/>
There's a way around this too.

### Block, element, modifier

BEM is a naming convention that makes css selectors more informative and adds a certain amount of strictness. The pattern it introduces is the following:


```css
.navigation {} /* Block */
.navigation__logo {} /* Element */
.navigation--collapsed {} /* Modifier */
```

Specificity is now achieved by the naming conventions. What it lacks is the nesting of pre-processors, but nothing stops us from writing it like this:


```css
.navigation {}
  .navigation__logo {}
     .navigation__logo__image {}
  .navigation--collapsed {}
```

Keep in mind that there are several variations of BEM conventions, but the core concept is the same for all of them. For some time now pre-processors introduced BEM support, which enables us to simulate the DOM structure & use proper nesting - without having specificity troubles. Both LESS and SASS support a form of the following:


```sass
.navigation {
  &__logo {
    &__image {}
  }
  &--collapsed {}
}
```

That's neat. As long as we don't mind the 'ugly' HTML that comes out of it, it's a very reasonable solution. The 'ugly' problem is discussed in [this article][bem-syntax] and I think the author has some good points. We shouldn't be dismissing this concept just because it looks odd.

## My conclusion so far

There's more to cover, but I feel it was too optimistic to do it all in 1 article. A particularly interesting subject are design style guides, which I plan to write about in a future post. Don't forget: write meaningful names for your selectors and pay attention to your HTML structure, many CSS problems start there.

What else do you use? Do you write a table of contents? Do you have global classes like '.hidden' or '.disabled'? How do you organize media queries? What about tools like [colorguard][colorguard], [PhantomCSS][phantom] or [parker][parker], can they help us organize our CSS better?

There's more. Self contained components à la React or Polymer. Are they the future?

Those are many interesting questions, but you scrolled this far to see a conclusion. Here it is: try out different methods until you and your team members are satisfied. There are many ways to organize stylesheets, but none of them are going to solve all of our problems. Me? I can't go back from LESS, autoprefixer, and cssmin. I try to write my stylesheets in a OOCSS-way, which sometimes works. Lastly, I think BEM is neat and will definitely push it more in my future projects.

[idiomatic-css]: https://github.com/necolas/idiomatic-css
[grunt]: http://gruntjs.com/
[gulp]: http://gulpjs.com/
[grunt-cssmin]: https://github.com/gruntjs/grunt-contrib-cssmin
[gulp-cssmin]: https://github.com/chilijung/gulp-cssmin
[grunt-autoprefixer]: https://github.com/nDmitry/grunt-autoprefixer
[gulp-autoprefixer]: https://github.com/sindresorhus/gulp-autoprefixer
[bem-syntax]: http://csswizardry.com/2013/01/mindbemding-getting-your-head-round-bem-syntax/
[colorguard]: https://github.com/SlexAxton/css-colorguard
[phantom]: https://github.com/Huddle/PhantomCSS
[parker]: https://github.com/katiefenn/parker
