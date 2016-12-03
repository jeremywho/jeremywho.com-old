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

date = "2016-12-01"
title = "Simple websocket client in Go using gorilla websocket"
highlight = true

+++

Make a simple websocket client using gorilla/websocket. Typically echo examples are saved for servers, 
but in this case, we are going to create a client that keeps sending a timestamp every n seconds to an echo server at http://www.websocket.org/echo.html

<!--more-->

---

View the source code: https://gist.github.com/jeremywho/c101d3bfdfff23ab0443477ef6e600af

**Please note that this sample app has no error handling**

### Step 1 - dial the server

    URL := "ws://echo.websocket.org/"

	var dialer *websocket.Dialer

	conn, _, err := dialer.Dial(URL, nil)
	if err != nil {
		fmt.Println(err)
		return
	}

This gives us back a [`*websocket.Conn`](https://godoc.org/github.com/gorilla/websocket#Conn) that we can use for sending and receiving messages.

Note the `ws://` protocol identifier, this is what is typicaly used when connecting via websockets, `wss://` also exists for a secure websocket connection (think `https://`)


### Step 2 - Start the datetime writer

    func timeWriter(conn *websocket.Conn) {
	  for {
	    time.Sleep(time.Second * 2)
		conn.WriteMessage(websocket.TextMessage, []byte(time.Now().Format("2006-01-02 15:04:05")))
	  }
    }

Here we have a function that takes a [`*websocket.Conn`](https://godoc.org/github.com/gorilla/websocket#Conn) and loops forever.
In the loop we sleep for n seconds (2 seconds in our example) and then writes some data to the connection. 

The [`WriteMessage`](https://godoc.org/github.com/gorilla/websocket#Conn.WriteMessage) function takes 2 parameters: a [`messageType`](https://godoc.org/github.com/gorilla/websocket#pkg-constants) and a byte array that contains the data we want to send.

    go timeWriter(conn)

The function is started in a [`goroutine`](https://gobyexample.com/goroutines).

Some helpful info:
[`WriteMessage`](https://godoc.org/github.com/gorilla/websocket#Conn.WriteMessage) and 
[`WriteJSON`](https://godoc.org/github.com/gorilla/websocket#Conn.WriteJSON)

### Step 3 - Start the websocket reader

    for {
		_, message, err := conn.ReadMessage()
		if err != nil {
			fmt.Println("read:", err)
			return
		}

		fmt.Printf("received: %s\n", message)
	}

If we take out the error handling we have:

    for {
		_, message, _ := conn.ReadMessage()
		fmt.Printf("received: %s\n", message)
	}

This section loops forever, reading from the [`*websocket.Conn`](https://godoc.org/github.com/gorilla/websocket#Conn) and printing what was read.

Some helpful info:
[`ReadMessage`](https://godoc.org/github.com/gorilla/websocket#Conn.ReadMessage) and 
[`ReadJSON`](https://godoc.org/github.com/gorilla/websocket#Conn.ReadJSON)




