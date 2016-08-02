---
layout: post
title:  "Blacklist Referer Spam Bots with NGINX"
description: "Seeing weird things in your analytics dashboard? Here's an easy approach for blacklisting Referer Spam Bots with NGINX."
date:   2015-04-22 18:51:00+02:00
size: 12
sizeDesktop: 4
textColor: grey-50
metaColor: grey-600
comments: true
author: justas
---

## Some Background

Recently, we submitted fadeit.dk to [AWWWARDS][awwwards] - the awards for design, creativity and innovation on the Internet. It gave us a nice little boost of website visitors. However, not all of that attention was positive...

## Referer Spam Bots

While the majority of our traffic came from genuine sources, we started noticing a pattern in our referral traffic.

![Google Analytics Dashboard - Acquisition / Referrals][ga-dashboard]



What's up with `social-buttons.com`? We didn't sign up for that... Apparently, this is something called [Referer Spam][wiki]\*:

> Referrer spam (also known as log spam or referrer bombing) is a kind of spamdexing (spamming aimed at search engines). The technique involves making repeated web site requests using a fake referer URL to the site the spammer wishes to advertise. Sites that publish their access logs, including referer statistics, will then inadvertently link back to the spammer's site. These links will be indexed by search engines as they crawl the access logs. - Wikipedia

This is not cool, Mr. social-buttons.com. P\*\*s off!

*\* No, I didn't mispell. The mispelling actually made it into the [HTTP/1.0 standard][rfc] and now it's there forever :)*

## NGINX Solution

Since we're using NGINX to serve our site, the solution is going to be described for NGINX. Apache people can take a look at this excellent [article][apache-block].

The official NGINX wiki does mention [a solution][nginx-block] to this problem. Basically, you just use [ngx_http_referer_module][referer-mod]\* and add something like this to your location or server block:


```bash
valid_referers none blocked server_names *.social-buttons.com social-buttons.com badreferer2.com;

if ($invalid_referer) {
  return   444;
}
```

This works, but what if we want to maintain a larger blacklist of referers? Our `valid_referers` directive would get crazy long. If that's fine with you, you can stop reading here. It sure isn't fine with me :).

In order to make our blacklist more maintainable, we can use [ngx\_http\_map\_module][map_mod]. Let's save `/etc/nginx/blacklist.conf` file with the following content:


```bash
# /etc/nginx/blacklist.conf
 
map $http_referer $bad_referer {
    hostnames;
 
    default                           0;

    # Put regexes for undesired referers here
    "~social-buttons.com"             1;
    "~semalt.com"                     1;
    "~kambasoft.com"                  1;
    "~savetubevideo.com"              1;
    "~descargar-musica-gratis.net"    1;
    "~7makemoneyonline.com"           1;
    "~baixar-musicas-gratis.com"      1;
    "~iloveitaly.com"                 1;
    "~ilovevitaly.ru"                 1;
    "~fbdownloader.com"               1;
    "~econom.co"                      1;
    "~buttons-for-website.com"        1;
    "~buttons-for-your-website.com"   1;
    "~srecorder.co"                   1;
    "~darodar.com"                    1;
    "~priceg.com"                     1;
    "~blackhatworth.com"              1;
    "~adviceforum.info"               1;
    "~hulfingtonpost.com"             1;
    "~best-seo-solution.com"          1;
    "~googlsucks.com"                 1;
    "~theguardlan.com"                1;
    "~i-x.wiki"                       1;
    "~buy-cheap-online.info"          1;
    "~Get-Free-Traffic-Now.com"       1;
}
```

Now include the file in the the http block of your `/etc/nginx/nginx.conf` file:


```bash
# /etc/nginx/nginx.conf
 
http {
# ...
 
  include blacklist.conf;
 
# ...
 
}
```

All that's left is to add conditions to the sites, for which, you want to block referer spam bots:


```bash
# /etc/nginx/sites-enabled/mysite.conf
 
server {
  # ...
 
  if ($bad_referer) { 
    return 444; 
  } 
 
  # ...
}
```

OK, now let's test if this thing works:


```bash
# with subdomain
 $ curl --referer http://www.social-buttons.com https://fadeit.dk/en
curl: (52) Empty reply from server

# without subdomain
 $ curl --referer http://social-buttons.com https://fadeit.dk/en
curl: (52) Empty reply from server
```

Sweet! It worked.

*\* Both [ngx\_http\_referer\_module][referer-mod] and [ngx\_http\_map\_module][map-mod] are included in the standard NGINX distribution and you don't need to recompile your server.*

## That's it!

What's your experience with Referer Spam? Don't hesitate to use the comment section :)

## Additional Resources


- [Blacklist by oohnoitz][blacklist] which blocks bad bots, pentest tools, surveillance bots, etc. It's an excellent addition to the referer spam blacklist described in this post.
- [Ultimate referer blacklist][blacklist-2]

[awwwards]: http://www.awwwards.com/best-websites/fadeit-1/
[ga-dashboard]: {{ site.baseurl }}/assets/nginx-referer-spam-blacklist/ga_dashboard.png
[wiki]: http://en.wikipedia.org/wiki/Referer_spam
[rfc]: http://tools.ietf.org/html/rfc1945
[apache-block]: https://www.addedbytes.com/blog/block-referrer-spam/
[nginx-block]: http://wiki.nginx.org/Referrer_Spam_Blocking
[referer-mod]: http://nginx.org/en/docs/http/ngx_http_referer_module.html
[map-mod]: http://nginx.org/en/docs/http/ngx_http_map_module.html
[blacklist]: https://github.com/oohnoitz/nginx-blacklist
[blacklist-2]: http://perishablepress.com/4g-ultimate-referrer-blacklist/
