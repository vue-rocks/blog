---
layout: post
title:  "Digital Ocean DNS setup for Mailgun"
description: "Making Digital Ocean and Mailgun play nice"
date:   2015-11-03 11:22:30+01:00
size: 12
sizeDesktop: 8
textColor: grey-50
metaColor: grey-600
comments: true
author: sander
---

When setting up Mailgun (MG) & Digital Ocean (DO) I ran into unexpected complications. Unfortunately DNS configuration feedback l<font size="4">&infin;</font>p takes long time (as DNS settings propagate), so it ended taking me some days just to try couple configurations. There are some misleading tutorials out there as-well, so hopefully I can clear it once for all.

So the first gotcha with DO is that the domain name is automatically added to the DNS records, therefore if MG setting says `email.fadeit.dk`, then that means only `email` should be entered in DO:

![Domain is automatically appended][cname-img]

Second gotcha requires us to wrap TXT record value in quotes:

![TXT records need to be wrapped in quotes][quotes-img]

So-far we've looked at the setup as it is for primary domain, however MG suggests configuring a subdomain instead. While it is possible to set the TXT & CNAME records for a subdomain, MX records can not be configured for a subdomain that way. Fortunately there is a (non-obvious) solution - subdomain should be added as a domain in DO panel:

![Subdomain should be added as a domain][subdomain-img]

Now we can configure MX records for that subdomain, and while at it...move other records to the subdomain configuration aswell. Keep in mind that now DO appends entire subdomain to TXT & CNAME records:

![Subdomain should be added as a domain][conf-img]


[cname-img]: {{ site.baseurl }}/assets/mailgun-dns-setup-digital-ocean/cname.jpg
[conf-img]: {{ site.baseurl }}/assets/mailgun-dns-setup-digital-ocean/conf.jpg
[quotes-img]: {{ site.baseurl }}/assets/mailgun-dns-setup-digital-ocean/quotes.jpg
[subdomain-img]: {{ site.baseurl }}/assets/mailgun-dns-setup-digital-ocean/subdomain.jpg
