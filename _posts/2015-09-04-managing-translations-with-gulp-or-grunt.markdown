---
layout: post
title:  "Loading translations from Google Spreadsheets"
description: "Improving our cloud translation management workflow."
date:   2015-09-04 11:16:30+02:00
size: 12
sizeDesktop: 4
textColor: grey-50
metaColor: grey-600
author: sander
---

## Introduction

This is a follow-up post to our [previous][prev-post] translation management article. To recap - in that article we solved the issue of loading 'fresh' translation on development server using nginx and proxying translation requests to the api which in turn will fetch new translations from Google Spreadsheets. It can be seen as overkill - this flow complicates nginx configuration, slows down translation loading on every refresh and will make a ton of (unnecessary) requests while developing. While it's nice to always get the latest translations, truth is that most of the time you don't need to fetch translations from the cloud. In this post we'll explore loading translations only during the build process, thus eliminating the need to proxy requests and write api endpoint in the first place.

## Workflow

Since writing previous post, I have also changed my mind about lazy-loading translations using partial loader. This is primarily due to the lengths I had to go to cloak translations while they're loading, especially considering that the size of the translations file is marginal for most applications. By packaging translations in the same minified file as the application I don't have to worry about [pre-loading][preloader-post] translations or flickering as they are being loaded. So we will introduce an extra task to our workflow that will fetch translations from Google Spreadsheets and converts it to a javascript file ready to be used by angular-translate.

## Using Gulp

Although here at fadeit we started out with using Grunt exclusively, we've come to appreciate Gulp more in our recent projects. Primarily since writing custom tasks in good ol' JavaScript is more intuitive as opposed to Grunt's configuration approach. Surely piping the result of previous task directly to next one without having to write it to disk is a plus, but not in current context.


```javascript
var gulp = require('gulp');
var request = require('request');
var fs = require('fs');

gulp.task('translations', function (cb) {
  var uuid = '1FsVuRLbtgxMZvWd4mpnKiAhqVYap-ZAx08LBeZ9HFJk';
  var page = 'od6'
  var url = 'https://spreadsheets.google.com/feeds/list/' + uuid + '/' + page + '/public/values?alt=json';
  request(url, function(err, res, body){
    var json = JSON.parse(body);
    var translations = {
      'en': {},
      'da': {}
    };
    for(var i = 0; i < json.feed.entry.length; i++){
      var entry = json.feed.entry[i];
      var key = entry.gsx$key.$t;
      var enValue = entry.gsx$en.$t;
      var daValue = entry.gsx$da.$t;
      translations.en[key] = enValue;
      translations.da[key] = daValue;
    }
    writeTranslations(translations, options.src + '/app/translations.js', cb);
  });
});

function writeTranslations(translations, file, cb){
  var contents = "var appModule = angular.module('fadeit');\n";
  contents += "appModule.config(function($translateProvider){\n";
  for(var property in translations){
    var lang = translations[property];
    contents += "$translateProvider.translations('" + property + "',\n";
    contents += JSON.stringify(lang);
    contents += ");\n";
  }
  contents += "});";
  fs.writeFile(file, contents, function(err){
    if(err){
      console.error('Writing translations failed:');
      console.error(err);
    }
    cb();
  });
}
```

As seen from the code, couple modules are used - namely `request` to fetch the JSON data and `fs` to write a file. The task won't only write the file, but it will also generate an angular.js module file that will configure translations ready to be used. It can be seen a bit hacky to generate javascript file like that, but I've yet to come up with more clean solution. The upside is that this code is not prone to change and I haven't touched it since writing it. There's caveat though - if you are using [browsersync][browsersync], then it may trigger refresh before the translations are actually pulled (because other files changed before). To fix this, we need to make sure that translations are loaded before any other tasks take place. It is easily solved with gulp-sync module, to execute certain tasks synchronously. Same would also apply if the process will concatenate files together - we need to make sure new translations are loaded before that happens. In the sample build process below I've split up the build process in two portions where each of them is executed asynchronously. The callback at the end of the gulp task is necessary so that gulp-sync knows when to start processing next batch of tasks.


```javascript
var gulpsync = require('gulp-sync')(gulp);
gulp.task('build', gulpsync.sync([['translations', 'clean'], ['html', 'scripts']]));
```

## Using Grunt

Firstly I must apologise as I did not take time to develop a pure grunt way of doing so. This is because it started out as a python script which I later incorporated in the grunt workflow. The script will serve same purpose as the one above, with a minor difference of calling an external python executeable instead.


```python
import json
import urllib.request
from collections import OrderedDict

sheet_id = '1FsVuRLbtgxMZvWd4mpnKiAhqVYap-ZAx08LBeZ9HFJk';
page_id = 'od6'
base = 'https://spreadsheets.google.com/feeds/list/{0}/{1}/public/values?alt=json'
translation_config = 'src/app/translations.js'

url = base.format(sheet_id, page_id)
json_data = json.loads(urllib.request.urlopen(url).read().decode('utf8'))
#we need to build different data structure
en_translation_map = OrderedDict()
da_translation_map = OrderedDict()
for entry in json_data['feed']['entry']:
    key = entry['gsx$key']['$t']
    en_value = entry['gsx$en']['$t']
    da_value = entry['gsx$da']['$t']
    en_translation_map[key] = en_value
    da_translation_map[key] = da_value
#Javascript for translations file we are generating
config = """
var appModule = angular.module('fadeit');
appModule.config(function($translateProvider){
"""
en_translations = json.dumps(en_translation_map, indent=4)
da_translations = json.dumps(da_translation_map, indent=4)
config = config + "\n$translateProvider.translations('en'," + en_translations + ');'
config = config + "\n$translateProvider.translations('da'," + da_translations + ');'
config = config + "\n});" #close function
with open(translation_config, 'w') as outfile:
    outfile.write(config)
```

Now we can save this file in the project (in our case src/translations.py) and call it using grunt-shell during build time.


```javascript
shell: {
  translations: {
    options: {
      stdout: true
    },
    command: 'python3 src/translations.py'
  }                                                                                                                                                   
}
```

Since grunt executes tasks synchronously, all there is left to do is call this task in the beginning of the build process and it'll generate a following file.


```javascript
var appModule = angular.module('fadeit');
appModule.config(function($translateProvider) {
  $translateProvider.translations('en',
    {
      "HELLO": "Hello"
    }
  );
  $translateProvider.translations('da',
    {
      "HELLO": "Hej"
    }
  );
});


```

## Closing words

Over-all I've come to prefer loading translations during build time instead. While it will add few seconds to the build process, I will make it up by not having to wait for translations being proxied on every reload. If I already have a server running and just wish to reload translations, I can issue the command directly instead of re-building everything. For gulp it would be `gulp translations` and for grunt `grunt shell:translations`. I've also decided to ditch versioning translations and generally will add `src/translations.js` to `.gitignore`.

[prev-post]: https://fadeit.dk/blog/post/managing-angular-translate-translations
[preloader-post]: https://fadeit.dk/blog/post/preload-angular-translate-partial-loader
[browsersync]: http://www.browsersync.io/
