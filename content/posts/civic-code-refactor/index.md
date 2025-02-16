---
title: "Civic Code Refactor"
date: 2025-02-16T13:36:53-05:00
draft: false
---

In November of 2024, I spent some time reworking the [city council document scraper]({{< ref "posts/scraping-council-meetings" >}}).

The original scraper was a quick and dirty way to get City Council and Committee meeting documents from the City of Windsor's website, and into something more accessible. I was happy with the results, but I knew that the code itself was not great, and if the scraper was going to be maintained, it needed to be improved.

With that in mind, I ported the code for the scraper over to [civic-code](https://github.com/dntiontk/civic-code). The code is cleaner, and will be easier to maintain long-term.

The new scraper is still a work in progress, but it is already better than the original.

I've also added a new tool, [doc-search](https://github.com/dntiontk/civic-code?tab=readme-ov-file#doc-search), which allows you to search, sort, and filter the documents.

I'm utilizing this tool to automatically generate updates once a week at [Documents/City Council]({{ ref "documents/city-council" }}). I'm hoping to add more tools in the future, so stay tuned!

---

**Keep coding with purpose!  ::dev**

