+++
title = "city coding"
date = "2023-03-30"
author = "dev"
cover = ""
tags = ["code", "politics", "windsor", "city-council", "planning", "analytics"]
showFullContent = false
+++

I frequently find myself on the [City of Windsor](https://www.citywindsor.ca/Pages/Home.aspx) website. I'll scroll through city council meeting agendas and videos, browse the open data catalogue, check for newsroom headlines; annoyed by how outdated it feels the entire time. This isn't about the look of the site either. I don't really care about the asthetics. I'm talking about it feeling like the site is intentionally disjointed and built in a way to make engagement a *little bit* harder. 

I'm not saying that *it is* purposeful. But if it was, and the goal was to encourage **apathy** while maintaining plausible deniability, I would consider it a very effective website.

There are absolutely positive things about the website and there is a lot of data if don't mind digging. However, the user experience is a deterrent. 

As you can probably tell from this site, my "user experience" skills aren't the best... but I do have the skills to improve **my** experience when interacting with the city's data.

### The plan

I've broken the plan down into three areas of focus to start. There may be more changes, but this gives me a place to start.

#### Open Data

There is an Open Data catalogue with 98 items, each of which have 1 or more data files. There is an RSS feed, so we have the ability to be notified on updates. We can use this data to build out dashboards, maps and other forms of data visualization that are actually meaningful to our community. Making data available is not the same as making data accessible.

#### Info Feeds

There are a few different RSS feeds that the city maintains. The one that I'm particularly interested in is the [Newsroom Headlines](https://www.citywindsor.ca/newsroom/Pages/default.aspx). Pairing this with other local news RSS feeds like [CBC Windsor](https://www.cbc.ca/cmlink/rss-canada-windsor) would allow the building of a simple news aggregator. Unfortunately, [Windsorite.ca](https://windsorite.ca/) does not have an RSS feed, but maybe that is a future project.

#### Council Data

This area was originally what got me fired up about this. Why, in the year 2023, on Soulja Boy's internet, do we not have downloadable transcriptions of city council meetings? You can get captions, if you watch the video on the janky sliq.net site, but its just the captions included in the screen recording of a Zoom session. The meeting index is not well organized either. You have to open up a menu just to see what documents (agenda, minutes, video link, appendices, etc.) have been added to a particular meeting. The meeting documents, and the video are hosted on separate sites, meaning I need both open if I want to follow along. Also, I shouldn't need a browser to download city council meeting videos. Ideally, I should be able to download the video, transcription and associated documents via an API.

### The action

The goal is to create a bunch of smaller programs (mostly in Golang, but maybe some python if necessary) that can interact with this blog in an automated way to give me (and hopefully others) a more user-friendly experience. This also allows me to play with certain disciplines (AI/Machine Learning, RSS feed aggregation, data vizualization) in a low-stakes sort of way.

---

**Keep coding with purpose!  ::dev**
