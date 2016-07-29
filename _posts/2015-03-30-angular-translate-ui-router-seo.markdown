---
layout: post
title:  "AngularJS SEO friendly translation URLs with ui-router"
description: "Making your multilingual AngularJS application properly crawlable by search engine bots."
date:   2015-03-30 18:10:55+02:00
size: 12
sizeDesktop: 4
textColor: grey-50
metaColor: grey-600
author: sander
---

## Introduction

In [previous][prev-post] post we explored how to introduce unique URLs per language using primarily NGINX and Grunt configuration. To recap, that approach is reasonable if you already have a working application and prefer not to re-factor every state and view that links to it. While it may be easier to implement in an existing application, it forces users to perform a refresh when switching between languages - though not a big issue (how often do users switch back-and-forth anyway), having a Single Page Application (SPA) that forces users to reload pages is sub-optimal. In this article we will explore possibilities of implementing SEO friendly URLs using ui-router.

Nested states is the key to introducing language as a URL variable to all states. We start off by creating the 'app' state that every other state will be child of. The app state is abstract, so that the state by itself can not be activated. Next we want the route to only match the languages we intend to support (in this example 'da' and 'en') to reduce possible false-positive matches e.g example.com/<b>da</b>ta. As ui-router injects the content of the child states into ui-view, we define it as an inline template for the parent route. We also define 'app.home' state that will handle example.com/en and example.com/da pages:


```javascript
$stateProvider.state('app', {
  abstract: true,
  url: '/{lang:(?:da|en)}',
  template: '<ui-view/>'
});
$stateProvider.state('app.home', {
  url: '',
    templateUrl: 'views/home-page.html',
});
```

Now every state that we want to translate should be a child of app state:


```javascript
$stateProvider.state('app.about', {
  url: '/about',
  templateUrl: 'views/about-page.html',
});
```

Since the child states contain a language path variable, we need to alter the links pointing to the child states to pass in `lang` parameter. But before we do so, we need to set `activeLang` on `$rootScope` so we can read it from there when generating links with `ui-sref`. Due to the fact that `activeLang` can change at state change, we need to subscribe to `stateChangeSuccess` event. While at it, we can also generate links for the same page in other languages - 'In English' link at example.com/da/about should point to example.com/en/about. In this simple example we use only two languages, so we'll be using only one variable `otherLangURL`:


```javascript
function navigationController($scope, $rootScope, $stateParams, $translate){
  $scope.$on('$stateChangeSuccess', function rootStateChangeSuccess(event, toState){
    if($stateParams.lang !== undefined){
        var otherLang = $stateParams.lang === 'da' ? 'en' : 'da';
        $rootScope.activeLang = $stateParams.lang;
        $rootScope.otherLangURL = $location.absUrl().replace('/' + $stateParams.lang, '/' +otherLang);
        $translate.use($stateParams.lang);
    }
  });
}
```

It would be nice to use `ui-sref` to dynamically generate the links to same page in other languages, but unfortunately `ui-sref` does not support variables in state names, therefore we have to fall back to `ng-href` for generating the links. Fortunately path variable substitution is supported, therefore we can generate menu links using `activeLang` variable we stored on `$rootScope`:


```html
<ul class="menu-navigation">
  ...
  <li ui-sref-active="activeMenu">
    <a href ui-sref="app.about({lang: activeLang})" translate>
      NAV_ABOUT
    </a>
  </li>
  <li>
    <a ng-href="{{otherLangURL}}" translate>
      IN_LANGUAGE
    </a>
  </li>
</ul>
```

## Conclusion

Incorporating language in the application routes from very start will save time in the long run as re-factoring hundreds of `ui-sref` links will take it's time. Ui-router fully supports nested sub-states, so there will not be any conflict if your application is already using nested states, just make sure that there is `ui-view` for each parent state to inject the content into. I was unable to make the lang parameter in the parent state optional, thus language will be enforced in every path (as opposed to [previous post][prev-post] where we were able to use paths without language defined in them). Have a better approach? leave a comment!

[prev-post]: https://fadeit.dk/blog/post/angularjs-seo-for-angular-translate
