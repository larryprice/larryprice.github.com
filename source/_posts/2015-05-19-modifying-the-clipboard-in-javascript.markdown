---
layout: post
title: "Modifying the Clipboard in Javascript"
date: 2015-05-19 10:09:56 -0400
comments: true
categories: web javascript
---

As of the writing of this post, Javascript is not allowed to modify the clipboard directly in modern web browsers. There is an [HTML5 spec in draft](http://www.w3.org/TR/clipboard-apis/), but it's unclear when/if this will become stable. Should we just give up on modifying users' clipboards and possibly violating their privacy? Heck no! We'll just use Flash.

[ZeroClipboard](https://github.com/zeroclipboard/zeroclipboard) is a library which will create an invisible Flash movie over an HTML element. Does that sound hacky to you? DOESN'T MATTER, COPIED TEXT.

ZeroClipboard is super easy to use, the only caveat being in order to use the Flash movie you're going to have to run some kind of web server. For the simple case of this example, I'll use ruby with the following [config.ru](https://raw.githubusercontent.com/larryprice/zeroclipboard-example/master/config.ru):

``` ruby
require 'bundler/setup'
require 'sinatra/base'

# The project root directory
$root = ::File.dirname(__FILE__)

class SinatraStaticServer < Sinatra::Base

  get(/.+/) do
    send_sinatra_file(request.path) {404}
  end

  not_found do
    send_file(File.join(File.dirname(__FILE__), 'public', '404.html'), {:status => 404})
  end

  def send_sinatra_file(path, &missing_file_block)
    file_path = File.join(File.dirname(__FILE__),  path)
    file_path = File.join(file_path, 'index.html') unless file_path =~ /\.[a-z]+$/i
    File.exist?(file_path) ? send_file(file_path) : missing_file_block.call
  end

end

run SinatraStaticServer
```

Download [ZeroClipboard.js](https://raw.githubusercontent.com/larryprice/zeroclipboard-example/master/ZeroClipboard.min.js) and [ZeroClipboard.swf](https://github.com/larryprice/zeroclipboard-example/raw/master/ZeroClipboard.swf) and save them to the base directory. Create a file called [index.html](https://github.com/larryprice/zeroclipboard-example/blob/d3a941f6b5783108ebec370d9ae16d1b534bf2ce/index.html):

``` html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<style>
  #copy:hover {
    cursor: pointer;
  }
</style>
</head>
<body>
  <button id="copy" data-clipboard-text="you should be more careful with your clipboard">
    Copy Something!
  </button>
</body>
<script type="text/javascript" src="ZeroClipboard.min.js"></script>
<script type="text/javascript">
  window.onload = function() {
    new ZeroClipboard(document.getElementById("copy"));
  };
</script>
</html>
```

This use case is simple: I create a button which the user will click to copy text to my keyboard. The text I want to copy is included on the element with the `data-clipboard-text` attribute. I initialize a ZeroClipboard client with `new ZeroClipboard(...)`. The application can be run with `rackup`: try it out for yourself!

Of course, things can always get more complicated. Let's say that I want to copy something dynamic to the clipboard, [like the current state of the seconds hand on the clock](https://github.com/larryprice/zeroclipboard-example/blob/5517b52b5713e75a183f6269184231342d561a7e/index.html):

``` html
<body>
  <button id="copy">Copy Something!</button>
</body>
<script type="text/javascript" src="ZeroClipboard.min.js"></script>
<script type="text/javascript">
  window.onload = function() {
    var zclip = new ZeroClipboard(document.getElementById("copy"));
    zclip.on('ready', function(event) {
      zclip.on('copy', function(event) {
        zclip.setText(new Date().getSeconds() + "");
      });
    });
  };
</script>
```

There are more events to play with, but this is enough to get started. Check out the [ZeroClipboard instructions](https://github.com/zeroclipboard/zeroclipboard/blob/master/docs/instructions.md) wiki for more examples.

All of the code in this post can be found [on Github](https://github.com/larryprice/zeroclipboard-example).
