---
layout: post
title: "A Hacky Solution to the Unicode data in a Unicode-only collation problem"
date: 2012-10-19 17:23
comments: true
categories: [ruby, rails, hacks, database, web]
---

The Issue Rises <a id="problem"></a>
---------------

Earlier this week a defect was found in my application. A defect that I could have sworn I fixed several weeks ago and written about in a [previous blog post][pb]. Let me start from the beginning:

[pb]: http://nullablevoid.blogspot.com/2012/10/unicode-data-in-unicode-only-collation.html

#### The Setup ####

The application is a web app using Rails 3.2, MSSQL Server for the database, and Tiny-TDS for database communications from the CloudFoundry server. There are three different types of builds and three databases I access: development (for development, obviously), staging (for testing), and production (for the users to complain about, mostly).

#### The Original Issue ####

I inherited this code and ran it on my dev build with no issues. Upon doing some testing with the staging build, the page crashed. The logs revealed the following (horrible) error:

``` ruby Vague error
Unicode data in a Unicode-only collation or ntext data cannot be sent to clients using DB-Library (such as ISQL) or ODBC version 3.7 or earlier.
```

What does this mean? Well. Beats me. [Several][s1] [sources][s2] had [similar][s3] [issues][s4] and the ones I liked eventually came to the conclusion that ntext and nvarchar variables in the database were ticking off the host server. [Apparently][msdn], text variables are translated to nvarchar(MAX), where MAX is something like 2GB of data. I hunted through my database and, sure enough, the 'Narrative' column was an nvarchar(MAX).

[s1]: http://dirk.net/2010/09/18/sql-server-with-freetds-unicode-data-error/
[s2]: http://stackoverflow.com/questions/5414890/mssql-query-issue-in-php-and-querying-text-data
[s3]: http://stackoverflow.com/questions/8705008/tiny-tds-error-on-heroku-connecting-to-sqlserver-db
[s4]: http://findyourscript.com/index.php/2011/05/20/unicode-data-in-a-unicode-only-collation-or-ntext-data-cannot-be-sent-to-clients-using-db-library/
[msdn]: http://msdn.microsoft.com/en-us/library/ms186939.aspx

#### The Original Fix ####

Based on the mighty power of the internet, I decided that the best thing for me to do was to change the variable in the database from a "text" to a "string" with a limit of 8000 (which translates to varchar(8000)) using this migration:

``` ruby Simple migration
class ChangeNarrativeColumnToVarChar < ActiveRecord::Migration
  def change
    change_column :evaluation, :narrative, :string, :limit => 8000
  end
```

I ran through my repro steps and... Drumroll... Suspense... It worked! Or so it appeared. I went to the narrative textbox and put some words in it, saved, and confirmed that everything was great. Then I pushed it to production and heard nothing for two weeks.

Failure Is Always An Option <a id="failure"></a>
---------------------------
Too bad that wasn't the end of the story. This week my users finally started using the app again. They found all kinds of defects, of course, but one in particular that caught me off guard: When viewing the narrative text, which we had "fixed" using that little migration above, the text cut off to about two lines. Two lines? I never really bothered to test more than a couple words or a short, goofy phrase. So I opened up the app on my dev build and it worked great with up to 8000 characters. I switched over to the staging build and was able to reproduce the error immediately.

At first I thought it was just the test_area, but I was wrong. Even static fields which displayed narrative cut off text. After some testing, the text was always cut off to 255 characters. I watched the SQL logs and confirmed that all 8000 characters would come back from the SQL queries. I looked in the database and verified the data was still present. What was going on?

#### Dear Rails: Oh, you ####

I thought and eventually realized something: the default of a "string" in Rails is 255 characters. Rails was cutting off my text. Curse you, Rails, I trusted you with my heart!

#### The Fix ####

Alright. The fix. Unfortunately, the fix sucks. In my app, I had to get the whole model that contained a narrative. Doing only that, the narrative would be cut short. So then I had to get the narrative again, and this time cast that sucker to a "text."

``` ruby < 1337 Hax
@eval = Evaluation.find(:id => id)
@eval.narrative = Evaluation.select("id as id, CAST(narrative as text) as narrative").where(:id => id).first.narrative
```

There's a part of me that likes sensible, clean code. This code did not come from that part of me. If you really want, you can do a select and get all the columns of your model, and then case the field in question, but what if your columns change? I didn't want to be responsible for that, especially after I hand this code off to someone else in the coming weeks.
