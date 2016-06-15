+++
date = "2016-06-16T07:47:11+12:00"
description = "A common pattern using Go interfaces"
draft = false
tags = ["golang", "interfaces", "patterns"]
title = "Public interface, private struct"
topics = []

+++

I have seen the following pattern used a lot, and used it a bit myself.  It's goal
is information hiding, allowing only those methods you allow to be used publicly.

<!--more-->

```go

type Thing interface {
  Name() string
}

func NewThing(name string) Thing {
  return &thing{name}
}

type thing struct {
  name string
}

func (t *thing) Name() string {
  return t.name
}

```

So in order you have:

1. the interface
2. the factory method
3. the struct or object (unexported)
4. a method on the object (unexported due to struct being unexported)

Now imagine that you are using this package.  The only thing you know about is that `NewThing()` returns a `Thing` that
can only do `Name()`.  You don't know about any of the internals of the package.  This is good because it decreases
coupling and keeps code deterministic because you know that no one is fucking with the internals of your object.  The
object is then in complete control of it's internals, and is the gatekeeper of any data going in or or out, making it a
black box.  The only problem is that it increases the amount of code you have to write by a little bit, but it's definitely
worth it to avoid tight coupling.

You can add methods to your struct and any methods you want the outside world to use get added to the interface. You might
even implement other interfaces, for example to return JSON:

```go

import "json"

type Thing interface {
  json.Marshaller

  Name()
}

// implement the json.Marshaller interface
func (t *thing) Marshal() ([]byte, error) {
  data := struct{N string `json:"name"`}{t.name}
  return json.Marshal(data)
}

```

### Ugly docling

One of the problems with this approach is that the docs on your object methods will not appear in the godocs because the
struct is not exported.  You will need to add comments above the interface methods for that.  I find this annoying because
then you do not get the individual HTML formatted explanation blocks, but rather the docs are just in the code block that
describes the interface.

See this [Semaphore](https://godoc.org/github.com/dropbox/godropbox/sync2#Semaphore) doc for an idea compared with the doc
for [BaseConnectionPool](https://godoc.org/github.com/dropbox/godropbox/net2#BaseConnectionPool).  All the methods on the
latter are laid out and nicely formatted.  But thats a tooling problem anyway.
