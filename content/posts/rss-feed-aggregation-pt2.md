+++
title = "RSS Feed aggregation part 2"
date = "2023-04-08"
author = "dev"
cover = ""
tags = ["code", "windsor", "rss", "aggregation", "news", "civics", "automation"]
showFullContent = false
+++

So we hit some snags in [part 1]({{< ref "/posts/rss-feed-aggregation.md" >}}). This inspired me to start doing some research. I guess I should have done my research earlier because I found a great post by Mita Williams called [Why isn't there an RSS feed for Windsor City Council web pages?](https://civics.aedileworks.com/2018/12/18/why-isnt-there-an-rss-feed-for-windsor-city-council-web-pages/) from 2018.

> I have sent a 311 request that RSS feeds be added to these pages with the next version of the City of Windsor website.

> Dave Meslin argues that our governments use design to discourage engagement.  The good news? We can re-design improvement. Or in this case, we can ask for RSS.

If you haven't, I recommend checking out more of her work at https://www.uofwinds.com/ where you can subscribe to her really insightful weekly newsletter.

So it appears that this **has** been an issue. The feeds now "exist", but they aren't maintained. And its looking intentional. This was made even more clear when I started to dig into the Newsroom feed.

---

If you checked the [code](https://github.com/dntiontk/dntiontk.github.io/blob/78d68df4b67ad079301c7f52ecbec680a1366467/code/rss-feed-aggregator/main.go#L48-L49), you might have noticed that I had to embed the `*.citywindsor.ca` CA certificate into my program to get it to work properly. Early in testing, I noticed that there appears to be an issue with the certificate when using `curl`.

```
$ curl -i https://www.citywindsor.ca/Pages/RssFeed.aspx\?Catalogue\=News

curl: (60) SSL certificate problem: unable to get local issuer certificate
More details here: https://curl.haxx.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
```

I thought this was strange, but figured that I would rather just add the ca cert to my client, bypassing any messing around with my local certificates. Easy fix, and definitely not enough to stiffle my progress.

Progress was stopped as I started to work with the Newsroom feed though. The first thing I see when I click the RSS button in my browser is:

![newsroom rss alert](/static/images/newsroom-alert.png)

The page that opens is blank, which isn't surprising since its a feed. When I inspect the page, I see XML and figure its fine.

I start updating my code, and **surprise**... the response body is empty and I decide to cut my losses and move on.

---

So our original plan was to create a summary with both the Newsroom and the Open Data feed. We now have the Open Data feed working, so we're going to cut the Newsroom for now and move on to the automation.