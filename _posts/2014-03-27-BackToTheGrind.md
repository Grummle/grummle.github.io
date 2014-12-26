---
layout: blog 
title: "Back to the grind" 
---

So I've got an endpoint, and I've got quite a few different filters setup. Now I need to get it so that it returns the total available after the filters are applied. Then order and skip/take a page.

I'm having issues with the repository and IOrderedQueryable types not casting properly. So time to figure that out.....

<!--more-->

*Skull bashing....*

Ok, easy fix after cleaning up the code a bit. Always good when its easy to understand wtf is going on.

And we've got timeouts from the database server.......  
And we've got someone running 6700 reports at the same time....  
And We've got possilbe issues with C# example on how to use Easy SMTP...  


And where back...
Digging through old code to figure out what we need to display for optout and bounce status.

*Sifting Shit....*

Ok, optout is bool, but bounce is a type. Looks like we are serializing the bounce type as a string so thats an easy one hopefully. Ok a coupel ng-if's and we've got that done.

Another 'feature' is subbit lists...  
So a sub bit is basically a bool on a row that determines if the user is part of that subbit. So add an endpoint to get current lists subbits

*Fubu Enpoint montage...*

Now add a function to the Lists.js data object like so

```javascript
      this.getSubBits = function(){            return $http.get(path+this.id+"/subbit").then(function(response){                return response.data;            })        }```

M'kay add yet another filter to the contacts endpoint to specify the subbit...

Add the select with a empty option to the html.

Ok done.

Add a link to the 'Add Recipient' button...
