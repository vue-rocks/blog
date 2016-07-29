---
layout: post
title:  "How to manage multilingual website translations?"
description: "Translations in different languages, files and formats have a tendency to become messy - outsourcing translation management to the cloud can be an easy way out."
date:   2015-05-08 12:25:54+02:00
size: 12
sizeDesktop: 4
textColor: grey-50
metaColor: grey-600
author: sander
---

**Update**: I have written a [follow-up post][follow-up] where I load translations during gulp/grunt build process instead of nginx proxy.

## Introduction

This article focuses on translation management for AngularJS, however it can be applied to any translation management process. Truth is... I haven't been a fan of the approach [gettext][gettext] takes, primarily because of the translation management process. I find the work-flow of having to extract translations from the code (.po file), send them to translator, compiling (.mo file) and deploying to consume significant amount of time. Key based translation like angular-translate may simplify certain aspects, but JSON files are still a limitation. They are uncomfortable to edit, prone to merge conflicts and have a strict format.

## Our apporach

Fortunately there is a tool out there that offers flexible user interface, real-time editing, it fast and easy to integrate with - Google Sheets. You may know that it is possible to publish spreadsheet as HTML, however what they don't tell you is that there is also an endpoint for returning the data in JSON format.

![Publishing Google Sheets][publish-img]

Google will then give you a URL that looks like this:

[https://docs.google.com/spreadsheets/d/1FsVuRLbtgxMZvWd4mpnKiAhqVYap-ZAx08LBeZ9HFJk/pubhtml][sheet-1].

To access JSON variant, take the id of document - `1FsVuRLbtgxMZvWd4mpnKiAhqVYap-ZAx08LBeZ9HFJk` and use it in spreadsheet feed service URL:

[https://spreadsheets.google.com/feeds/list/1FsVuRLbtgxMZvWd4mpnKiAhqVYap-ZAx08LBeZ9HFJk/od6/public/values?alt=json][sheet-2].

That URL however will only display data for first page only - by default first page id in Google Docs is `od6`. In order to organize translations we want to store them on multiple pages (I have created Common & Register pages in the screenshot above), therefore we need to get IDs of other pages using yet another feed:

[https://spreadsheets.google.com/feeds/worksheets/1FsVuRLbtgxMZvWd4mpnKiAhqVYap-ZAx08LBeZ9HFJk/private/full][sheet-3].

From the XML we can search for `</id>` to extract other sheet IDs. As we see, the ID for second sheet is no longer following any pattern - [o14w8rc][sheet-5].

Even more feeds are covered on the [Official API documentation][sheet-api-docs].

![Formatted JSON from Google Spreadsheet feed][formatted-img]

While JSON representation is good starting point, it is a serialized representation of the document that contains a lot of meta-data. We are only interesed in the nested values, so data-structure to be transformed into what i18n framework can read. For that we are going to implement an API endpoint that will fetch translations from Google Docs, transform the structure and return translations. As seen from the screenshot above, the pattern for extracting a value is `feed->entry->gsx${language}->$t`. The minimal example below utilizes [Flask microframework][flask] written in Python


```python
import os
import json
import urllib
from flask import Flask, abort
from collections import OrderedDict


app = Flask(__name__)

@app.route('/translation/<lang>/<filename>')
def fetch_translation(lang, filename):
    #  Mapping Google Docs page ID to the translation file
    page_map = {
        'common.json': 'od6',
        'register.json': 'o14w8rc'
    }

    sheet_id = '1FsVuRLbtgxMZvWd4mpnKiAhqVYap-ZAx08LBeZ9HFJk'
    base = 'https://spreadsheets.google.com/feeds/list/{0}/{1}/public/values?alt=json'
    if filename not in page_map:
        abort(404)
    page_id = page_map[filename]
    url = base.format(sheet_id, page_id)

    json_data = json.loads(urllib.request.urlopen(url).read().decode('utf8'))
    #  We need to build different data structure
    translation_map = OrderedDict()
    for entry in json_data['feed']['entry']:
        key = entry['gsx$key']['$t']
        value = entry['gsx${0}'.format(lang)]['$t']
        if value:
            translation_map[key] = value

    return json.dumps(translation_map)
        
if __name__ == '__main__':
    app.run()
 
```

Now let's run the server and fetch translations at http://127.0.0.1:5000/translation/en-us/register.json


```bash
curl http://127.0.0.1:5000/translation/en-us/register.json
{"WELCOME": "Welcome"}
```

Fetching translations is convenient while developing and translating, however we wouldn't want production server to fetch translations from Google Docs every time it gets a request. Easiest solution would be to cache translations for certain amount of time, we also want to version control translations, therefore the strategy is a little different. Production server would be serving static JSON files from the disc, whereas on development instance NGINX will intercept any requests going to /translation and proxy them to the application server instead:

![Translation loading flow][flow-img]

For development NGINX configuration we need to add a location block like so:


```bash
#proxy translations from Google Docs on development
location /translation {
    proxy_pass         http://127.0.0.1:5000$uri;
    proxy_redirect     off;
    proxy_set_header   Host             $host;
    proxy_set_header   X-Real-IP        $remote_addr;
    proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
} 
```

## Conclusion

All translations now come from one consistent source, no more mismatches and conflicts.

With Git we get:

- Accidental modifications in Google Docs don't automatically end up in production
- Each translation has a revision history</li><li>Finding differences between edits is simple
- Blame view helps finding wrongdoer

Google Docs brings to the table:

- Spotting missing translations is easy because all languages are visible on same sheet (.json files are separate for each language)
- Access and control management of editors is simple
- Multiple people can edit same document in realtime
- Changes to translations are visible immediately (on development setup)
- Sheets supports highlighting and commenting

Dependencies needed to run the flask server:

```bash
#Install dependencies
sudo apt-get install virtualenv python3.4-dev python-pip
#Create virtual environment
virtualenv -p /usr/bin/python3.4 venv
#Activate virtual environment
source venv/bin/activate
#Install libraries
pip install flask
#Run server
python server.py
#Fetch translations
http://localhost:5000/translation/en-us/register.json
```

[follow-up]: https://fadeit.dk/blog/post/managing-translations-with-gulp-or-grunt
[gettext]: https://en.wikipedia.org/wiki/Gettext
[publish-img]: {{ site.baseurl }}/assets/managing-angular-translate-translations/publish.png
[formatted-img]: {{ site.baseurl }}/assets/managing-angular-translate-translations/formatted.png
[flow-img]: {{ site.baseurl }}/assets/managing-angular-translate-translations/flow.png
[sheet-1]: https://docs.google.com/spreadsheets/d/1FsVuRLbtgxMZvWd4mpnKiAhqVYap-ZAx08LBeZ9HFJk/pubhtml
[sheet-2]: https://spreadsheets.google.com/feeds/list/1FsVuRLbtgxMZvWd4mpnKiAhqVYap-ZAx08LBeZ9HFJk/od6/public/values?alt=json
[sheet-3]: https://spreadsheets.google.com/feeds/worksheets/1FsVuRLbtgxMZvWd4mpnKiAhqVYap-ZAx08LBeZ9HFJk/private/full
[sheet-5]: https://spreadsheets.google.com/feeds/list/1FsVuRLbtgxMZvWd4mpnKiAhqVYap-ZAx08LBeZ9HFJk/o14w8rc/public/values?alt=json
[sheet-api-docs]: https://developers.google.com/google-apps/spreadsheets/
[flask]: http://flask.pocoo.org/
