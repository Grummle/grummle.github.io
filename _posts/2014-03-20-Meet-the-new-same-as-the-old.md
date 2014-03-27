---
layout: blog 
title: "Meet the New blog, same as the old blog" 
---

So for the most part I'd stopped blogging, having a baby does that to you. However it seems like it'd probably be a good thing to show that I didn't stop working 6 months ago so probably looking at taking it up again. As before it'll be a rambling stream of consciousness that is poorly if ever spell checked. You have been warned.

<!--more-->

So today we've got to get back in the swing of things after a couple days off. Yesturday I had to fix some incompetents wiring job in our new office. 30% of the ports where either miswired or improperly punched down. What I was working on was a conversion of the contacts detail page.

## Contacts Detail
So as per RM usual we don't have a designer on staff so we are redoing the page with AngularJS to make it nicer to work with, but not taking full advantage by redoing the UX. Whateves. 

The Old: 
![Old Contact Details](/images/contact.details.old.png)

So first off we've got got a list of contacts and a few details. Not a big deal. Complicating this is that a list can have a basically unlimited number of contacts. We've got a couple lists with 5M+ so pulling them all down to the client isn't an option. AngularJS also doesn't seem to like having all that many on screen with two way binding. So we'll stick to the same layout as before. 20 or so on screen with paging buttons. Ugly but functional. So the end point so far is going to need to supply a list of contacts from a given list, that is sortable by several columns and is pagable.

We'll also need the endpoint to support searching through a list by column. Also it'll need to apply 'Segments'. This is where things get interesting.

### Technical Debt Rangling (aka Yak Shaving)

So Segments in RM are filters that are applied against a list. For instance in english a typical filter would be something like. "Give me all contacts that are from the city of Chicago" This is stored as a string of SQL in the database. We've moved away from SQL string concatenation (Finally). We are using a ORM that supports 'dynamic' SQL tables. [Gribble](https://github.com/mikeobrien/Gribble), which is written by a co-worker. Gribble is a Linq to SQL type library. So It'll take C# expressions. Ok, so we dig up a [SQL parser](http://www.sqlparser.com/sql-parser-dotnet.php). We'll use that to generate a tree, that we'll then visit to generate a C# expression for use with gribble. I think long term we'll generate a AST we'll store as json in the database to create our own little mini query language for but for right now this'll get our current project done and keep the boss happy.

Other things that the end point needs to be able to do is to filter based on the contacts current mailable status. Active, Opted In, Soft Bounce and Hard Bounce. This is also stored ad SQL string in the database (Lame), so we extend the visitor a little farther to handle those as well. So now we'll have to add in some bit fields for the endpoint and get those filters going.

*Furious Typing.....*

Ok, so it appears I've made a boo boo. My converter is not handling 'IS NOT NULL' properly, or the Gribble ORM isn't, but its the end of the day so... Note to self, your stuff is busted.....till tomorrow...

### 3/14/2014 12:41 - And Where Back....Finally

Its amazing how much time you can suck up with new security cameras and setting up your desk. Though you've got to admit the result is pretty freaking awesome.
![BattleStation](/images/battlestation.jpg)

So back to figuring out why when I visit a 'IS NOT NULL' node that I'm not getting whats expected. I missed this the first time around cause I missed the all important 'Writing a failing test first'. So now I have a failing test. And I'm an idiot. It works correctly. My string concatenation was messed up. Lost a day on a 10 sec issue. Dammit.

Ok, so now I've got about 5 filters I need to add to the xhr request. So I made a lame fluent'esk js object to do the query to the contacts endpoint.

```javascript
   var ContactsQuery = function(listId){        this.query = {};        this.listId = listId;        this.order = function(order,desc){            this.query.order = order;            this.query.desc = desc;            return this;        }        this.count = function(count){            this.query.count = count;            return this;        }        this.skip = function(count){            this.query.skip = count;            return this;        }        this.search = function(term, columnId){            this.query.search = term;            this.query.column = columnId;            return this;        }        this.filter = function(filterId){            this.query.filterId = filterId;            return this;        }        this.active = function(value){            this.query.active = value;            return this;        }        this.optout = function(value){            this.query.optout= value;            return this;        }        this.softBounce = function(value){            this.query.softBounce = value;            return this;        }        this.hardBounce= function(value){            this.query.hardBounce= value;            return this;        }        this.execute = function(){            return $http.get(path+listId+"/contacts",{params:this.query}).then(function(response){                return response.data;            })        }    }
```

Not sure I really should have bothered, but it works and I don't have a better idea at the moment.

Ok, current problem is that I need the total to reflect the filtered total not the total records available. I've got to run 2 queries. One to get the total and one to get the list of actual records I'm sending back. Not ideal.


