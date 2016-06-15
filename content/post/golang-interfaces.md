+++
date = "2016-06-15T19:03:04+12:00"
description = "Decreasing coupling and enabling testing"
draft = false
tags = ["golang", "interfaces"]
title = "Golang interfaces"

+++

So I've been learning Golang for work and have been mucking around with it for a while now.  I like it a lot, it feels
like a scripting language with compiled language extensions.  If you don't use interfaces though, you're doing it wrong.
It took me a while to understand how useful interfaces are in Golang, but I think I get it now.

<!--more-->

In general, interfaces:

* are a discrete set of methods that an object provides
* allow you to transparently switch types when passing in and out of methods
* don’t need to cover all methods of an object
* need objects to implement all methods of an interface to satisfy it
* make testing way easier when used to inject dependencies
* make testing way easier because it allows for faking and mocking
* can be a pointer or not
* can be nil


Lets look at this example:

```go

type MissileFirer interface {
  FireMissiles()
}

type Helicopter interface {
  Hover()
}

type Jet interface {
  StartJetEngines()
}

type AttackHelicopter interface {
  MissileFirer
  Hover
}

type JetFighter interface {
  MissileFirer
  Jet
}

// this is a MissileFirer, Jet and a JetFighter
type F14Tomcat struct {}
func (jet *F14Tomcat) FireMissiles() {}
func (jet *F14Tomcat) StartJetEngines() {}

// this is a MissileFirer, Helicopter and an AttackHelicopter
type AH64Apache struct {}
func (chopper *AH64Apache) FireMissiles() {}
func (chopper *AH64Apache) Hover() {}

// this is only a MissileFirer
type SAMSite struct {}
func (sam *SAMSite) FireMissiles() {}

// accept any interface that implements MissileFirer
func fireMissiles(hostile MissileFirer) {
  hostile.FireMissiles()
}

func main() {
  // return the structs as interfaces
  f14 := JetFighter(&F14Tomcat{})
  ah64 := AttackHelicopter(&AH64Apache{})

  fireMissiles(f14)
  fireMissiles(ah64)

  sam := &SAMSite{}

  // this will error because sam is a struct, even though it has FireMissiles()
  //fireMissiles(sam)

  // this works because we are passing SAM as an interface
  fireMissiles(MissileFirer(sam))

  // this won't work because SAM is not an AttackHelicopter - doesn't have Hover()
  // fireMissiles(AttackHelicopter(sam))
}

```

One thing I like to do is when I'm getting an API object, but only using the GET method, I can narrow the scope to just that
method:

```go

type apiGetter interface {
  Get(string) []byte
}

func GetTheThing(api apiGetter) []byte {
  return api.Get("/stuff")
}

```

Then I don't need to import an external API package, because all I need is something that satisfies that interface.
