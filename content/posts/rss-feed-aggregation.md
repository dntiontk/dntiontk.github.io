+++
title = "RSS Feed aggregation"
date = "2023-04-03"
author = "dev"
cover = ""
tags = ["code", "windsor", "rss", "aggregation", "news", "civics"]
showFullContent = false
+++

This post is related to my previous post about [civic code]({{< ref "/posts/civic-code.md" >}}). I mentioned that there are multiple RSS feeds that the city maintains, and that this data is not particularly easy to find and parse. 

We are going to create an RSS feed aggregator creates a weekly summary and automatically creates a new post on this blog.

## The data

First, we need to identify which feeds we'll be using. For now, we are going to keep the list small, however we expect that there will be more sources as time goes on:
- City of Windsor Newsroom -> https://www.citywindsor.ca/Pages/RssFeed.aspx?Catalogue=News
- City of Windsor Open Data Catalogue -> https://opendata.citywindsor.ca/RSS

I like to make a quick prototype out the expected output before beginning development. I typically keep it quick and dirty, with just enough information to get started.

The output will look something like:

```
### Weekly summary for {week}

#### News
- itemized list of changes from [previous week](link to previous week)

#### Open Data Catalogue
- itemized list of changes from [previous week](link to previous week)
```

## The process

The flow we expect to use is that our aggregator fetches the current raw feed (if it exists) from its local storage. It compares the local copy to the remote copy of the feed and uses that data to summarize the changes over the past week. The summary is published as a `hugo content post`, the local feed state is updated, and a commit is pushed via git. 

## The aggregator

As I started to explore the Open Data catalogue, I found something that suprised me, although in retrospect, I should have known. The Open Data RSS feed is **horribly** maintained. It seems that files are uploaded frequently, but there is no metadata. Basically, every item has a title, a link (usually a YTD .csv file) and a publish date. 

Before we get to the code, I'm going to go on a little bit of a rant. City-based Open Data catalogues provide an incredible opportunity for a city to grow, become more resilient, and make effective and positive change. We can better understand and address the issues that affect our communities. We can make intelligent and informed decisions, and really take control of our collective destiny.

Anyway, here is the data structure of the 5th item in the feed:
```
{
    "title": "05 Grand Marais.csv",
    "link": "http://opendata.citywindsor.ca/uploads/05 Grand Marais.csv",
    "links": [
        "http://opendata.citywindsor.ca/uploads/05 Grand Marais.csv"
    ],
    "pubDate": "3/23/2023 2:46:24 PM",
    "pubDateParsed": "2023-03-23T14:46:24Z"
}
```

*What is this?* After doing a bit of digging, I found out that its **precipitation** data from the Grand Marais Rd. precipitation gauge.

And this is what I mean when I say that it feels like they've made it intentionally bad. 

---

Now let's get to the code. We have a pretty simple program for the most part. 

First we parse the local copy of the Open Data feed. If it doesn't exist, we return an empty feed. We then add each item to a map with a formatted timestamp.

```golang
// main.go#61-80
// Parse our local copy of the opendata feed
localOpendataFeed, err := opendataFeed.parseLocalFeed()
if err != nil {
	log.Fatal(err)
}

itemMap := make(map[string]time.Time)

for _, item := range localOpendataFeed.Items {
	formatted := item.PubDateParsed.Format(time.RFC3339)
	pubDate, err := time.Parse(time.RFC3339, formatted)
	if err != nil {
		log.Fatalf("unable to parse date from local feed: %v", err)
	}
	itemMap[item.Title] = pubDate
}
```

Next we parse the remote feed, and write the XML data to a file. We're going to use a data type we'll call `FeedConfig` to simplify this process.

```golang
// main.go#147-167
func (fc *FeedConfig) parseRemoteFeed(c *http.Client) (*rss.Feed, error) {
	resp, err := c.Get(fc.url)
	if err != nil {
		return nil, fmt.Errorf("unable to get remote feed: %v", err)
	}
	defer resp.Body.Close()

	data, err := io.ReadAll(resp.Body)
	if err != nil {
		return nil, err
	}
	if err := fc.write(data); err != nil {
		return nil, err
	}

	feed, err := fc.parseRSSFeed(bytes.NewBuffer(data))
	if err != nil {
		return nil, fmt.Errorf("unable to parse remote feed: %v", err)
	}
	return feed, nil
}
```

Lastly, we lookup each item in the map we created earlier. If the item is not in the map, we add it to our updated items list. If the item is in the map, but the timestamps don't match, we add it to updated items list.

```golang
// main#103-120
func lookupUpdates(m map[string]time.Time, items []*rss.Item) ([]*rss.Item, error) {
	out := make([]*rss.Item, 0)
	for _, i := range items {
		if date, ok := m[i.Title]; ok {
			formatted := i.PubDateParsed.Format(time.RFC3339)
			rDate, err := time.Parse(time.RFC3339, formatted)
			if err != nil {
				return nil, err
			}
			if !rDate.Equal(date) {
				out = append(out, i)
			}
		} else {
			out = append(out, i)
		}
	}
	return out, nil
}
```

To see the full code, go to https://github.com/dntiontk/dntiontk.github.io/code/rss-feed-aggregator


Next time, we'll work on setting up our **publisher** Github Action to generate a new hugo post and commit the change. We'll also try adding in our Newsroom feed.

---

**Keep coding with purpose!  ::dev**
