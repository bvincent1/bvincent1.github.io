---
layout: post
title: "Simple Tracking Snippet"
description: "How to make a tracking snippet in under 20 minutes"
tags: [serverless, javascript]
---

# Tracking a page
Making a website is an awesome feeling. No matter who you're making it for or what purpose it has, it's always satisfying to know that there are people out there seeing your content and navigating your site. It's only after that first buzz that you start to question yourself and wonder who actually is visiting your site. Maybe you add a tag manager, and analytics (probably google analytics, but I don't judge). Then you patiently wait, hoping to see blips on the graph that show your traffic and imagine the steadily rising number. But maybe you want more. You want to be able to track specific people, or even track where your guests are coming from.

Welcome to snitcher, my custom tracking snippet. If your reading this now then congrats! You've been logged as a visitor and I have the following pieces of information about you and your visit.

- ip address
- time of visit
- url that you visited
- query values

Now this is some pretty sparse information, and really I could do a lot better using some user id generation, header analysis, and whatnot. But for my purposes this is fine.


My main reason for having this snippet is to know when someone in particular navigates my site. This is done via the query values. I generally leave special `ref=blah` key-values in all the links to my blog. These refs allow me to track where the user came from. For instance my resume has a link to my blog, and it includes the key value `ref=resume`. Now when I send in my resume I can roughly check to see if they even looked at it, or if they just ignored it and moved on. It's not perfect, but really... I'm not picky.

At it's core this project consists of 2 chunks, the frontend snippet that handles the data gathering, and the backend that logs this information. Let's dive into the frontend and see what we can scrape up!

# Snitcher Snippet - Special Methods

Here's the source for my link manipulation of for getting the url query parameters. It's not terribly special but it is key for tracking a user's navigation across the site. I kinda lucked out since all of the links are `<a>` tags (but not really since it's my site and I could change them if this weren't the case).

In addition I have to get the query and parameter from the url, this uses some [stackoverflow](http://stackoverflow.com/questions/901115/how-can-i-get-query-string-values-in-javascript) resources to get them by keyword. The only real design that this causes is that I have to know what the parameter is ahead of time, which means that I won't propagate extraneous or extra links automatically.
```html
<script>
  function mutateLinks(query) {
    document.querySelectorAll("a").forEach(function (e,i,a) {
      if (e.href.indexOf("?") < 0) {
        e.href += "?" + query;
      }
      else {
        e.href += query;
      }

    });
  }

  function getParameterByName(name, url) {
    if (!url) {
      url = window.location.href;
    }
    name = name.replace(/[\[\]]/g, "\\$&");
    var regex = new RegExp("[?&]" + name + "(=([^&#]*)|&|#|$)"),
        results = regex.exec(url);
    if (!results) return null;
    if (!results[2]) return '';
    return decodeURIComponent(results[2].replace(/\+/g, " "));
  }
</script>
```

# Snitcher Snippet - Main

Here's the main method that I call that sends the gathered information to my backend lambda method. It first gets the ip address of the client using [httpbin](https://httpbin.org/) ( A fave GET / POST testing site of mine that also happens to have IP lookup as an API) In the future I plan on replacing this with a different API call that can also do region and hostname logging so I can scrape additional information for my lookup.

Once I've fetched the resources its all a simple JSON post to my backend and then I'm done!

```html
<script src="https://ajax.googleapis.com/ajax/libs/jquery/2.1.4/jquery.min.js"></script>
<script>
  function main() {
    const endpoint = "GET_UR_OWN";

    var ref = getParameterByName("ref");
    if (ref) {
      mutateLinks("ref=" + ref + "&");
    }

    $.getJSON("https://httpbin.org/ip").done(function (data) {
      var view = {};
      view.origin = data.origin;
      view.ref = getParameterByName("ref");
      view.date = Date.now();
      view.href = window.location.href;

      $.post(endpoint, JSON.stringify(view));
    });
  }

  main();
</script>
```
