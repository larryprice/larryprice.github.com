---
layout: post
title: "Finishing the Google Go Writing Web Applications Tutorial"
date: 2014-01-07 22:45:03 -0500
comments: true
categories: golang web
---

###A golang web app tutorial

I did some work with [Google Go](http://golang.org/) recently and had the chance to follow their great tutorial _[Writing Web Applications](http://golang.org/doc/articles/wiki/)_. The tutorial is pretty simple: use the Go http library to create a very simple wiki-style site. I like this tutorial a lot because there's not too much hand-holding, but they do eventually hand you the [final code listing](http://golang.org/doc/articles/wiki/final.go). Then the good people at Google give you the tall task of completing the following 'Other tasks' without solutions:

* Store templates in tmpl/ and page data in data/.
* Add a handler to make the web root redirect to /view/FrontPage.
* Spruce up the page templates by making them valid HTML and adding some CSS rules.
* Implement inter-page linking by converting instances of [PageName] to 
\<a href="/view/PageName"\>PageName\</a\>. (hint: you could use regexp.ReplaceAllFunc to do this)

This is what I'd like to go over. I scoured the web and didn't have much luck finding solutions to these issues. That would be okay if they were all trivial, but the final step is not straightforward. I'm going to assume you've already gone over the tutorial. You can see [my repository on Github](https://github.com/larryprice/gowiki/), and I have included links to the appropriate commits in the code sections of this blog post.

####Store templates in tmpl/ and page data in data/

The tutorial originally has the developer store all pages in the project directory. Every time a user made a new wiki page, a new file would creep into the project directory. All HTML templates were also stored in the project directory.

Moving templates is quite trivial. In the global scope:

``` diff wiki.go https://github.com/larryprice/gowiki/commit/9994d11b5275bc5faee911e5db2c994bc91052e2
-var templates = template.Must(template.ParseFiles("edit.html", "view.html"))
+var templates = template.Must(template.ParseFiles("tmpl/edit.html", "tmpl/view.html"))
```

I found moving the page data to `data/` was a little trickier, especially if the directory didn't already exist. You may not have the same issue, but I remedied this by creating the directory if it doesn't exist. My `save` function differences:

``` diff wiki.go https://github.com/larryprice/gowiki/commit/e86a707d37b802b2d59b8ef261b3fdcab46d5870
func (p *Page) save() error {
-    filename := p.Title + ".txt"
-    return ioutil.WriteFile(filename, p.Body, 0600)
+  os.Mkdir("data", 0777)
+  filename := "data/" + p.Title + ".txt"
+  return ioutil.WriteFile(filename, p.Body, 0600)
}
```

####Add a handler to make the web root redirect to /view/FrontPage

All we're going to do is create a simple handler called `rootHandler` that redirects to a new page called `FrontPage`. We then add it in the `main` function. The tutorial had us wrap out handlers in a function call to take some special actions, but that wrapper would mess up our handler in its current form. So I just `Redirect` to the `view` handler, which will then decide whether to view or create the FrontPage.

``` diff wiki.go https://github.com/larryprice/gowiki/commit/e41fccc2d244a3b0d62d600d94897a076c87d53d
+ func rootHandler(w http.ResponseWriter, r *http.Request) {
+   http.Redirect(w, r, "/view/FrontPage", http.StatusFound)
+ }

...

func main() {
+  http.HandleFunc("/", rootHandler)
  http.HandleFunc("/view/", makeHandler(viewHandler))
  http.HandleFunc("/edit/", makeHandler(editHandler))
  http.HandleFunc("/save/", makeHandler(saveHandler))
```

####Spruce up the page templates by making them valid HTML and adding some CSS rules.

I took my old `.html` files and put them through [a validator](http://validator.w3.org/#validate_by_input). Making them valid only involved adding `DOCTYPE`, `html`, and `head` tags. The `head` tag needed `meta`, and `title` tags and we were valid. I've shown `view.html` below.

``` diff view.html https://github.com/larryprice/gowiki/commit/771b4ecc8a550ee438720dc5c3d3f47954a1e4ff
+<!DOCTYPE html>
+<html>
+<head>
+<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
+<title>Wiki made using Golang</title>
+</head>
 <h1>{{.Title}}</h1>
 
 <p>[<a href="/edit/{{.Title}}">edit</a>]</p>
 
 <div>{{printf "%s" .Body}}</div>
+</html>
```

####Implement inter-page linking by converting instances of [PageName]

Converting [PageName] to a hyperlink was a bit more complicated than expected. I originally just tried to run the string through `ReplaceAllFunc` and replace all instance of [PageName] with an equivalent hyperlink. This does not work because we use Go's `ExecuteTemplate` method to render our template. `ExecuteTemplate` escapes any HTML that we give it to prevent us from displaying unwanted HTML. Getting around this was the fun part, because I want the benefit of escaped HTML while still having the ability to substitute my own HTML.

As it turns out, `ExecuteTemplate` will not escape variables of the type `template.HTML`. So I added another variable onto the `Page` struct called `DisplayBody`.

``` diff wiki.go https://github.com/larryprice/gowiki/commit/38c48717420de78f15dc48152ce16d1bdb417288
type Page struct {
     Title string
     Body  []byte
+    DisplayBody template.HTML
}
```

Next, I create a regular expression to find instances of [PageName] and I put the defintion above the `main` method.

``` diff wiki.go https://github.com/larryprice/gowiki/commit/38c48717420de78f15dc48152ce16d1bdb417288
+var linkRegexp = regexp.MustCompile("\\[([a-zA-Z0-9]+)\\]")
```

In my `viewHandler`, I escape `Body` and then set `DisplayBody` to that escaped string with the links substituted.

``` diff wiki.go https://github.com/larryprice/gowiki/commit/38c48717420de78f15dc48152ce16d1bdb417288
+  escapedBody := []byte(template.HTMLEscapeString(string(p.Body)))
+
+  p.DisplayBody = template.HTML(linkRegexp.ReplaceAllFunc(escapedBody, func(str []byte) []byte {
+      matched := linkRegexp.FindStringSubmatch(string(str))
+      out := []byte("<a href=\"/view/"+matched[1]+"\">"+matched[1]+"</a>")
+      return out
+    }))
  renderTemplate(w, "view", p)
}
```

To finish up, I modify the `view.html` to show `DisplayBody`. I don't use `printf`, because that would turn `DisplayBody` back into a `string` and `ExecuteTemplate` would escape it.

``` diff wiki.go https://github.com/larryprice/gowiki/commit/38c48717420de78f15dc48152ce16d1bdb417288
-<div>{{printf "%s" .Body}}</div>
+<div>{{.DisplayBody}}</div>
```

And that completes the extra exercises for Google's _Writing Web Applications_ tutorial. Hopefully one day this helps someone who gets stuck.
