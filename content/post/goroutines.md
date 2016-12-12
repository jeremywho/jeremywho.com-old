+++
Description = "Let's look at goroutines"
Tags = [
  "Development",
  "golang",
  "go",
  "goroutine"
]
Categories = [
  "Development",
  "GoLang",
]

date = "2016-12-11"
title = "Let's look at goroutines"
highlight = true

+++

One of the flagship features of Go are goroutines. Let's take a look at what they are.

<!--more-->

---

View the code here: https://gist.github.com/jeremywho/4aa3f0f9854df0339a49f6b5760beb4f

### What are goroutines?

Go is known for handling concurrency well and [goroutines](https://golang.org/ref/spec#Go_statements) are at the heart of this.
Goroutines are lightweight threads that allow sections of code to be run concurrently.

#### Use with existing functions

Take a look at this silly program:

    func PrintGo() {
        for {
            fmt.Println("go")
            time.Sleep(1 * time.Second)
        }
    }

    func main() {
        go PrintGo()
        time.Sleep(10 * time.Second)
    }

You can try this out [here](https://play.golang.org/p/1skF0aOQvL)

There is a function `PrintGo()` that is executed with the keyword [`go`](https://golang.org/ref/spec#Go_statements) in front of it:

    go PrintGo()

Inside the function, it loops forever, printing "go" and then sleeping for one second.

After the `PrintGo` goroutine is started, we have another timer in the `main` function that sleeps for 10 seconds, we'll take a look at why this is used in just a bit.

#### Use with anonymous functions

Goroutines can be used with anonymous functions, letting us kick off small code blocks that can run concurrently.

The same app before, using an anonymous function:

func main() {
	go func() {
		for {
			fmt.Println("go")
			time.Sleep(1 * time.Second)
		}
	}()
	
	time.Sleep(10 * time.Second)
}

Try this out [here](https://play.golang.org/p/bgq10cnMxW)

This can be a little confusing at first, lets break it down.

We use the `go` keyword, just like the previous example to start a code block running concurrently.

Then we tell the go routine that we have a function without a name (hence, anonymous):

     func() {
         //same loop as before
     }

Lastly, we tell the anonymous function to begin executing now by adding the parenthesis `()` at the end:

     func() {
         //same loop as before
     }()

### How can we wait for goroutines to finish?

We need to keep our main function alive long enough for the concurrent goroutines to finish their work.  Let's look at some ways to make this happen.

#### Option 1: Have something that lasts longer

This is *not* a very good solution, but does work. 

We use this method in the example above by having a timer that lasts for 10 seconds, which lets the goroutine run for 10 seconds before the program exits.

#### Option 2: Use a channel

Try it [here](https://play.golang.org/p/1pMB1IunkT)

    func PrintGo(done chan bool) {
        for i := 0; i < 10; i++ {
            fmt.Println("go")
            time.Sleep(1 * time.Second)
        }
        done <- true
    }

    func main() {
        done := make(chan bool)
        go PrintGo(done)
        <-done
    }

This time we do something a bit different.

we create a [channel](https://golang.org/ref/spec#Channel_types) `done` that holds boolean values and pass that channel to our `PrintGo` goroutine:

    done := make(chan bool)
    go PrintGo(done)

Then we try to read from the done channel:

    <-done

This channel read will block until there is some data on the channel, which lets us keep our program running until something shows up on the `done` channel.  

The last thing to observe is that the `PrintGo` function now loops 10 times and then sends `true` to the `done` channel.

So the overall flow is to start the concurrent goroutine, then block the main portion of the program by trying to read from the `done` channel. The goroutine will do its work concurrently  and then signal it's done by passing `true` to the `done` channel, allowing the main function to read from the channel and end the program. 

#### Option 3: use a WaitGroup

Try it [here](https://play.golang.org/p/OdQkFrAedi)

The last way is to use a [`WaitGroup`](https://golang.org/pkg/sync/#WaitGroup) from the [`sync`](https://golang.org/pkg/sync/) package.

Instead of channels, we create a WaitGroup and pass that to the goroutine function:

    func main() {
        var wg sync.WaitGroup
        wg.Add(1)

        go PrintGo(&wg)

        wg.Wait()
    }

Here we do [`wg.Add(1)`](https://golang.org/pkg/sync/#WaitGroup.Add) because we have one _thing_ we are waiting for. Then we just call [`wg.Wait()`](https://golang.org/pkg/sync/#WaitGroup.Wait) to wait for it to finish.

In the function created by the coroutine, we pass in the [`WaitGroup`](https://golang.org/pkg/sync/#WaitGroup) and then call `defer` on [`wg.Done()`](https://golang.org/pkg/sync/#WaitGroup.Done):

    func PrintGo(wg *sync.WaitGroup) {
        defer wg.Done()
        // for loop from previous examples
    }

WaitGroups are especially useful when dealing with more than one goroutine:

Try it [here](https://play.golang.org/p/pprSlQb2W7)

    func main() {
        var wg sync.WaitGroup
        wg.Add(2)

        go PrintGo(&wg)
        go PrintIsRad(&wg)

        wg.Wait()
    }

