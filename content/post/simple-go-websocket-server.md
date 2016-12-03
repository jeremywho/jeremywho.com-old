+++
Description = ""
Tags = [
  "Development",
  "golang",
  "websocket",
  "go" 
]
Categories = [
  "Development",
  "GoLang",
]

date = "2016-12-02"
title = "Simple websocket server in Go using gorilla websocket"
highlight = true

+++

Make a simple websocket echo server using gorilla/websocket. 

<!--more-->

---

View the source code: https://gist.github.com/jeremywho/2bb5f9fd98b13b048ab7b8f5a0d2a68c

### Step 1 - Setup http server

Let's look at the main function

    func main() {
	  http.HandleFunc("/", echo)
	  http.ListenAndServe(":8080", nil)
    }

First we setup the route handler: `http.HandleFunc("/", echo)` 
This will send all requests to a function called `echo` (which we will take a look at in a moment)

Next is to start the http server: `http.ListenAndServe(":8080", nil)`
This will start an http server that listens on port `8080`

### Step 2 - Upgrade the http connection to a websocket connection

Take a look at the `echo` function:

    func echo(w http.ResponseWriter, r *http.Request) {
        upgrader := websocket.Upgrader{}
        conn, _ := upgrader.Upgrade(w, r, nil)
        defer conn.Close()

        for {
            mt, message, _ := conn.ReadMessage()

            fmt.Printf("recv: %s\n", message)

            _ = conn.WriteMessage(mt, message)
        }
    }

It is important to understand what the [`upgrader`](https://godoc.org/github.com/gorilla/websocket#Upgrader) does.  

    upgrader := websocket.Upgrader{}
    conn, _ := upgrader.Upgrade(w, r, nil)

The echo handler takes two parameters `w http.ResponseWriter` and `r *http.Request` but we don't want to do http operations, we want to do websocket stuff.  

So the [`upgrader`](https://godoc.org/github.com/gorilla/websocket#Upgrader) takes our http goodies and gives us back a [`websocket connection`](https://godoc.org/github.com/gorilla/websocket#Conn), thus _upgrading_ our http connection to a websocket.

### Step 3 - Loop forever

    for {
            mt, message, _ := conn.ReadMessage()
            // ...
            _ = conn.WriteMessage(mt, message)
        }

The last thing to do is to setup a loop that reads any data on the websocket and writes the data back to the connection.  
The [`ReadMessage`](https://godoc.org/github.com/gorilla/websocket#Conn.ReadMessage) function returns a [`messageType`](https://godoc.org/github.com/gorilla/websocket#pkg-constants) and a byte array that contains the data read.
The [`WriteMessage`](https://godoc.org/github.com/gorilla/websocket#Conn.WriteMessage) function takes a [`messageType`](https://godoc.org/github.com/gorilla/websocket#pkg-constants) and a byte array with the data to send.

You can also read and write JSON:
[`ReadJSON`](https://godoc.org/github.com/gorilla/websocket#Conn.ReadJSON)
[`WriteJSON`](https://godoc.org/github.com/gorilla/websocket#Conn.WriteJSON)