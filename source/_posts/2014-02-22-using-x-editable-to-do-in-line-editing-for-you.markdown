---
layout: post
title: "Using x-editable to do in-line editing for you"
date: 2014-02-22 19:15:00 -0500
comments: true
categories: web ruby
---

In-line editing is traditionally difficult. Taking a static HTML node and turning it into an editable text field and then sending that data off somewhere is a little bit more Javascript than I like to write. My team and I came up against a very difficult UX problem which I spent over a week trying to understand. After building and discussing several solutions, we eventually decided to narrow the scope of what the user should be able to change. We would present the user with a table where one of the columns would be editable.

Rather than showing a table with the third column always as a field for text entry, I used the Javascript library [X-editable](//vitalets.github.io/x-editable/). X-editable allows me to display my editable item as a link. When the user clicks the item, the link turns into a text field with a Save/Cancel button (and a 'clear' button as a bonus). The 'Save' button submits an AJAX 'POST' or 'PUTS', where we are then allowed to validate and save the data. The 'Cancel' button turns the text field back into a link with the original value.

Hooray, someone else has already done the hard work for us! So, what I want to do is dynamically generate a table with objects and allow the user to edit one of the columns. In my case, only the end column is editable, but I could just as easily spread this availability to all my columns. I'm using all my favorite tools to create this page, specifically [ruby](//ruby-lang.org/en/), [Sinatra](//sinatrarb.com), [Twitter Bootstrap](//getbootstrap.com), and [HAML](//haml.info).

X-editable has implementations using Bootstrap, jQuery UI, and jQuery. Since I'm already using Bootstrap in my app, I'm going to go with that version. Note that there are Bootstrap 2 and Bootstrap 3 builds of X-editable, and I'm using the Bootstrap 3 variety.

First I include the necessary files at the top of my HAML document.

``` haml /views/manage_data.haml
%script{src: "//code.jquery.com/jquery.js"}
%script{src: "//getbootstrap.com/dist/js/bootstrap.min.js"}
%link{href: "//getbootstrap.com/dist/css/bootstrap.min.css", rel: "stylesheet"}

%link{href: "//cdnjs.cloudflare.com/ajax/libs/x-editable/1.5.0/bootstrap3-editable/css/bootstrap-editable.css", rel: "stylesheet"}
%script{src: "//cdnjs.cloudflare.com/ajax/libs/x-editable/1.5.0/bootstrap3-editable/js/bootstrap-editable.min.js"}
```

I'm going to dumb my page down so I only have to show the important parts. I'm going to create a table from an array of hashes called `@options`. Each hash has two important fields for this table: Points and Dollars. I will iterate over `@options`, displaying the `points` field as plain text and the `dollars` field as an X-editable link. Each hash also has a `value` field containing the primary key for the object to be edited.

``` haml /views/manage_data.haml
.container
  .row
    .col-md-6
      %table{id: "edit_points_goal", class: "table table-responsive table-hover table-bordered"}
        %thead
          %tr
            %th{width: "35%"}
              Points
            %th{width: "65%"}
              Dollars
        %tbody
          - @options.each do |option|
            %tr
              %td{style: "height: 45px; padding: 4px 8px; vertical-align: middle;"}
                #{option[:points]}
              %td{style: "height: 45px; padding: 4px 8px; vertical-align: middle;"}
                %a{href:"javascript:void(0)", "data-type"=>"text", "data-pk"=>"#{option[:value]}", "data-url"=>"/update", "data-title"=>"Enter dollar amount"}
                  #{option[:dollars]}
```

The important part is the `a` tag:

* `href` goes nowhere.
* `data-type` tells X-editable how it will edit the presented data. According to the docs, types include text, textarea, select, date, checklist and more.
* `data-pk` is the primary key of our data in the database. In my case, that comes to me stored in the `value` field of the option hash.
* `data-url` is the `post` method that will be used to interpret the data.
* `data-title` is used to tell the user what to do.

Our code won't do anything yet seeing as it's not connected. We need some Javascript to do that. I put the Javascript at the top of my file, underneath the included files. The first thing we need to do is tell X-editable what type of editing we'll be doing. The options are in-line editing or a pop-up. I want the in-line editing in this case. The second thing I want to do is to set the `editable` attribute on the appropriate DOM objects. Since I'm using an array, I found the easiest way to do this was to start at the table's id (edit_points_goal) and trace down to the `a` tag.

``` haml /views/manage_data.haml
:javascript
  $.fn.editable.defaults.mode = 'inline';

  $(document).ready(function() {
    if (#{vm.editable}) {
      $('#edit_points_goal tbody tr td a').editable();
    }
  });
```

Now we need to deal with the `post` request. If you're really impatient to see things work, you should be able to see your in-line editable code in action, but the `post` call will fail with a "NoMethodError."

Our `post` is going to be really simple. We verify that we have a non-negative integer and we either save and return 200 or we return 400 with an appropriate message. Our table in this example is just called `Data`, and we use `find` to get a value out of the database.

``` ruby web.rb
post "/update" do
  data = Data.find(id: params["pk"])

  unless params["value"].match(/[^0-9]/)
    data = params["value"].to_i
    data.save
    return 200
  end

  return {400, [], "Please enter a valid non-negative number"}
end
```

Things should be working now. When you enter good data and click the 'Ok' button, our `post` will be called and the text field will turn back into a link. When you enter bad data, you should see our error message below the text field box.

Issues I encountered:

* Table column width shifting - Fixed by setting the widths explicitly, as seen above (35% and 65%).
* Table height shifting - Fixed by setting the style of the `td` as seen above to give a larger height, more padding, and aligning the inner objects explicitly. I would recommend moving this definition to a `.css` or `.scss` file.
* "NoMethodError" after clicking Go - Unfortunately, if you made any coding errors in your route, you won't be able to see the Sinatra error page but will instead you'll be presented with a giant wall of HTML below the text box. Try to parse this error, but if you struggle to find out what went wrong you can always substitute a form-post method in place of the `a` tag, which may allow you to more easily figure out the problem.

The [X-editable docs](//vitalets.github.io/x-editable/docs.html) are extremely helpful for beginners. There is even more detail on the X-editable site, including editing multiple items, editing dates, and using a pop-up to edit the data.
