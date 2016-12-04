+++
Description = ""
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

date = "2016-12-03"
title = "Simple Golang unit-testing using the built-in testing package"
highlight = true

+++

Let's setup a unit test for our rad package!

<!--more-->

---

View the code here: http://github.com/jeremywho/gosimpletesting

You can grab the code if you want to try it out by running:

    go get github.com/jeremywho/gosimpletesting && cd $GOPATH/src/github.com/jeremywho/gosimpletesting

### Create the package we want to test

We create a simple go package we can test called `myradpackage.go`

    package myradpackage

    import (
      "strings"
    )

    func GetProtocol(url string) string {
        return strings.Split(url, ":")[0]
    }

Here we have a simple package `myradpackage` that contains one function `GetProtocol`
This function takes a string containing a url and returns a string containing the protocol of the url.

Now lets test it!

### Setup our unit test

Create a test file called `myradpackage_test.go`

    package myradpackage_test

    import (
      "testing"

      "github.com/jeremywho/gosimpletesting"
    )

    func TestMyRadPackage(t *testing.T) {
      if myradpackage.GetProtocol("http://jeremywho.com") != "http" {
        t.Error("Expected http")
      }
      if myradpackage.GetProtocol("ws://jeremywho.com") != "ws" {
        t.Error("Expected ws")
      }
    }

Here we import the testing package and the rad package we want to test. 

Next is the test function `func TestMyRadPackage(t *testing.T)`

Inside this function we add our actual tests.  


### Run the test

To run our tests we just use `go test` from the directory containing the files.  
We should see:

    Jeremys-MacBook-Pro:gosimpletesting jeremy$ go test
    PASS
    ok     	github.com/jeremywho/gosimpletesting   	0.006s

A failed test would look something like this:

    Jeremys-MacBook-Pro:gosimpletesting jeremy$ go test
    --- FAIL: TestMyRadPackage (0.00s)
            myradpackage_test.go:11: Expected http
    FAIL
    exit status 1
    FAIL   	github.com/jeremywho/gosimpletesting   	0.006s

