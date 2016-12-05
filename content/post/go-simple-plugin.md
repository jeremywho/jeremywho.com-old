+++
Description = "Go 1.8 - Plugins!"
Tags = [
  "Development",
  "golang",
  "go1.8",
  "go",
  "plugin"
]
Categories = [
  "Development",
  "GoLang",
]

date = "2016-12-05"
title = "Go 1.8 - Plugins!"
highlight = true

+++

Go 1.8 gives us a new tool for creating shared libraries, called [plugins](https://beta.golang.org/pkg/plugin/)!
Let's take a look at creating and using one.

<!--more-->

---

**Please note that this sample app has no error handling**

**Plugins are a feature available starting in Go 1.8, this post was created using [Go 1.8beta](https://golang.org/dl/#go1.8beta1)**

_**Special thanks to [@AWildTyphlosion](https://twitter.com/AWildTyphlosion) for helping with the initial sample app and a docker container running go 1.8beta1!**_ 

View the code here: https://github.com/jeremywho/pluginTest

### Step 0 - Setup Go 1.8beta1 in a docker container

With docker installed, create a directory with a Dockerfile that contains:

    from golang:1.8beta1-wheezy

    CMD ["bash"]

Inside that directory run the following:

    docker build -t go1.8 . && docker run -it go1.8

You should now be at the command prompt inside your container:

    root@cc5dae79d6fe:/go#

Then run:

    go get github.com/jeremywho/pluginTest && cd $GOPATH/src/github.com/jeremywho/pluginTest

You should now be ready to follow along!

### Step 1 - Create the plugin

Our plugin:

    package main

    func Add(x, y int) int {
        return x+y
    }

A simple method to add to integers. That's it for this example.  

### Step 2 - Build the plugin

According to the [documentation](https://beta.golang.org/pkg/plugin/#pkg-overview) we can build plugins with:
    
    go build -buildmode=plugin

We are going to be a bit more verbose with our build command:

    go build -buildmode=plugin -o myplugin.so myplugin.go

Let's specify the output file with `-o myplugin.so` and the input file at the end of the command

If you take a look in your directory, you should see the `myplugin.so` library:

    root@cc5dae79d6fe:/go/src/github.com/jeremywho/pluginTest# ls
    main.go  myplugin.go  myplugin.so

Now that we have a shared library, we can...

### Step 3 - Load the plugin

Make sure the `plugin` package is imported.

Load the shared library file we built:

    p, _ := plugin.Open("./myplugin.so")  

We call [`plugin.Open()`](https://beta.golang.org/pkg/plugin/#Open) with the path to the shared library file we created in the previous step.
This gives us back a pointer to a [`Plugin`](https://beta.golang.org/pkg/plugin/#Plugin) type.

Next we call [`plugin.Lookup()`](https://beta.golang.org/pkg/plugin/#Plugin.Lookup) with the name of the [`Symbol`](https://beta.golang.org/pkg/plugin/#Symbol) we want to use:

    add, _ := p.Lookup("Add")

A [`Symbol`](https://beta.golang.org/pkg/plugin/#Symbol) is can be "any exported variable or function".

### Step 4 - Use it!

Here is how we call your plugin method:

    sum := add.(func(int, int) int)(1, 2)

This was confusing to me at first, let's break it down.

The original plugin function signature looks like:

    func Add(x, y int) int

Which is a function that takes two integers `func(int,int)` and returns an integer:
    
    func(int,int) int

We do a [type assertion](https://tour.golang.org/methods/15) (think cast) on our [`Symbol`](https://beta.golang.org/pkg/plugin/#Symbol) `add`
    
    add.(func(int, int) int)

which tells _Go_ that add is a function with that signature.

Lastly we call it with the parameters we want to add `(1,2)` and store the results in `sum`

    sum := add.(func(int, int) int)(1, 2)

Thats it!  We can run our program and see the results:

    root@cc5dae79d6fe:/go/src/github.com/jeremywho/pluginTest# go run main.go
    3    

### Note

Something that was pointed out by [@AWildTyphlosion](https://twitter.com/AWildTyphlosion) is the size of binary plugin files:

    -rw-r--r-- 1 root root     198 Dec  5 20:14 main.go
    -rw-r--r-- 1 root root      56 Dec  5 20:00 myplugin.go
    -rw-r--r-- 1 root root 1616361 Dec  5 20:14 myplugin.so

1.5 MB for a 4 line plugin!

I'm not sure why this is.  This was built on a beta version of Go, maybe that will change for the final release.
