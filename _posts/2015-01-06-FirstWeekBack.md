---
layout: blog
title: "First Week Back"
---

Time to vomit out some more basic todo list type stuff to see if we can keep Mr. Dillon....SQUIRREL! on track.

<!--more-->

##TODO
* ~~~Clean up Office, per JM, "Hide all the Crap!"~~
* Open Ticket with Dell regarding drives not recognized
* ~~List Segment change to OR~~
* List Upload
  * ~~require checking 'This is not an uploaded list...'~~
  * require one and only one email column
* Improve error handling
  * How to serialize exceptions properly for Elastisearch
  * take a look at filtering Path to long errors.
* Make up some questions for interviewies
* Shopping List
  * resolve CSS conflicts
* Put away Laundry
* Post Camera Equipment on Craigslist
  * pictures of gear
* Post hiking gear on craigslist
  * pictures of gear
* Hydroponics
  * ~~make cutting list~~
  * cut lumber
  * ordre some more seeds
* Spice Rack
  * get more containers
  * design holder
* clean out DIY soylent stuff
* start crashteststocks blog
* ~~try and setup comed auto pay, if they finally have their shit together.~~
* ~~pick up photos from Walgreens~~
* ~~Check on brake pads, wtf are they? Shipping 1/5~~?
* ~~Update drivers/firmware for RMSQL3~~
  * ~~PERC H710P~~
    * ~~Driver (updated to 6.8)~~
    * ~~firmware (check, pretty sure up to date)~~
  * ~~Bios~~
  * ~~iDrac7 (up to date afaict)~~
  * ~~lifecycle Controller~~
* ~~take a look at reproducing Angular loop on login page~~
  * ~~hear back from dan (opera on linux, not failing same way)~~
  * ~~opera on mac (couldn't get it to fail)~~
* ~~Post New years part Deux photos~~

## List Upload

###Require them to swear on their first born that the list isn't purchased or some such nonsesne. Eaisly enough you can just do a ng-require="true" and it'll 
invalidate the form if they don't check the box. Nice.

###Require one and only one email column
So if a user uploads a list we need to have them choose what columns match with which list elements. 
Actually we could expand this out alot for warnings on types and stuff like that but for right now we are just going to go with looking at if they
have a column marked as email yet or not. You HAVE to have an email column (duh, otherwise its kinda useless to RM). So this is basically form validation that spans not only multiple form elements but also dynamic form elements. This is a new one for me in Angular.
