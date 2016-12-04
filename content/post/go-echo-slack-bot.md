+++
Description = "Let's make a slack bot in Go - Simple echo bot"
Tags = [
  "Development",
  "golang",
  "testing",
  "go",
  "unit testing"   
]
Categories = [
  "Development",
  "GoLang",
]

date = "2016-12-04"
title = "Let's make a slack bot in Go"
highlight = true

+++

Slack bots are so hot right now. 
Let's see what it takes to create a simple one in Go.

<!--more-->

---

View the code here: https://gist.github.com/jeremywho/4aa3f0f9854df0339a49f6b5760beb4f

### Step 0 - What do you need?

1. A slack [team](https://slack.com/create#email).
2. A slack bot [user](https://my.slack.com/services/new/bot)
3. The API token for the bot user you created (looks like _"xoxb-...."_)

### Step 1 - Get the websocket url 

Slack has an api call that begins what they refer to as the Real Time Messaging API ([RTM](https://api.slack.com/rtm)).  
Sounds pretty serious, right?  All this means is we can talk to the slack api over a websocket. 

To connect via websockets, we need to get the websocket url, which is done via the slack [rtm.slack](https://api.slack.com/methods/rtm.start) api call.

This call is simple, do an http GET to this url 

    https://slack.com/api/rtm.start?token=xoxb-xxxx 

Putting your bot token in the url
    
    token=xoxb-xxxx... 

and get a bunch of json back.  

Out of all the json we get back, we just need one thing, the value of the `"api"` key, which contains our websocket url!

The sample code has a nice little function to do all of this for you

    type authResponse struct {
        URL string `json:"url"`
    }

    func getWebsocketURL(token string) string {
        url := "https://slack.com/api/rtm.start?token=" + token
        resp, err := http.Get(url)
        if err != nil {
            fmt.Println(err)
        }

        defer resp.Body.Close()
        body, err := ioutil.ReadAll(resp.Body)

        res := authResponse{}
        json.Unmarshal([]byte(body), &res)
        return res.URL
    } 

Here is a sample I got recently:

    wss://mpmulti-n0i3.slack-msgs.com/websocket/rbB625hiLXuF4e91xEq79fnpzTSDKT-hRDyJ1i62lnjdmQnSYQxV2YR2HpWSAqM2ugClq2fNkAGjMCGZM0BUcE0LCrdXW6aqoE1lu0zJB-RQLnrR3Z7H84ZIx49iFMNK3hCburw4TZvTj6Pc__hvvA==

_Note that the websocket connection is using secure websocket connection (wss://)_     

### Step 2 - Connect to the slack via websockets

If you're curious about websockets in Go, take a look at my posts for a simple websocket [client](/simple-websocket-client-in-go-using-gorilla-websocket) and [server](/simple-websocket-server-in-go-using-gorilla-websocket)

Now that we have the websocket url, we need to dial it. (This example uses [gorilla/websocket](https://github.com/gorilla/websocket))

	URL := getWebsocketURL(token)
	var dialer *websocket.Dialer
	conn, _, _ := dialer.Dial(URL, nil)

This gives us back a [websocket connection](https://godoc.org/github.com/gorilla/websocket#Conn) that lets us talk to slack.

### Step 3 - Loop forever

The last step is to setup a loop that reads from the connection and writes back what it read.

A condensed version looks like this

    m := message{}
    err := conn.ReadJSON(&m)

    echo := message{
        ID:      "1",
        Type:    m.Type,
        Channel: m.Channel,
        Text:    m.Text,
    }

    conn.WriteJSON(echo)

Here we 

1. read the json data from the websocket 
2. build a response message 
3. send the response

### Notes

##### Note 1

We created a `message` struct that is used for serializing/deserializing the json data:

    type message struct {
        ID string `json:"id"`
        ReplyTo string `json:"reply_to"`
        Type    string `json:"type"`
        Channel string `json:"channel"`
        Text string `json:"text"`
    }

We are just doing a simple example here, there is [more](https://api.slack.com/events/message) to this message.

##### Note 2

In the loop, we are doing a check before sending our echo response

    if m.Type == "message" && m.ReplyTo != "1" { 
        // send reply
    }

This is simply to allow us to echo only direct messages sent by users, we aren't echo-ing system message, etc.

##### More info

Take a look at the official [RTM](https://api.slack.com/rtm) documentation to get some ideas on where to go from here.