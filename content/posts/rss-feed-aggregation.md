+++
title = "RSS Feed aggregation"
date = "2023-04-27"
author = "dev"
cover = ""
tags = ["code", "windsor", "rss", "aggregation", "news", "civics"]
showFullContent = false
+++

This post is related to my previous post about [civic code]({{< ref "/posts/civic-code.md" >}}). I mentioned that there are multiple RSS feeds that the city maintains, and that this data is not particularly easy to find and parse. 

We are going to create an RSS feed aggregator creates a weekly summary and automatically creates a new post on this blog. For now, we'll just use the [City of Windsor Open Data Catalogue](https://opendata.citywindsor.ca/), but we'll probably add more feeds in a future iteration.

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

And this is what I mean when I say that it feels like they've made it intentionally bad. Rather than getting too hung up on the minor details, we're going to push forward and work with what we have.

---

### The code

Now let's get to the code. We have a pretty simple program for the most part. All the code can be found at https://github.com/dntiontk/rss-feed-aggregator.

First we parse the local copy of the Open Data feed. If it doesn't exist, we return an empty feed:

```golang
func parseLocalFeed(path string) (*rss.Feed, error) {
	b, err := os.ReadFile(path)
	if err != nil {
		if os.IsNotExist(err) {
			return &rss.Feed{}, nil
		}
		return &rss.Feed{}, fmt.Errorf("unable to read local feed: %v", err)
	}


	feed, err := parseRSSFeed(bytes.NewBuffer(b))
	if err != nil {
		return &rss.Feed{}, fmt.Errorf("unable to parse local feed: %v", err)
	}


	return feed, nil
}
```


Next we parse the remote feed, and write the XML data to a file. You may notice that I'm including an HTTP client as an argument. This is to work around an SSL cert issue that I encountered. Instead of messing around with my local certs, I'm just going to add it to the transport cert pool. You can find the code for that further down in this post.

```golang
func parseRemoteFeed(c *http.Client, path, url string) (*rss.Feed, error) {
	resp, err := c.Get(url)
	if err != nil {
		return nil, fmt.Errorf("unable to get remote feed: %v", err)
	}
	defer resp.Body.Close()


	data, err := io.ReadAll(resp.Body)
	if err != nil {
		return nil, err
	}
	if err := write(data, path); err != nil {
		return nil, err
	}


	feed, err := parseRSSFeed(bytes.NewBuffer(data))
	if err != nil {
		return nil, fmt.Errorf("unable to parse remote feed: %v", err)
	}
	return feed, nil
}
```


We're going to now create a `getFeedUpdates` function which fetches the local feed, creates a `map[string]time.Time` to lookup items fetches the remote feed, and call the `lookupUpdates` function.

```golang
func getFeedUpdates(client *http.Client, path, url string) ([]*rss.Item, error) {
	localFeed, err := parseLocalFeed(path)
	if err != nil {
		return nil, err
	}

	/*
		Let's create a map[string]time.Time to quickly lookup items and
		compare dates
	*/
	itemMap := make(map[string]time.Time)


	for _, item := range localFeed.Items {
		formatted := item.PubDateParsed.Format(time.RFC3339)
		pubDate, err := time.Parse(time.RFC3339, formatted)
		if err != nil {
			return nil, fmt.Errorf("unable to parse date from local feed: %v", err)
		}
		itemMap[item.Title] = pubDate
	}


	// Parse the remote copy of the opendata feed
	remoteFeed, err := parseRemoteFeed(client, path, url)
	if err != nil {
		return nil, fmt.Errorf("unable to parse remote feed: %v", err)
	}


	// Make updatedItems lists
	return lookupUpdates(itemMap, remoteFeed.Items)
}
```


We need to lookup each item in the map we created earlier. If the item is not in the map, we add it to our updated items list. If the item is in the map, but the timestamps don't match, we add it to updated items list.

```golang
func lookupUpdates(m map[string]time.Time, items []*rss.Item) ([]*rss.Item, error) {
	updatedItems := make([]*rss.Item, 0)
	for _, i := range items {
		if date, ok := m[i.Title]; ok {
			formatted := i.PubDateParsed.Format(time.RFC3339)
			rDate, err := time.Parse(time.RFC3339, formatted)
			if err != nil {
				return nil, err
			}
			if !rDate.Equal(date) {
				updatedItems = append(updatedItems, i)
			}
		} else {
			updatedItems = append(updatedItems, i)
		}
	}
	return updatedItems, nil
}
```


Here are some helper functions to write to a local file and to parse the RSS feed:

```golang
func write(b []byte, path string) error {
	f, err := os.Create(path)
	if err != nil {
		return err
	}
	if _, err := f.Write(b); err != nil {
		return err
	}
	return nil
}


func parseRSSFeed(r io.Reader) (*rss.Feed, error) {
	fp := rss.Parser{}


	feed, err := fp.Parse(r)
	if err != nil {
		return nil, err
	}
	return feed, nil
}
```


I mentioned above that I encountered an SSL cert issue and I didn't want to mess around with my local certs. We're going to embed the cert and make a function to parse it and return an HTTP client with an updated cert pool.

```golang
//go:embed star.citywindsor.ca
var cert []byte

// newClientWithCA reads a CA cert as bytes and returns an HTTP client with the appropriate cert pool
func newClientWithCA(cert []byte) (*http.Client, error) {
	pool := x509.NewCertPool()
	if ok := pool.AppendCertsFromPEM(cert); !ok {
		return nil, fmt.Errorf("unable to append ca to cert pool")
	}


	return &http.Client{
		Transport: &http.Transport{
			TLSClientConfig: &tls.Config{
				RootCAs: pool,
			},
		},
	}, nil
}
```


When invoking this program, we're going to pass it flags for the path to the local XML and the remote URL of the feed. Our main function wraps all the above and outputs the changes in JSON format.

```golang
func main() {
	flag.StringVar(&pathFlag, "path", "./feeds/opendata.xml", "path to local xml file to diff")
	flag.StringVar(&urlFlag, "url", "https://opendata.citywindsor.ca/RSS", "RSS feed url")
	flag.Parse()
	/*
		Note that we need to add the ca-cert for "citywindsor.ca" to
		to our HTTP client in order to access the data programatically
	*/
	client, err := newClientWithCA(cert)
	if err != nil {
		log.Fatal(err)
	}


	// Get our Open Data update list
	opendataUpdates, err := getFeedUpdates(client, pathFlag, urlFlag)
	if err != nil {
		log.Fatal(err)
	}


	// exit if no changes found
	if len(opendataUpdates) == 0 {
		log.Printf("no changes found")
	} else {
		b, err := json.MarshalIndent(opendataUpdates, "", "  ")
		if err != nil {
			log.Fatal(err)
		}


		log.Printf("%s", b)
	}
}
```


This code will likely change over time, but the concept works. Next time, we'll write the workflow that runs the `rss-feed-aggregator`, creates a new hugo post, updates the local copy of the feed, and commits the changes on a weekly basis.

---

**Keep coding with purpose!  ::dev**
