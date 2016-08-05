---
layout: post
title:  "Getting started with Vue.js: AngularJS perspective"
description: "Writing Vue.js applications after working with AngularJS will feel familiar. Vue won't pack as many goodies out of the box, but it will still give Angular a run for its money."
date:   2015-05-14 15:14:03+02:00
size: 12
sizeDesktop: 12
textColor: grey-50
metaColor: grey-600
comments: true
author: dan
---

## Background

First and foremost, what is Vue.js and why should you use it?

Vue is a library for developing interactive web interfaces. Its API is inspired by Angular & Backbone, but you can read more about it in [this guide][vue-guide]. What's more interesting is <u>why use it</u>, especially when you are familiar with Angular. First of all, it's very flexible. It allows mixing and matching all the libraries you love. That means you can create your own little front-end stack. Secondly, it's more snappy because it's simpler and doesn't use dirty checking to observe changes. On top of that, learning Vue doesn't take much time. This will make it easy to bring in new team members.

You can read even more about it in Vue's [FAQ page][vue-faq], where Vue is compared to Angular, React and more.

<small>**Updated 15 May 2015**</small>

I jumped on the AngularJS train more than 1 year ago and had lots of fun working with it. From August to January 2015 I had the chance to research Angular application structures. The study was in relation to performance, but it touched on many other subjects, such as component reusability and learnability.

I am new to Vue.js, therefore this article will be about my first encounter with the library. Later on I will try to make a more comprehensive analysis.

> Angular <b>1.\*</b> and Vue.js <b>0.11</b> are considered in the article.
> Breaking changes might come in [Vue 0.12][vue-158]. As for Angular, version 2 will be a completely different beast.

## Vue.js and AngularJS

I felt comfortable with Vue within a few days. It might be because it resembles other frameworks. It might be because of its simplicity, I couldn’t tell. To better portray the similarities & differences between the two, I devised a brief comparison of some of the high-level concepts in Angular and Vue. Here's what I ended up with:

{'rows': {'1': {'columns': {'1': {'content': {'1': {'file': 'table-directive-vue.js.txt', 'language': 'javascript', 'type': 'code'}, '0': {'text': 'Directives', 'type': 'title'}}, 'size': '8'}, '0': {'content': {'1': {'file': 'table-directive.js.txt', 'language': 'javascript', 'type': 'code'}, '0': {'text': 'Directives', 'type': 'title'}}, 'size': '8'}, '2': {'content': {'1': {'text': 'Directives are simpler in Vue; they seem to be more focused. In Angular a directive can be many things, better resembling a ‘Component’ in Vue.', 'type': 'p'}, '0': {'text': 'Notes', 'type': 'title'}}, 'size': '8'}}}, '0': {'columns': {'1': {'content': {'1': {'file': 'table-module-vue.js.txt', 'language': 'javascript', 'type': 'code'}, '0': {'text': 'Components (module equivalent in Vue)', 'type': 'title'}}, 'size': '8'}, '0': {'content': {'1': {'file': 'table-module.js.txt', 'language': 'javascript', 'type': 'code'}, '0': {'text': 'Angular Modules', 'type': 'title'}}, 'size': '8'}, '2': {'content': {'1': {'text': 'Modules are a wrapper in Angular. In Vue they will hold most component logic. <br/><br/>You can see much thought was put into Vue by noticing the small details. For example, one of the component options is ‘name’, a property that customizes the component name (will be displayed in the console, great help for debugging)', 'type': 'p'}, '0': {'text': 'Notes', 'type': 'title'}}, 'size': '8'}}}, '2': {'columns': {'1': {'content': {'1': {'file': 'table-filter-vue.js.txt', 'language': 'javascript', 'type': 'code'}, '0': {'text': 'Filters', 'type': 'title'}}, 'size': '8'}, '0': {'content': {'1': {'file': 'table-filter.js.txt', 'language': 'javascript', 'type': 'code'}, '0': {'text': 'Filters', 'type': 'title'}}, 'size': '8'}, '2': {'content': {'1': {'text': "Filters aren’t much different, although Vue provides read/write options. (see <a href='http://vuejs.org/guide/custom-filter.html' target='_blank'>this guide</a>)", 'type': 'p'}, '0': {'text': 'Notes', 'type': 'title'}}, 'size': '8'}}}}, 'type': 'fake-table', 'header': {'1': {'text': 'Vue.js', 'size': '8'}, '0': {'text': 'AngularJS', 'size': '8'}, '2': {'text': 'Notes', 'size': '8'}}}

