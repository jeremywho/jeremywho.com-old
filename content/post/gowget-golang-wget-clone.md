+++
Description = "gowget - wget clone written in golang"
Tags = [
  "Development",
  "golang",
  "go",
  "wget"   
]
Categories = [
  "Development",
  "GoLang",
]

date = "2016-12-08"
title = "gowget - Let's make wget in Go!"
highlight = true

+++

Fun little project to make a wget clone in Go.  Complete with a command line progress indicator!

![gif of gowget downloading](/images/gowget.gif "gowget downliading")

<!--more-->

---

_**Special thanks to [@GrimTheReaper](https://github.com/GrimTheReaper) for helping with the app idea!**_

View the code here: https://github.com/jeremywho/gowget

Ok, its not a full clone of wget.  But it lets you download a file and save the output!

### Command line progress indicator 
Let's start with the progress indicator:

    func printProgress(curr, total float64) {
        width := 40.0
        output := ""
        threshold := (curr / total) * float64(width)
        for i := 0.0; i < width; i++ {
            if i < threshold {
                output += "="
            } else {
                output += " "
            }
        }

        fmt.Printf("\r[%s] %.1f of %.1fMB", output, curr/bytesToMegaBytes, total/bytesToMegaBytes)
    }

The trickiest part here is the Printf with `"\r"` which is a carriage return. Often this is seen together with the newline character '\n'.  
The carriage return simply moves the cursor to the start of the line.  So if you combine it with the newline character, you get the cursor at the start of a new line.
If you use carriage return by itself, you get the cursor at the start of the same line, letting you overwrite what was previously written.

Other than that, we just determine the % complete, and then fill out the output string, using `'='` if less than the completed percentage and a space `' '` if greater than the completed percentage

### io.Reader progress

The next thing to check out is getting the completed progress from an [`io.Reader`](https://golang.org/pkg/io/#Reader)

This code was originally found on [StackOverflow](http://stackoverflow.com/a/22422650/613575) from user [jimt](http://stackoverflow.com/users/357705/jimt) 

The first thing we need is a struct to hold to data

    type PassThru struct {
        io.Reader
        curr  int64
        total float64
    }

We have an `io.Reader` [anonymous field](http://dennissuratna.com/using-anonymous-field-to-extend-in-go/), this gives the PassThru type the ability to be used like an [`io.Reader`](https://golang.org/pkg/io/#Reader)

Now we can "override" the stander io.Reader read() function:

    func (pt *PassThru) Read(p []byte) (int, error) {
        n, err := pt.Reader.Read(p)
        pt.curr += int64(n)

        // last read will have EOF err
        if err == nil || (err == io.EOF && n > 0) {
            printProgress(float64(pt.curr), pt.total)
        }

        return n, err
    }

Basically all this does is call the underlying Reader.Read() function and then we add the number of bytes read into a counter.

This is a really cool technique to add your own functionality to existing code.

### Build and download!

I used this to build mine:

    go build -o wget main.go

Now I can download things!!

    ./wget http://ipv4.download.thinkbroadband.com/5MB.zip output.zip

![gif of gowget downloading](/images/gowget.gif "gowget downliading")

