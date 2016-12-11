+++
Description = ""
Tags = [
  "Development",
  "golang",
  "go"
]
Categories = [
  "Development",
  "GoLang",
]

date = "2016-12-10"
title = "Go web crawler"
highlight = true

+++

Let's make a crawler to explore this website in Go!

<!--more-->

---

View the code here: https://github.com/jeremywho/crawl

### main

We'll start by taking a look at our main function:

    func main() {
        urlProcessor := make(chan string)
        done := make(chan bool)

        go processURL(urlProcessor, done)
        urlProcessor <- "https://jeremywho.com"

        <-done
        fmt.Println("Done")
    }

First we setup our needed channels, one to send urls to the url processor and one that waits until done.

Next we kick off the goroutine to process the urls `processURL` and send it the starting URL.

### Process Url in a goroutine

Let's take a look at the function `processURL`:

    func processURL(urlProcessor chan string, done chan bool) {
        visited := make(map[string]bool)
        for {
            select {
            case url := <-urlProcessor:
                if _, ok := visited[url]; ok {
                    continue
                } else {
                    visited[url] = true
                    go exploreURL(url, urlProcessor)
                }
            case <-time.After(1 * time.Second):
                fmt.Printf("Explored %d pages\n", len(visited))
                done <- true
            }
        }
    }     

We need a map to keep track of our visited urls. This map will only be accessed by this goroutine, so we don't have to worry about thread safety, like using a mutex.

In the infinite loop we have a select statement with two possibilities:

1. Read a url from the channel:
    * check if we've visited that url before: 
        - if so, just continue on.
        - if not, add it to the `visited` map and kick off the `exploreURL()` funcion in a goroutine

1. Read from the [`time.After()`](https://golang.org/pkg/time/#After) channel after one second of inactivity on the `urlProcessor` channel:
    * This allows send on our `done` channel, signaling the end of our web crawling.

### Look for links to explore on a URL

Let's take a look at the flow of the `exploreURL` function.

    resp, err := http.Get(url)
    // ...
    z := html.NewTokenizer(resp.Body)

Here we do an [http](https://golang.org/pkg/net/http/#pkg-overview) [Get](https://golang.org/pkg/net/http/#Client.Get) on the url and create a [tokenizer](https://godoc.org/golang.org/x/net/html) from the response body.

Now we need to loop through all of the tokens: 

    for {
		tt := z.Next()
		if tt == html.ErrorToken {
			return
		}

		if tt == html.StartTagToken {
			t := z.Token()

			if t.Data == "a" {
				for _, a := range t.Attr {
					if a.Key == "href" {

						// if link is within jeremywho.com
						if strings.HasPrefix(a.Val, "https://jeremywho.com") {
							urlProcessor <- a.Val
						}
					}
				}
			}
		}
	}  

As soon as an opening html `<a>` tag is found, we loop through that tags attributes, looking for `href` that contains the links url.
When we find one, we add the url to the `urlProcessor` channel so it can be explored.

Note that we are limiting our exploration to links that live within _jeremywho.com_.

![image of crawler output](/images/crawler.png "crawler output")