---
layout: post
title: "A Quick Guide to Using docker-compose (previously fig)"
date: 2015-02-26 07:43:59 -0500
comments: true
categories: docker linux
---

Have you used [Docker](https://www.docker.com/) yet? Docker is awesome! Docker is a tool for managing isolated system environments. I have started using Docker for __everything__. This is not [the first time I've written about this delightful tool](/blog/2015/01/11/an-example-use-case-for-docker).

Though the ideas and implementation behind Docker are really cool, the command line syntax is extremely cumbersome. There used to be this great third-party tool to address this problem called [fig](http://www.fig.sh/). But, as of [yesterday](https://github.com/docker/compose/releases/tag/1.1.0), it looks like Docker is maintaining `fig` and have renamed it to `docker-compose`. The impact of this change is minimal on developers - rename `fig.yml` files to `docker-compose.yml`, use `docker-compose up` instead of `fig up`. Some great features have been added that I've been sorely missing, including the ability to use environment files and only grabbing the latest tagged images from [the hub](https://registry.hub.docker.com/).

Let's learn you some `docker-compose`.

### Installation

Two ways: `pip install docker-compose` (if you're into that kind of thing) or through a questionable bash script:

``` bash
$ curl -L https://github.com/docker/compose/releases/download/1.1.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
$ chmod +x /usr/local/bin/docker-compose
```

You probably want bash completion while you're at it:

``` bash
$ curl -L https://raw.githubusercontent.com/docker/compose/1.1.0/contrib/completion/bash/docker-compose > /etc/bash_completion.d/docker-compose
```

### Compose, World

Let's say I have a simple Go application which only prints the obligatory "Hello, World":

``` go simple-golang-app/main.go
package main

import "fmt"

func main() {
  fmt.Println("Hello, World")
}
```

I want to run this with `docker-compose`, so I create the following file:

``` yaml docker-compose.yml
app:
  image: golang:1.4
  working_dir: /go/src/simple-golang-app
  command: go run main.go
  volumes:
    - ./simple-golang-app:/go/src/simple-golang-app
```

This file specifies using the `golang:1.4` image as the base image. It maps the our code directory to a directory in the `$GOPATH` in the container. It then specifies running the command `go run main.go` from the working directory of `/go/src/simple-golang-app`. So let's run it:

``` bash
$ docker-compose up
Creating dockercomposeexample_app_1...
Attaching to dockercomposeexample_app_1
app_1 | Hello, World
dockercomposeexample_app_1 exited with code 0
Gracefully stopping... (press Ctrl+C again to force)
```

### Composing a Database Connection

Now let's say I want to connect to a database and find some information. The following example connects to a Mongo database, inserts a record, and spits out all the existing records:

``` go database-golang-app/main.go
package main

import (
  "fmt"
  "gopkg.in/mgo.v2"
  "gopkg.in/mgo.v2/bson"
  "os"
  "time"
)

type Ping struct {
  Id   bson.ObjectId `bson:"_id"`
  Time time.Time     `bson:"time"`
}

func main() {
  // get the session using information from environment, ignore errors
  session, _ := mgo.Dial(os.Getenv("DATABASE_PORT_27017_TCP_ADDR"))
  db := session.DB(os.Getenv("DB_NAME"))
  defer session.Close()

  // insert new record
  ping := Ping{
    Id:   bson.NewObjectId(),
    Time: time.Now(),
  }
  db.C("pings").Insert(ping)

  // get all records
  pings := []Ping{}
  db.C("pings").Find(nil).All(&pings)

  fmt.Println(pings)
}
```

This application has an external dependency on `mgo`, which means I'll need to have that dependency installed somehow. I'll make a `Dockerfile` to live alongside this example:

``` Dockerfile database-golang-app/Dockerfile
FROM golang:1.4

RUN go get gopkg.in/mgo.v2
```

Now I'll construct a yaml file for `docker-compose` to consume. This time, we need to `build` the main app from our `Dockerfile`, add in a second container for the database, add a `link` between our application and the database, and specify an `environment` variable.

``` yaml docker-compose.yaml
advanced:
  build: ./database-golang-app
  working_dir: /go/src/database-golang-app
  command: go run main.go
  volumes:
    - ./database-golang-app:/go/src/database-golang-app
  links:
    - database
  environment:
    - DB_NAME=advanced-golang-db
database:
  image: mongo:3.0
  command: mongod --smallfiles --quiet --logpath=/dev/null
```

Since `mongod` has a tendency to spit up a big boilerplate on initialization, I redirect its output to `/dev/null` for this example. Let's see what happens when we compose:

``` bash
$ docker-compose up
Creating dockercomposeexample_database_1...
Creating dockercomposeexample_advanced_1...
Attaching to dockercomposeexample_database_1, dockercomposeexample_advanced_1
advanced_1 | [{ObjectIdHex("54efd8f889120d000d000001") 2015-02-27 02:39:52.53 +0000 UTC}]
dockercomposeexample_advanced_1 exited with code 0
Gracefully stopping... (press Ctrl+C again to force)
Stopping dockercomposeexample_database_1...
```

If you squint, you can see the output next to the `advanced_1` tag. We printed out one item from our database - the one we just added. What if we run it again?

``` bash
$ docker-compose up
...
advanced_1 | [{ObjectIdHex("54efd8f889120d000d000001") 2015-02-27 02:39:52.53 +0000 UTC} {ObjectIdHex("54efdbb936cc3f000e000001") 2015-02-27 02:51:37.879 +0000 UTC}]
...
```

This output proves that we're fetching from the database each time in addition to adding new records. Since the Mongo `Dockerfile` we used to build the image creates a `volume`, we get to see the data persisted.

To achieve the same goal with the Docker CLI, I would need to do this:

``` bash
$ docker run -d --name database mongo:3.0
$ docker build -t database-golang-app ./database-golang-app
$ docker run --link database:database -e DB_NAME=advanced-golang-db \
    -v /home/lrp/Projects/2015/docker-compose-example/database-golang-app:/go/src/database-golang-app \
     database-golang-app go run /go/src/database-golang-app/main.go
```

Keeping that much information at your fingertips all at once absolutely _hurts_. Making that line reproducible means copy/pasting it into/out of a README somewhere, or running some "single-line script" which only the one guy on the team is allowed to touch. `docker-compose` fixes these issues and makes `docker` a joy to use.

_You can find all of the source code for this blog post on Github: [https://github.com/larryprice/docker-compose-example](https://github.com/larryprice/docker-compose-example)._
