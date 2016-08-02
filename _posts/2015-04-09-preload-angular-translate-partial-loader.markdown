---
layout: post
title:  "Pre-loading partial loader translations"
description: "Finding the balance between eager loading and lazy loading translation tables"
date:   2015-04-09 16:20:16+02:00
size: 12
sizeDesktop: 4
textColor: grey-50
metaColor: grey-600
comments: true
author: sander
---

## Introduction

[Angular-translate][angular-translate] is the de-facto internationalisation module for AngularJS applications. Basically it works by replacing translation keys from the translation dictionary. The problem is that larger applications have a lot of content, thus a lot of translations so it doesn't make sense to send the entire translations file to the client on initial page load. The good news is that there is a solution - [partial loader][loader-partial]. It works as advertised - we can define translation files that are required to load for that page, thus lowering the amount of data required to transfer. The downside to partial loading is that first browser must initialize angular, which in turn will start loading the translation partials. The issue becomes apparent on slower connections where translation keys are either shown or invisible until partial loader finishes fetching the files. It can be a factor when trying to make a first impression. What if we could pre-load some parts that are common throughout our application? It would help improve the user experience on most pages, but will not eliminate the use of partial translations.

Behind the scenes the partial loader uses angular's `$http` service to fetch a JSON file and store it in an array. Fortunately for us, angular has a built-in caching mechanism for `$http` service that we could leverage for our purpose. We are going to store the parts we want to pre-load in the $http cache, so when partial loader tries to fetch the files, it will hit the cache.<br/>By default the caching mechanism for `$http` service is using `$cacheFactory`'s cache with `'$http'` key, although caching mechanism can be overridden. Cache key is the request URL and value is an array containing 4 items:

- response status - 200 for success
- response headers - we leave it blank
- data - JSON for our needs
- status text - 'OK' for fulfilled request

Assuming the translation file we want to pre-load is available at http://example.com/assets/translations/da-dk/common.json, let's test out the concept by creating a function that fakes cache entry for us:


```javascript
var app = angular.module('app', ['pascalprecht.translate']);
app.config(function($translateProvider) {
    $translateProvider.useLoader('$translatePartialLoader', {
        urlTemplate: '/assets/translations/{lang}/{part}.json',
        $http: { cache: true } //Enable caching for PartialLoader
    });
});
app.run(function run ($cacheFactory) {
    var httpCache = $cacheFactory.get('$http');

    var putHttpCache = function putHttpCache(cacheKey, cacheValue){
        var cacheObj = [200, cacheValue, {}, "OK"];
        httpCache.put(cacheKey, cacheObj);
    };
    var cacheValue = angular.toJson({TEST_KEY: "Cached key test"});
    putHttpCache('/assets/translations/da-dk/common.json', cacheValue);
});
```

It works, however we have translation keys in code. Now we need to come up with a way to ship the desired JSON files at initial application load. An approach would be to keep translations in a .js file instead (an angular service, returning JSON string on call), however that would require us to migrate existing JSON files into Javascript objects and reduce translation maintainability. But wait, there is a Grunt plugin that already stores HTML templates in a similar way ([html2js][html2js]). The only difference is that the plugin uses `$templateCache` instead. Since `html2js` is part of our build process already, what we can do is pass the JSON translations to the angular app using the existing Grunt build task. Once angular is bootstrapped, we'll load translations from `$templateCache` and store them into the `$cacheFactory` cache as well. Here's how the Grunt task for storing .json files in `$templateCache` looks like:


```javascript
html2js: {
    //...template caching task omitted...
    translations: {
        options: {
            htmlmin: {}, //override minification settings
            base: 'src'
        },
        src: [ 'src/assets/translations/**/*.json' ],
        dest: '<%= build_dir %>/templates-translations.js'
    }
}
```

Let's now 'teach' the `putHttpCache` method to take translations from `$templateCache`:


```javascript
app.run(function run ($cacheFactory, $templateCache) {
    var httpCache = $cacheFactory.get('$http');
    
    var putHttpCache = function putHttpCache(cacheKey){
        var cacheObj = [200, $templateCache.get(cacheKey), {}, "OK"];
        httpCache.put('/' + cacheKey, cacheObj); //prepend '/' to make url's match
    };
    putHttpCache('assets/translations/da-dk/common.json');
});
```

## Conclusion

Now we can take the best of both worlds - preload most frequently used translations and lazy-load translations for less visited pages. Our html2js build process works in similar manner - all templates ending with `*.tpl2js.html` are stored in `$templateCache`, whereas templates ending with `*.tpl.html` are only fetched on demand.

[angular-translate]: https://angular-translate.github.io/
[loader-partial]: https://github.com/angular-translate/angular-translate/blob/master/src/service/loader-partial.js
[html2js]: https://github.com/karlgoldstein/grunt-html2js
