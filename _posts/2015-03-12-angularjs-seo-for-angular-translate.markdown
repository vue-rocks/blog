---
layout: post
title:  "AngularJS SEO friendly translations"
description: "SEO is a weak spot for AngularJS to begin with, but the issue takes a different turn when angular-translate is thrown into the mix."
date:   2015-03-12 17:30:16 +0200
size: 12
sizeDesktop: 4
textColor: grey-50
metaColor: grey-600
comments: true
author: sander
---

**Update**: I have written a follow-up post solving the same problem using ui-router [here][update-post]

## Introduction

Instant translations, this is what [angular-translate][angular-translate] brings to the table. While it is significantly more convenient than waiting one more site load cycle for a different language to load (especially on sites that are slow to begin with), it doesn't come without complications. If each translation language does not have a unique URL, search engines can not link search results to it, let alone crawl the site properly.

Here's what Google has to say on the matter:

> **Make sure each language version is easily discoverable**
> *Keep the content for each language on separate URL's. Don’t use cookies to show translated versions of the page. Consider cross-linking each language version of a page.*
> [https://support.google.com/webmasters/answer/182192][google-webmasters]

## Options

Okay, perhaps it's a nice practice to have language in the URL. Here are the options:

1. Store the language in a GET parameter e.g: **example.com/about?lang=en**
2. Store the language in a path e.g: **example.com/en/about**
3. Store the language in a sub-domain e.g: **en.example.com/about**

While the first option would be the easiest to implement, it is also the ugliest. The other two are relatively similar in terms of appearance and it boils down to your personal preference. An important distincion to make is that if language is stored in a sub-domain, it excludes the possibility of handling the routing with angular as subdomain is out of application router scope. We decided to explore the second option - having the language in a URL path.

The preferred approach would be to introduce a path variable on a router base path and have all children be relative to it or escape the language before routing is performed. While it is possible, it seems that the only way to achieve it is to nest all states in the following manner:

```javascript
angular.module('app').config([
  '$stateProvider', function ($stateProvider) {
    $stateProvider.state('app', {
      abstract: true,
      url: '/{lang:(?:da|en)}',
      template: '<ui-view/>',
      controller: 'RootController'
    });
    $stateProvider.state('app.root', {
      url: '',
      templateUrl: 'views/home.html',
    });
    $stateProvider.state('app.about', {
      url: '',
      templateUrl: 'views/about.html',
    });
  }
]); 
```

It is likely that by the time SEO becomes a priority, the application is already built and refactoring code does not sound appealing. To name a few, using ui-router has following downsides:

- Language slug becomes mandatory for each state, so example.com/about would become example.com/da/about
- All child state names need to be changed
- All links have to be refactored

We'll be taking a closer look at ui-router in a separate post soon, while this article focuses on finding an alternative configuration-based approach.

## Automation to the rescue!

Hosting Angular application is just serving static files, where each URL maps to a directory/file by default. Therefore, if we are to host the same application in a different directory for each language, we get SEO friendly URLs without a major overhaul of the code. If you are working on a big Angular project, the chances are that either Grunt or Gulp is in the mix. Grunt is our weapon of choice, so we need to add an extra copy task at the end of the build process that would copy all files into a subdirectory:

```javascript
copy: {
  build_en_lang: {
    files: [
      {
        src: [ '**', '!en/**'],
        dest: '<%= build_dir %>/en',
        cwd: '<%= build_dir %>',
        expand: true
      }
    ]
  }
}
```

`!en/**` tells Grunt not to copy itself every time that task is called.

The directory structure should be following:

- /path/to/app/index.html (will default to Danish)
- /path/to/app/other/files
- /path/to/app/en/index.html
- /path/to/app/en/other/files

Now we need to tell Angular to check for URL when determining language:

```javascript
var absUrl = $location.absUrl();
if(absUrl.indexOf('/en/') !== -1){
  $scope.activeLang = 'en-us';
}
else{
  $scope.activeLang = 'da-dk';
}
$translate.use($scope.activeLang);
```

`activeLang` is stored on scope because it will be later used to determine the class of language switching button

Google also suggests cross-linking each language version of the page. If user navigates to a different page, we should also update the link that points to the same page in other languages. For that we use $stateChangeSuccess event:

```javascript
//Construct url base
var port = $location.port();
//Port can be omitted on 80 or 443
port = (port === 80 || port === 443) ? '' : ':' + port;
var urlBase = $location.protocol() + '://' + $location.host() + port;

$rootScope.$on('$stateChangeSuccess',function(){
  $scope.enUrl =  urlBase + '/en' + $location.url();
  $scope.daUrl =  urlBase + $location.url();
});
```

The URLs need to be absolute, otherwise if this code was executed at example.com/en, it would calculate the 'en' URL to be at example.com/en/en and so on.

Next we need to display our cross-links in the template using `ng-href` directive:

```html
<a ng-href="{{enUrl}}" target="_self" ng-class="{'active': activeLang === 'en-us'}">
  <img src="assets/images/gb.svg" alt="Change language to English"/>
</a>
<a ng-href="{{daUrl}}" target="_self" ng-class="{'active': activeLang === 'da-dk'}">
  <img src="assets/images/dk.svg" alt="Change language to Danish"/>
</a>
```

`target='_self'` is used to tell Angular not to load this link in ajax but to navigate to it the Angular way. We need the browser to fetch appropriate index.html page and bootstrap the application (since we have one application per language).

The application is now SEO friendly, but we are not quite done yet. We are also using HTML5 mode to have pretty URLs. When using HTML5 mode, we must define base tag in the index file so Angular knows where the application path starts from:

```html
<!DOCTYPE html>
  <html lang="da" ng-app="example" ng-controller="AppCtrl">
    <head>
    <base href="/">
    <link rel="alternate" hreflang="en" href="http://example.com/en"/>
    <link rel="alternate" hreflang="da" href="http://example.com"/>
    ...
```

In case `hreflang` caught your eye, you can read more about it [here][google-webmasters-2].

However, now we also have a copy of the site running at /en, so the base tag needs to be appropriate.

## Grunt to the rescue, again!

We need to update the base tag when making a copy of the site to match the path. Grunt's copy task features a process option that is called for each file copied. The process option enables us to transform the contents of the file:


```javascript
copy: {
  options: {
    processContent: function (contents, srcpath) {
      if(srcpath.indexOf('index.html') >= 0){
        //note spaces around in order not to match 'hreflang' tag
        contents = contents.replace(' lang="da" ', ' lang="en" ');
        contents = contents.replace('<base href="/">', '<base href="/en/">');
      }
      return contents;
    }
  }
  //build_en_lang omitted
}
```

We are using older version of Grunt, so the option is called `processContent` instead of `process`.

HTML5 mode also requires URL rewriting on the server side. If a request is made at example.com/about, the server should still return the index.html page and it is up to Angular router to figure out what to do with the 'about' path variable. In Nginx, this is typically achieved with try\_files directive:

```bash
location / {
    try_files $uri $uri/ $uri/index.html /index.html;
} 
```

If a request is made at example.com/en/about, the configuration above will result in Nginx trying following files:

- en/about --> not found
- en/about/ --> not found
- en/about/index.html  --> not found
- index.html --> found

This results in the server returning the index.html file of the root rather than of 'en' directory. Now the router is trying to figure out which state to match the `en/about` path to, which it will of course fail to do accomplish. We need to define another location to load proper index.html, if $uri is not a match:

```bash
location /en/ {
    try_files $uri $uri/ $uri/index.html /en/index.html;
} 
```

Finally, SEO friendly multilingual application!<br/>Now that we've established that there is no way around configuring web server, let's make most of it! Since the only file that actually changed is index.html, there is no real need to copy other resources. We start out by changing the grunt copy task to only copy index file to same directory and rename it to index.en.html:

```javascript
copy{
  build_en_lang: {
    options: {
      processContent: function (contents) {
        //note spaces around in order not to match 'hreflang' tag
        contents = contents.replace(' lang="da" ', ' lang="en" ');
        contents = contents.replace('<base href="/">', '<base href="/en/">');
        return contents;
      },
    },
    files: [
      {
        src: [ 'index.html'],
        dest: '<%= build_dir %>/',
        cwd: '<%= build_dir %>',
        expand: true,
        rename: function(dest, src){
          return dest + src.replace('.html', '.en.html');
        }
      }
    ]
  }
}
```

Now Nginx 'en' location needs to serve index.en.html instead of index.html, otherwise it should ignore /en/ part of the URL when serving static files. We will use regular expression to match /en and capture everything that comes after into $1. Then we use captured group in try\_files directive:

```bash
location ~ ^/en/?(.*)$ {
  index index.en.html;
  try_files /$1 /$1/ /index.en.html;
}
```

We have jumped through several hoops in order to get rid of the comfort angular-translate provides. Fortunately we can bring some of it back - authenticated users are not crawlers, so we can display 'quick buttons' for them while displaying SEO friendly cross-links to unauthenticated users:

```html
<div ng-if="!auth.isAuthenticated">
  <a ng-href="{{enUrl}}" target="_self" ng-class="{'active': activeLang === 'en-us'}">
    <img src="assets/images/gb.svg" alt="Change language to English"/>
  </a>
  ...
</div>
<div ng-if="auth.isAuthenticated">
  <a ng-href="#" ng-click="changeLang('en-us')" ng-class="{'active': activeLang === 'en-us'}">
    <img src="assets/images/gb.svg" alt="Change language to English"/>
  </a>
  ...
</div>
```

## Conclusion

With this approach we have tried to solve as much as possible with configuration, so we've managed not to entangle language awareness with every state in the application. While ui-router configuration would be preferred, it does not come without limitations: ui-router can not be used for sub-domains and ui-router enforces using language in every url rather than having it as optional parameter.

At the end of day there is no good way to to play by SEO rules, so hopefully AngularJS 2.0 will change this.

[update-post]: https://fadeit.dk/blog/post/angular-translate-ui-router-seo
[google-webmasters]: https://support.google.com/webmasters/answer/182192
[google-webmasters-2]: https://support.google.com/webmasters/answer/182192
[angular-translate]: http://angular-translate.github.io/
