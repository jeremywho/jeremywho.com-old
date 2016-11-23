+++
Description = ""
Tags = [
  "Development",
  "golang",
]
Categories = [
  "Development",
  "GoLang",
]
menu = "main"
date = "2016-11-15T16:26:13-08:00"
title = "Parsing json in Go"
highlight = true

+++

Let's look at some simple ways to parse json in Golang, including one trick I was not aware of.

<!--more-->

<<<<<<< HEAD
---

### Parse simple json with map of empty interfaces

Sample json:

    { 
        "firstname": "Bob",
        "lastname": "Loblaw",
        "stat": "California"        
    }

declare map named `data`:

    var data map[string]interface{}

parse the json string:

    _ = json.Unmarshal([]byte(simpleJSONString), &data)

printing data `fmt.Println(data)` gives us:

    map[firstname:Bob lastname:Loblaw state:California]
    
and we can access elements `fmt.Println(data["lastname"])`

    Loblaw

Try this on the Go Playground: https://play.golang.org/p/frFgp1ajkM

---

### Parse more complex json with map of empty interfaces

Sample json:

    { 
        "firstname": "Bob",
        "lastname": "Loblaw",
        "state": "California",
        "phoneNumbers": [
          "555-333-3333",
          "555-333-3131"
        ]        
    }

declare map named `data`:

    var data map[string]interface{}

parse the json string:

    _ = json.Unmarshal([]byte(simpleJSONString), &data)

printing data `fmt.Println(data)` gives us:

    map[firstname:Bob lastname:Loblaw state:California phoneNumbers:[555-333-3333 555-333-3131]]
    
Now what if we want to access elements inside the array `fmt.Println(data["lastname"][0])`

    invalid operation: data["phoneNumbers"][0] (type interface {} does not support indexing)
=======
# h1 Heading
## h2 Heading
### h3 Heading
#### h4 Heading
##### h5 Heading
###### h6 Heading
##### h5 Heading
>>>>>>> 25389c4d5a9557e850ee879626174a378b3f3a7f

At this point `data["phoneNumbers"]` is an `interfact{}` that we need to convert to an array or slice of strings.
We are kind of stuck, unless we want to use reflection, something like this: https://play.golang.org/p/gQhCTiwPAq 

---

### Parse more complex json with defined types

Sample json:

    { 
        "firstname": "Bob",
        "lastname": "Loblaw",
        "state": "California",
        "phoneNumbers": [
          "555-333-3333",
          "555-333-3131"
        ]        
    }

This time we will define a type:

    type simpleData struct {
	  Firstname    string   `json:"firstname"`
	  Lastname     string   `json:"lastname"`
	  State        string   `json:"state"`
	  PhoneNumbers []string `json:"phoneNumbers"`
    }

create an empty `simpleData`:

    data := simpleData{}

parse the json string, same as before:

    _ = json.Unmarshal([]byte(simpleJSONString), &data)

now when we print data `fmt.Println(data)` we get:

    {Bob Loblaw California [555-333-3333 555-333-3131]}
    
And we can access elements inside the array `fmt.Println(data.PhoneNumbers[0])`

    555-333-3333

Try this out here: https://play.golang.org/p/NCIz6TH4bH

---

### The struct used to parse json doesn't need a field for every json key

Sample json:

    { 
        "firstname": "Bob",
        "lastname": "Loblaw",
        "state": "California",
        //... more items ...
        "phoneNumbers": [
          "555-333-3333",
          "555-333-3131"
        ]        
    }

We only care about `state` so we create a `struct` with just that:

    type simpleData struct {
	      State    string   `json:"state"`
    }

create an empty `simpleData`:

    data := simpleData{}

parse the json string, same as before:

    _ = json.Unmarshal([]byte(simpleJSONString), &data)

now when we print data `fmt.Println(data)` we get a struct with just the value of the State property:

    {California}
    
And we can access the state field `fmt.Println(data.State)`:

    California

Try this out here: https://play.golang.org/p/-5M35wMj1Y

### End of transmission