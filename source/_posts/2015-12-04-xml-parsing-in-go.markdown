---
layout: post
title: "XML Parsing in Go"
date: 2015-12-04 15:30:14 -0500
comments: true
categories: go golang xml
---

XML Parsing in Go
=================

Need to parse an XML file in go? XML (en|de)coding is build into go's standard library. Basic parsing of nodes and embedded nodes is straightforward, but there are some interesting things to think about when you need to parse attributes, lists, and if you don't want to create new structs all over the place.

For this exercise, let's look at the following XML document:

``` xml
<person>
	<name>Luann Van Houten</name>
	<addresses>
		<address type="secondary">
			<street>321 MadeUp Lane</street>
			<city>Shelbyville</city>
		</address>
		<address type="primary">
			<street>123 Fake St</street>
			<city>Springfield</city>
		</address>
	</addresses>
</person>
```

### Defining a Data Structure ###

The simplest node to parse is going to be `<name>`. We'll need an object to unmarshal the XML into, which we'll call `Person`. Every field on `Person` will be evaluated by the XML encoder to be populated based on the field's name. However, our struct field names and node names don't usually correspond that simply. Because of this, we'll use `xml` struct tags to identify how to map each node to a go field. Here's an example for `<name>`:

``` go
type Person struct {
  Name string `xml:"string"`
}
```

The next node contains a list of `<address>` nodes. For an address node, we'll have to create a similar struct with `City` and `Street` fields:

```
type Address struct {
	Street string `xml:"street"`
	City string   `xml:"city"`
}
```

While parsing each `Address`, we also want to find the `type` of address, which can be found as an attribute on the `<address>` node. By adding `attr` to our XML struct tag, we can properly parse the `type` field as well:

``` go
type Address struct {
	Street string `xml:"street"`
	City string   `xml:"city"`
	Type string   `xml:"type,attr"`
}
```

A `Person` has a list of `Address`es. Since we know that `address` is a direct descendant, we can use the `>` keyword to parse each `<address>` from `<addresses>` into an embedded struct on our `Person` struct.

``` go
type Person struct {
  Name string         `xml:"string"`
	Addresses []Address `xml:"addresses>address"`
}
```

This code will work to parse the original document, but do we really need to define a formal struct for addresses? If there was only one address, we could put all the fields directly on the `Person` struct. However, since we're dealing with a list, our best option is to use an anonymous struct:

``` go
type Person struct {
	Name      string `xml:"name"`
	Addresses []struct {
		Street string `xml:"street"`
		City   string `xml:"city"`
		Type   string `xml:"type,attr"`
	} `xml:"addresses>address"`
}
```

### Binding the Data Structure ###

We can use the [encoding/xml](https://golang.org/pkg/encoding/xml/) package to decode the given XML. Given that our raw XML document is stored in a `[]byte` called `document`, we'll use `xml.Unmarshal` to bind our data structure to the XML document:

``` go
var luann Person
xml.Unmarshal(document, &luann)
```

### Final Code ###

Let's put it all together, including a `main` function that will use `fmt.Println` to print the results of binding the data structure to the XML document. `Unmarshal` could return an error, but we'll ignore it in this case.

``` go
package main

import (
	"encoding/xml"
	"fmt"
)

var personXML = []byte(`
<person>
	<name>Luann Van Houten</name>
	<addresses>
		<address type="secondary">
			<street>321 MadeUp Lane</street>
			<city>Shelbyville</city>
		</address>
		<address type="primary">
			<street>123 Fake St</street>
			<city>Springfield</city>
		</address>
	</addresses>
</person>`)

type Person struct {
	Name      string `xml:"name"`
	Addresses []struct {
		Street string `xml:"street"`
		City   string `xml:"city"`
		Type   string `xml:"type,attr"`
	} `xml:"addresses>address"`
}

func main() {
	var luann Person
	xml.Unmarshal(personXML, &luann)
	fmt.Println(luann)
}
```

I've posted this code as [a gist](https://gist.github.com/larryprice/204fec2e8d33979f8cac) in [the go playground](https://play.golang.org/p/qiSoxxb5tp) so you can see it in action.
