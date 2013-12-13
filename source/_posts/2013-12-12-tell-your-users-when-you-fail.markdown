---
layout: post
title: "Tell Your Users When You Fail"
date: 2013-12-12 19:41
comments: true
categories: non-technical
---

Your users put a lot of faith in you. They put up with your terrible design, your horrible taste in color scheme, and your ability to take up over 100% of their computer's memory. With that in mind, don't these poor people deserve a little bit of honesty?

I came across some code recently that irked me. This code asks for a handle to a database abstraction class and then runs a query on it. It looks kinda like this:

``` ruby settings.rb
def changeSettings(newSettings)
  db = dbFactory.get("settings.db")
  if !db.nil?
    db.execute(buildSaveQuery(newSettings))
  end
end
```

How nice that `changeSettings` doesn't try to play with a `nil` database, right? But what about the user's data? It wasn't saved. And no one knows. _Not even you_. Now, maybe the user will happily continue what he's doing until he notices that his new settings aren't quite right. So he goes back to them again. Same result. Repeat ad infinitum.

How I would initially approach this is to add an `else` statement that launches a dialog giving some useful data to the user. Whatever you do, for the love of code, DO NOT tell the user that you had database problems. For most applications, the user will have no idea what that means. It might look something like this:

``` ruby settings.rb
def changeSettings(newSettings)
  db = dbFactory.get("settings.db")
  if !db.nil?
    db.execute(buildSaveQuery(newSettings))
  else
    showDialog(title="Save Failed!",
               msg="I'm sorry, but your data could not be saved." +
                   "Press 'Try Again' to give it another go." +
                   "If the problem persists, restart the application" +
                   " or contact system support.")
  end
end
```

This is not a bad solution. But what about all the other places in the code I get the database from the factory? I'd have to update those as well. It would be better to be able to code the factory to launch a similar error dialog whenever this happens. Even better, let me hand the factory a reference to an appropriate dialog whenever I call `dbFactory.get`.

Another much more frightening possibility is to skip the `nil` check. Can you guarantee that `dbFactory.get` will never be `nil`? In that case, this option is actually pretty good.

Even if that's not a guarantee you can make, it may still be a valid solution. [Crash early before you do too much damage](http://www.artima.com/intv/defenseP.html). Now, if there's some kind of crash-handler or global exception-catcher in your application, then you can catch the impending crash and tell the user the application needs to be restarted... And the app probably needs to be restarted anyway! There's no need to keep running after you've reached some horrible error state that needs to be addressed.

Just do _something_! Your users (especially if that happens to include me) will appreciate your honesty, even if it's so brutal that they have to restart the application.
