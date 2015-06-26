---
layout: post
title: "Architecture for a Golang Web App"
date: 2015-06-25 07:24:27 -0400
comments: true
categories: golang web
---

Setting up a web application in Golang can be a daunting task. If you're used to rails, you probably miss having your hand held at this point. I'm using this architecture on [Refer Madness](https://www.refer-madness.com) [[source](https://github.com/larryprice/refermadness)]. Here's my directory structure:

```
-public/
-views/
-models/
-utils/
-controllers/
-web/
-main.go
```

I'll break this down architecturally. At each directory level, files are only allowed to access code from directories at the same or a higher level. This means that `utils` can access itself and `models`; `web` can access itself, `controllers`, `utils`, and `models`; and `models` can only access itself. I do this to keep a thick line between all the layers and to avoid recursive dependencies. Let's talk about each level individually!

#### main.go

`main.go` is my main file in charge of creating dependencies, discovering environment variables, and starting the web server. Here's a sample:

``` go main.go
package main

import (
  "github.com/larryprice/refermadness/utils"
  "github.com/larryprice/refermadness/web"
  "github.com/stretchr/graceful"
  "os"
)

func main() {
  isDevelopment := os.Getenv("ENVIRONMENT") == "development"
  dbURL := os.Getenv("MONGOLAB_URI")
  if isDevelopment {
    dbURL = os.Getenv("DB_PORT_27017_TCP_ADDR")
  }

  dbAccessor := utils.NewDatabaseAccessor(dbURL, os.Getenv("DATABASE_NAME"), 0)
  cuAccessor := utils.NewCurrentUserAccessor(1)
  s := web.NewServer(*dbAccessor, *cuAccessor, os.Getenv("GOOGLE_OAUTH2_CLIENT_ID"),
    os.Getenv("GOOGLE_OAUTH2_CLIENT_SECRET"), os.Getenv("SESSION_SECRET"),
    isDevelopment, os.Getenv("GOOGLE_ANALYTICS_KEY"))

  port := os.Getenv("PORT")
  if port == "" {
    port = "3000"
  }

  graceful.Run(":"+port, 0, s)
}
```

Since `main.go` is on the lowest level, it has access to every other directory: in this case, `utils` and `web`. I grab all the environment variables and inject them where necessary. I create the server, injecting dependencies, and then start it on the port I want.

#### web

I keep my server code in the `web` directory, which includes some web middleware. Here's the internal structure of the `web` directory:

```
-web/
|-middleware/
|-server.go
```

`server.go` contains the definition of my web server. The web server creates all my web controllers and adds middleware to my negroni server. Here's an excerpt:

``` go web/server.go
package web

import (
  "github.com/codegangsta/negroni"
  "github.com/goincremental/negroni-sessions"
  "github.com/goincremental/negroni-sessions/cookiestore"
  "github.com/gorilla/mux"
  "github.com/larryprice/refermadness/controllers"
  "github.com/larryprice/refermadness/utils"
  "github.com/larryprice/refermadness/web/middleware"
  "github.com/unrolled/secure"
  "gopkg.in/unrolled/render.v1"
  "html/template"
  "net/http"
)

type Server struct {
  *negroni.Negroni
}

func NewServer(dba utils.DatabaseAccessor, cua utils.CurrentUserAccessor, clientID, clientSecret,
  sessionSecret string, isDevelopment bool, gaKey string) *Server {
  s := Server{negroni.Classic()}
  session := utils.NewSessionManager()
  basePage := utils.NewBasePageCreator(cua, gaKey)
  renderer := render.New()

  router := mux.NewRouter()

  // ...

  accountController := controllers.NewAccountController(clientID, clientSecret, isDevelopment, session, dba, cua, basePage, renderer)
  accountController.Register(router)

  // ...

  s.Use(sessions.Sessions("refermadness", cookiestore.New([]byte(sessionSecret))))
  s.Use(middleware.NewAuthenticator(dba, session, cua).Middleware())
  s.UseHandler(router)
  return &s
}
```

A `server` struct is a `negroni.Negroni` web server. I create some dependencies from `utils` and external packages, create a router, create controllers with the dependencies, and register the router with my controllers. I also add in any middleware I may need. Speaking of middleware, here's an example:

``` go web/middleware/database.go
package middleware

import (
  "github.com/codegangsta/negroni"
  "github.com/larryprice/refermadness/utils"
  "net/http"
)

type Database struct {
  da utils.DatabaseAccessor
}

func NewDatabase(da utils.DatabaseAccessor) *Database {
  return &Database{da}
}

func (d *Database) Middleware() negroni.HandlerFunc {
  return func(rw http.ResponseWriter, r *http.Request, next http.HandlerFunc) {
    reqSession := d.da.Clone()
    defer reqSession.Close()
    d.da.Set(r, reqSession)
    next(rw, r)
  }
}
```

This is a standard middleware definition for accessing a database session from an HTTP route. The base of this file was taken from [Brian Gesiak's blog post on RESTful Go](http://modocache.svbtle.com/restful-go) and was modifiedto fit my needs.

#### controllers/

A controller is similar to the controller concept found in Rails. A controller registers its own route with the router provided and then sets up function calls. An abbreviated example:

``` go controllers/service_controller.go
package controllers

import (
  "encoding/json"
  "errors"
  "github.com/gorilla/mux"
  "github.com/larryprice/refermadness/models"
  "github.com/larryprice/refermadness/utils"
  "gopkg.in/mgo.v2/bson"
  "gopkg.in/unrolled/render.v1"
  "html/template"
  "net/http"
  "strings"
)

type ServiceControllerImpl struct {
  currentUser utils.CurrentUserAccessor
  basePage    utils.BasePageCreator
  renderer    *render.Render
  database    utils.DatabaseAccessor
}

func NewServiceController(currentUser utils.CurrentUserAccessor, basePage utils.BasePageCreator,
  renderer *render.Render, database utils.DatabaseAccessor) *ServiceControllerImpl {
  return &ServiceControllerImpl{
    currentUser: currentUser,
    basePage:    basePage,
    renderer:    renderer,
    database:    database,
  }
}

func (sc *ServiceControllerImpl) Register(router *mux.Router) {
  router.HandleFunc("/service/{id}", sc.single)
  // ...
}

// ...

type serviceResult struct {
  *models.Service
  RandomCode *models.ReferralCode
  UserCode   *models.ReferralCode
}

type servicePage struct {
  utils.BasePage
  ResultString string
}

func (sc *ServiceControllerImpl) single(w http.ResponseWriter, r *http.Request) {
  data, err := sc.get(w, r)

  if len(r.Header["Content-Type"]) == 1 && strings.Contains(r.Header["Content-Type"][0], "application/json") {
    if err != nil {
      sc.renderer.JSON(w, http.StatusBadRequest, map[string]string{
        "error": err.Error(),
      })
      return
    }
    sc.renderer.JSON(w, http.StatusOK, data)
    return
  } else if err != nil {
    http.Error(w, err.Error(), http.StatusBadRequest)
  }

  resultString, _ := json.Marshal(data)
  t, _ := template.ParseFiles("views/layout.html", "views/service.html")
  t.Execute(w, servicePage{sc.basePage.Get(r), string(resultString)})
}

// ...
```

#### utils/

Files that belong in the `utils` directory provide small levels of functionality to support the rest of the codebase. In my case, I have structs which access the request context, manipulate the session, and define a base page for specific pages to inherit. Here's where I access the context to set the user:

``` go utils/current_user_accessor.go
package utils

import (
  "github.com/gorilla/context"
  "github.com/larryprice/refermadness/models"
  "net/http"
)

type CurrentUserAccessor struct {
  key int
}

func NewCurrentUserAccessor(key int) *CurrentUserAccessor {
  return &CurrentUserAccessor{key}
}

func (cua *CurrentUserAccessor) Set(r *http.Request, user *models.User) {
  context.Set(r, cua.key, user)
}

func (cua *CurrentUserAccessor) Clear(r *http.Request) {
  context.Delete(r, cua.key)
}

func (cua *CurrentUserAccessor) Get(r *http.Request) *models.User {
  if rv := context.Get(r, cua.key); rv != nil {
    return rv.(*models.User)
  }
  return nil
}
```

#### models/

Models are data structures for representing my database concepts. I use my models to access the database - I create an empty model and then populate it with a database query. Here's an example of a model I use:

``` go models/service.go
package models

import (
  "gopkg.in/mgo.v2"
  "gopkg.in/mgo.v2/bson"
  "strings"
  "time"
)

type Service struct {
  // identification information
  ID          bson.ObjectId `bson:"_id"`
  Name        string        `bson:"name"`
  Description string        `bson:"description"`
  URL         string        `bson:"url"`
  Search      string        `bson:"search"`
}

func NewService(name, description, url string, creatorID bson.ObjectId) *Service {
  url = strings.TrimPrefix(strings.TrimPrefix(url, "http://"), "https://")
  return &Service{
    ID:            bson.NewObjectId(),
    Name:          name,
    URL:           url,
    Description:   description,
    Search:        strings.ToLower(name) + ";" + strings.ToLower(description) + ";" + strings.ToLower(url),
  }
}

func (s *Service) Save(db *mgo.Database) error {
  _, err := s.coll(db).UpsertId(s.ID, s)
  return err
}

func (s *Service) FindByID(id bson.ObjectId, db *mgo.Database) error {
  return s.coll(db).FindId(id).One(s)
}

func (*Service) coll(db *mgo.Database) *mgo.Collection {
  return db.C("service")
}

type Services []Service

func (s *Services) FindByIDs(ids []bson.ObjectId, db *mgo.Database) error {
  return s.coll(db).Find(bson.M{"_id": bson.M{"$in": ids}}).Sort("name").All(s)
}

func (*Services) coll(db *mgo.Database) *mgo.Collection {
  return db.C("service")
}

// ...
```

#### views/

I keep my goalng templates in the `views` directory. Pretty basic - whatever templating engine I use, I throw it into this `views` directory.

#### public/

As usual, this directory is all my public files - `css`, `img`, `scripts`.

### That's cool - how do I run this?

It's no secret that I like [docker](https://www.docker.com/), so that's my preferred method of running this application. If I wanted to run this without docker, I would throw the root of this directory into `$GOPATH/src/github.com/larryprice/refermadness`. Running `go get` will fetch all the missing dependencies and running `go run main.go` or `go build; ./refermadness` should run the file. If you do want to use docker (and you know I do), I start with a `Dockerfile`:

``` docker Dockerfile
FROM golang:1.4

RUN go get github.com/codegangsta/gin

ADD . /go/src/github.com/larryprice/refermadness
WORKDIR /go/src/github.com/larryprice/refermadness
RUN go get
```

And, since I also love [compose](https://github.com/docker/compose), I run everything with a compose file. I use a JSC transformer, a SASS transpiler, and a mongo database, so my `docker-compose.yml` file looks like:

``` yml docker-compose.yml
main:
  build: .
  command: gin run
  env_file: .env
  volumes:
    - ./:/go/src/github.com/larryprice/refermadness
  working_dir: /go/src/github.com/larryprice/refermadness
  ports:
    - "3000:3000"
  links:
    - db
sass:
  image: larryprice/sass
  volumes: 
    - ./public/css:/src
jsx:
  image: larryprice/jsx
  volumes: 
    - ./public/scripts:/src
db:
  image: mongo:3.0
  command: mongod --smallfiles --quiet --logpath=/dev/null
  volumes_from:
    - dbvolume
dbvolume:
  image: busybox:ubuntu-14.04
  volumes:
    - /data/db
```

And then run `docker-compose up` to run all the containers and start up the server.

...And that's that! I like setting my code up this way as a compromise between idiomatic Go and ruby on rails. I inject parameters whrever possible to keep things testable. I keep my concerns separated as much as possible. Go is fast and the syntax is clean, but there aren't enough examples of creating full-blown web applications available. At this point, we don't know the future of Go, but I anticipate it becoming big. Hopefully this architecture overview can help someone get started making the transition from a more traditional MVC framework to golang.