Because Vue will only help us manage the VM, it won’t tackle some of the challenges that Angular covers out of the box. Services / HTTP reqs, routing, promises (just to name a few) are not among Vue’s concerns. That’s good and bad. Perhaps "bad" is an exaggeration; but it’s a negative aspect for someone that got used to Angular and didn't have to lift a finger for it. In other words, we have to mix and match libraries as we go to cover our needs. That can be quite a task, but some developers love having this kind of control (others think it’s a waste of time). However, once you find a few good libraries, you won’t have to do it again. I think this kind of flexibility is very empowering. Imagine building your own smaller, focused framework (like Angular) on top of a good foundation: Vue.

Creating a personalized front-end stack is great fun too. The first stack I ended up with was made out of: Director (routing), Reqwest (you guessed it), Q (promises) and of course Vue. The best name I can make out of these is QRVD (kinda sounds like curved). For the rest of the article you could have this stack in mind, although it won’t change much if you don’t.

## Templating

For the same reasons that’s easy to write templates in Angular, it’s really easy to write them in Vue. In fact, it’s a bit of a struggle to find differences between the two.

Here’s an overview:

{'rows': {'4': {'columns': {'1': {'content': {'1': {'file': 'table2-vue-conditional-classes.html.txt', 'language': 'markup', 'type': 'code'}, '0': {'text': 'Conditional classes', 'type': 'title'}}, 'size': '8'}, '0': {'content': {'1': {'file': 'table2-conditional-classes.html.txt', 'language': 'markup', 'type': 'code'}, '0': {'text': 'Conditional classes', 'type': 'title'}}, 'size': '8'}, '2': {'content': {'1': {'text': 'Another example that shows similarities. Both also support <em>v-attr/ng-attr</em>.', 'type': 'p'}, '0': {'text': 'Notes', 'type': 'title'}}, 'size': '8'}}}, '1': {'columns': {'1': {'content': {'1': {'file': 'table2-vue-model-binding.html.txt', 'language': 'markup', 'type': 'code'}, '0': {'text': 'Model binding', 'type': 'title'}}, 'size': '8'}, '0': {'content': {'1': {'file': 'table2-model-binding.html.txt', 'language': 'markup', 'type': 'code'}, '0': {'text': 'Model binding', 'type': 'title'}}, 'size': '8'}}}, '5': {'columns': {'1': {'content': {'1': {'file': 'table2-vue-dom-events.html.txt', 'language': 'markup', 'type': 'code'}, '0': {'text': 'Event binding', 'type': 'title'}}, 'size': '8'}, '0': {'content': {'1': {'file': 'table2-dom-events.html.txt', 'language': 'markup', 'type': 'code'}, '0': {'text': 'Event binding', 'type': 'title'}}, 'size': '8'}, '2': {'content': {'1': {'text': "The generic v-on directive makes things more consistent, but I guess it's down to personal preference.", 'type': 'p'}, '0': {'text': 'Notes', 'type': 'title'}}, 'size': '8'}}}, '3': {'columns': {'1': {'content': {'1': {'file': 'table2-vue-conditionals.html.txt', 'language': 'markup', 'type': 'code'}, '0': {'text': 'Conditionals', 'type': 'title'}}, 'size': '8'}, '0': {'content': {'1': {'file': 'table2-conditionals.html.txt', 'language': 'markup', 'type': 'code'}, '0': {'text': 'Conditionals', 'type': 'title'}}, 'size': '8'}}}, '0': {'columns': {'1': {'content': {'1': {'file': 'table2-vue-interpolations.html.txt', 'language': 'markup', 'type': 'code'}, '0': {'text': 'Interpolation', 'type': 'title'}}, 'size': '8'}, '0': {'content': {'1': {'file': 'table2-interpolations.html.txt', 'language': 'markup', 'type': 'code'}, '0': {'text': 'Interpolation', 'type': 'title'}}, 'size': '8'}, '2': {'content': {'1': {'text': 'Unfortunately, if we try to output and object or array, Vue won’t be able to render it (we’ll see <em>[Object]</em>). I always found that useful in Angular. <br/><br/> <b>Update:</b> you can do the same in Vue with the built-in json filter. <em>{{someObject | json}}</em>', 'type': 'p'}, '0': {'text': 'Notes', 'type': 'title'}}, 'size': '8'}}}, '2': {'columns': {'1': {'content': {'1': {'file': 'table2-vue-loops.html.txt', 'language': 'markup', 'type': 'code'}, '0': {'text': 'Loops', 'type': 'title'}}, 'size': '8'}, '0': {'content': {'1': {'file': 'table2-loops.html.txt', 'language': 'markup', 'type': 'code'}, '0': {'text': 'Loops', 'type': 'title'}}, 'size': '8'}, '2': {'content': {'1': {'text': "Both support model options (ex. debounce for a loop filter). BTW, we can also asign repeated objects in Vue <br/><br/> <em>v-repeat='item: items'</em>", 'type': 'p'}, '0': {'text': 'Notes', 'type': 'title'}}, 'size': '8'}}}}, 'type': 'fake-table', 'header': {'1': {'text': 'Vue.js', 'size': '8'}, '0': {'text': 'AngularJS', 'size': '8'}, '2': {'text': 'Notes', 'size': '8'}}}

