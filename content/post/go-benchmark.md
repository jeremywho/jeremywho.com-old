+++
Description = ""
Tags = [
  "Development",
  "golang",
  "testing",
  "go",
  "benchmark"
]
Categories = [
  "Development",
  "GoLang",
]

date = "2016-12-06"
title = "Simple Golang benchmark test using the built-in testing package"
highlight = true

+++

I was curious about the speed differences when parsing json, so I made a simple benchmark test to find out what's going on. 

<!--more-->

---

View the code here: https://github.com/jeremywho/goBenchmark

When [parsing json](/parse-json-go), I was curious if there was a difference in speed when using a struct with just the items you want vs a struct with lots of fields.

Compare the following two structs

    type simpleData struct {
        Name string `json:"name"`
    }  

vs

    type complexData struct {
        Id         string   `json:"_id"`
        Index      int      `json:"index"`
        Guid       string   `json:"guid"`
        IsActive   bool     `json:"isActive"`
        Balance    string   `json:"balance"`
        Picture    string   `json:"picture"`
        Age        int      `json:"age"`
        EyeColor   string   `json:"eyeColor"`
        Name       string   `json:"name"`
        Gender     string   `json:"gender"`
        Company    string   `json:"company"`
        Email      string   `json:"email"`
        Phone      string   `json:"phone"`
        Address    string   `json:"address"`
        About      string   `json:"about"`
        Registered string   `json:"registered"`
        Lat        int      `json:"latitude"`
        Long       int      `json:"longitude"`
        Tags       []string `json:"tags"`
        Greeting   string   `json:"greeting"`
        FavFruit   string   `json:"favoriteFruit"`
    }

 Using the [encoding/json](https://golang.org/pkg/encoding/json/) package, we can unmarshal into a type that contains just the fields we want.

 I'm curious if it might be faster to unmarhsal with a smaller struct, so I created a simple [benchmark](https://golang.org/pkg/testing/#hdr-Benchmarks) test.

 This test needs to read some data in from a large json file, so we create a [TestMain](https://golang.org/pkg/testing/#hdr-Main) function to make that happen:

    func TestMain(m *testing.M) {
        flag.Parse()

        inputData, _ = ioutil.ReadFile("input.json")

        os.Exit(m.Run())
    }    

Now that we have our json, we can create our benchmarks:

    func BenchmarkComplexJSONParse(b *testing.B) {
        for i := 0; i < b.N; i++ {
            _ = json.Unmarshal(inputData, &c)
        }
    }

and

    func BenchmarkSimpleJSONParse(b *testing.B) {
        for i := 0; i < b.N; i++ {
            _ = json.Unmarshal(inputData, &s)
        }
    }

To run the tests we can use:

    go test -bench=.

and get output that looks something like this:

    Jeremys-MacBook-Pro:goBenchmark jeremy$ go test -bench=.
    testing: warning: no tests to run
    BenchmarkComplexJSONParse-8   	    1000	   1425967 ns/op
    BenchmarkSimpleJSONParse-8    	    1000	   1232077 ns/op
    PASS
    ok  	github.com/jeremywho/goBenchmark	2.936s

Notice the warning `testing: warning: no tests to run`, which means we don't have any unit tests (`func TestXxx(*testing.T)`) in the file.