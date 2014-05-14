---
layout: post
title: "Better testing in Go with gocheck"
date: 2014-05-13 21:14:39 -0400
comments: true
categories: golang testing
---

As a quick reminder, [golang](//golang.org/) is a really fun programming language to use. It even includes [testing out of the box](//golang.org/doc/code.html#Testing)! Unfortunately, this out-of-the-box testing framework isn't all that great. It lacks the syntactic sugar of mature frameworks like [rspec](//rspec.info) or [gtest](//code.google.com/p/googletest/).

Of course, there are alternatives. I found an open-source library (licensed with Simplified BSD) called [gocheck](//labix.org/gocheck).

Gocheck, how do I love thee? Let me count the ways:

* Test fixtures
* Improved assertions
* Improved test output
* Sugar-coated syntax
* Test skipping
* Oh my

As usual, it's time to guide you through a contrived example. Start by installing the package:

``` bash
$ go get gopkg.in/check.v1
```

Let's see, what should we make... how about a tip calculating library? We should start by testing, because we're obsessed with TDD.

Test #1: Returns 0 for free meal

``` go calculator_test.go
package cheapskate

import (
  "testing"
  . "gopkg.in/check.v1"
)

// Hook up gocheck into the "go test" runner.
func Test(t *testing.T) { TestingT(t) }

type MySuite struct{}

var _ = Suite(&MySuite{})

func (s *MySuite) TestReturns0ForFreeMeal(c *C) {
  c.Assert(calculateTip(0.0), Equals, 0.0)
}
```

Not quite as obvious as the internal testing framework. First we hook up gocheck into the "go test" runner. Then we create a test suite; ours is empty for now and called `MySuite`. We call `Suite` to intialize the test runner with our custom suite. We then write our first test to assert that calculating the tip returns a value equal to 0. All tests must be prefixed with the word "Test". Now I'll write the implementation:

``` go calculator.go
package cheapskate

func calculateTip(bill float64) float64 {
  return 0.0
}
```

Running the tests...

``` bash
$ go test
OK: 1 passed
PASS
ok    _/home/lrp/Projects/2014/gocheck-quick  0.003s
```

Woohoo! All passed. What happens if we write a failing test?

``` go calculator_test.go
...
func (s *MySuite) TestReturns15PercentByDefault(c *C) {
  c.Assert(calculateTip(100.0), Equals, 15.0)
}
```

Results:

``` bash
lrp@cilantro:~/Projects/2014/gocheck-quick$ go test

----------------------------------------------------------------------
FAIL: calculator_test.go:19: MySuite.TestReturns15PercentByDefault

calculator_test.go:20:
    c.Assert(calculateTip(100.0), Equals, 15.0)
... obtained float64 = 0
... expected float64 = 15

OOPS: 1 passed, 1 FAILED
--- FAIL: Test (0.00 seconds)
FAIL
exit status 1
FAIL  _/home/lrp/Projects/2014/gocheck-quick  0.003s
```

A nasty failure that one. I'll fix it and continue:

``` go calculator.go
package cheapskate

func calculateTip(bill float64) float64 {
  return .15 * bill
}
```

I want to create a Setup method for my entire suite. I'll store some silly information there for information's sake. The minBill and maxBill variables will only be set when I first load the suite.

``` go calculator_test.go
package cheapskate

import (
  "testing"
  . "gopkg.in/check.v1"
)

// Hook up gocheck into the "go test" runner.
func Test(t *testing.T) { TestingT(t) }

type MySuite struct{
  minBill float64
  maxBill float64
}

func (s *MySuite) SetUpSuite(c *C) {
  s.minBill = 0
  s.maxBill = 100
}

var _ = Suite(&MySuite{})

func (s *MySuite) TestReturns0ForFreeMeal(c *C) {
  c.Assert(calculateTip(s.minBill), Equals, 0.0)
}

func (s *MySuite) TestReturns15PercentByDefault(c *C) {
  c.Assert(calculateTip(s.maxBill), Equals, 15.0)
}
```

What if I wanted to set some information at the start of each test? I'll log the current test number on the suite, updating it every time I run a test:

``` go calculator_test.go
package cheapskate

import (
  "testing"
  . "gopkg.in/check.v1"

  "fmt"
)

// Hook up gocheck into the "go test" runner.
func Test(t *testing.T) { TestingT(t) }

type MySuite struct{
  testNumber int
}

func (s *MySuite) SetUpTest(c *C) {
  fmt.Println(s.testNumber)
  s.testNumber += 1
}
...
```

Result:

``` bash
$ go test
0
1
OK: 2 passed
PASS
ok    _/home/lrp/Projects/2014/gocheck-quick  0.004s
```

You can create tear down methods for suites and tests in the same manner, replacing the appropriate words above.

There's loads of other cool stuff gocheck can do, I've barely scratched the surface with what little experience I've had using it. Like any testing framework, I'm sure it has its advantages and disadvantages, but it sure beats the pants off the off-the-shelf framework Google includes with golang. 