The list goes on. There’s even a v-cloak directive that hides the bindings before they are rendered (as ng-cloak does).

However, there’s one directive that you won’t find in Angular: `v-with`. Attaching this directive and passing a data object will allow a child VM to inherit data from his parent. In my first encounter, I though of it as a drastically simplified Angular Controller, managing the data flow across the application (without the need of an equivalent to `$scope`. By default, components will have an isolated scope. That makes this directive essential, although components also accept <em>inherit</em> as a property (see [more on scope inheritance][vue-scope-inheritance]).

## Modularity / Application structures

As I mentioned, I researched this area in Angular. My conclusion was that Angular both nailed it and it didn’t. You can create great modular architectures that are indeed maintainable, reusable, comprehensible, etc. However, you’ll need a monster-build strategy to go with it, or perhaps use tools like [Require.js][requirejs] or [Browserify][browserify] to ease the pain. That's not even the worst part. If you are just starting with Angular, you probably won’t come across a good structuring guide, which is a shame. It’s only later (or maybe too late) that Angular developers decide to look into it.

So how does Vue.js perform? More or less the same. There's a very short section in the [Vue guide][vue-app-guide] about app structuring, but it's not enough. I believe developers are able to produce kick-ass modular architectures with Vue, but there are no comprehensive guides or suggestions on how to do it. I wasn't satisfied, so in the ‘Building larger applications’ section I tried to sketch out an example that may help developers new to Vue.

## Components

Before that, I have to add a comment on components. I think, Vue managed to clearly separate components, which we can’t really say about Angular. It’s perhaps closer to Angular 2, which is great news in my mind. <br/> As a result, there's no need for a `$scope` equivalent.

## Scopes / Data flow

I truly believe Vue is elegant when it comes to Data flow & control. It felt natural and there was no need for an emit/broadcast mechanism to communicate with parent components (although Vue supports such events). There is a downside to it: the components and views become even more coupled (but hey, it is called a VM). Sometimes it's tricky to understand which data is inherited from which `v-with`, so use it sparingly. There are ways to prevent spaghetti code from creeping in. Services are one good method, but there’s not much documentation about them.

## Services

*My source is written in CoffeeScript, so I’ll provide examples in both Coffee and Vanilla JS format.*

One approach for writing services is to use a `HTTP` -> `Service` -> `Component` mechanism. HTTP has to be generic, so we can extend it for each object:

{'header': {'1': {'text': 'JavaScript', 'size': '12'}, '0': {'text': 'CoffeeScript', 'size': '12'}}, 'rows': {'0': {'columns': {'1': {'content': {'0': {'file': 'structure1.js.txt', 'language': 'javascript', 'type': 'code'}}, 'size': '12'}, '0': {'content': {'0': {'file': 'structure1.coffee.txt', 'language': 'coffeescript', 'type': 'code'}}, 'size': '12'}}}}, 'type': 'fake-table', 'largeCode': True}

Just for reference, a patch request using reqwest looks like this:


```javascript
reqwest({
  url: url,
  type: 'json',
  method: 'patch',
  data: JSON.stringify(data),
  contentType: 'application/json'
});
```

With CoffeeScript we can extend the HTTP class to adapt it for other use cases. For example, the User object could look like: <br/> (Remember: Angular already has the $http service, so we don't have to do any of these bits)

{'header': {'1': {'text': 'JavaScript', 'size': '12'}, '0': {'text': 'CoffeeScript', 'size': '12'}}, 'rows': {'0': {'columns': {'1': {'content': {'0': {'file': 'user-http.js.txt', 'language': 'javascript', 'type': 'code'}}, 'size': '12'}, '0': {'content': {'0': {'file': 'user-http.coffee.txt', 'language': 'coffeescript', 'type': 'code'}}, 'size': '12'}}}}, 'type': 'fake-table', 'largeCode': True}

Then come the actual services, which are perhaps closer to what we’re used to in Angular. For the same Object, we need to pass UserHTTP as a constructor param and then we’ll be able to call its methods and process the data as needed (before or after requests are sent).

{'header': {'1': {'text': 'JavaScript', 'size': '12'}, '0': {'text': 'CoffeeScript', 'size': '12'}}, 'rows': {'0': {'columns': {'1': {'content': {'0': {'file': 'user-service.js.txt', 'language': 'javascript', 'type': 'code'}}, 'size': '12'}, '0': {'content': {'0': {'file': 'user-service.coffee.txt', 'language': 'coffeescript', 'type': 'code'}}, 'size': '12'}}}}, 'type': 'fake-table', 'largeCode': True}

Finally, in any component we can instantiate a service and call the methods that in turn will make an HTTP request.


```javascript
...
HTTP         = new HTTP();
userService  = new UserService(HTTP.users);
...


// Inside a Vue component
...
ready: function(){
  vm = this;

  userService.all()
      .then(function(users){
        vm.$set('users', users);
      });
   ...
```

The next section contains a diagram that illustrates how this all fits together.

## Building larger applications

First of all, what is a large app? Well, potentially any app can become large. When it’s hard to understand how things are working and the structure is not comprehensible for outsiders, then you’re probably dealing with a large app.

In my case, after about a week of working on a Vue application the structure was almost chaotic. It wasn't easy to tell how it all fits together. On one hand, components weren't properly separated, but this doesn’t have anything to do with Vue. So it was time for taking a step back, analyzing and refactoring. I sketched out my components and tried to create a structure that would make things better.

Before you check out the diagram below, it is worth mentioning that Browserify was used to split components up. As I said before, you can either follow this approach or create a build strategy with Gulp/Grunt that concatenates the files. <br/><br/> Get a full copy of the diagram [here][diagram-img].

![Diagram: one way to structure larger Vue.js apps.][diagram-img]

In terms of file/directory structure, here’s how it looked:

(get a full copy [here][dir-img])

![Possible directory structure for a larger Vue.js app.][dir-img]

## Final words

I hope these sketches will be a good starting point/source of inspiration for devs that are starting out.

In future posts I’ll probably write more about testing, maintainability and scalability, if I discover something interesting.

My final words are: try out Vue, you might find it useful for many of your projects. It’s a nice little library with as many super powers as Angular!


[browserify]: http://browserify.org/
[diagram-img]: {{ site.baseurl }}/assets/getting-started-with-vuejs-angularjs-perspective/vue-large-app-structure-diagram.svg
[dir-img]: {{ site.baseurl }}/assets/getting-started-with-vuejs-angularjs-perspective/vue-large-app-directory-structure.svg
[requirejs]: http://requirejs.org/
[vue-158]: https://github.com/vuejs/Discussion/issues/158
[vue-app-guide]: http://vuejs.org/guide/application.html
[vue-faq]: http://vuejs.org/guide/faq.html
[vue-guide]: http://vuejs.org/guide/
[vue-scope-inheritance]: http://vuejs.org/guide/components.html#Scope_Inheritance
