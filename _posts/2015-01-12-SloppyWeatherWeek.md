---
layout: blog
title: "Sloppy Weather Week"
---

Time to vomit out some more basic todo list type stuff to see if we can keep Mr. Dillon....SQUIRREL! on track.

<!--more-->

##TODO
* Get Laundry Info for Jay (Landlord)
  * Washer/Dryer Models
  * Alcove Dimensions
* ~~Reconcile CC and personal account~~
* Swap Drive in RMDATA
* Swap memory DIMMs in RMVSH
  * ~~Swapped out possible bad into A2, wait for fail?~~
* SurveyMonkey Integration
* ~~Open Ticket with Dell regarding drives not recognized~~
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
  * fold laundry, the bain of my existence 
* Post Camera Equipment on Craigslist
  * pictures of gear
* Post hiking gear on craigslist
  * pictures of gear
* Hydroponics
  * ~~make cutting list~~
  * cut lumber
  * ordre some more seeds
* Spice Rack
  * ~~get more containers~~
  * design holder
* clean out DIY soylent stuff
* start crashteststocks blog
* Check on brake pads, wtf are they? (got a return notice for them.... wtf)?


##Round six of driver/firmware upgrades
So the company we bought the drives off of came back and said to update Open Manage Server Administrator and the make sure the H710P's driver was up to date.
The H710P driver has been up to date at version 6.802.19.0 and I've previously downloaded the [OM SA 7.4.02 patch](http://www.dell.com/support/home/us/en/19/Drivers/DriversDetails?driverId=24H24&fileId=3397983531&osCode=WS8R2&productCode=poweredge-r720xd&languageCode=EN&categoryId=SM) on several occasions and couldn't get it to install.
Its a .msp file not a .msi so the ```msiexec /i <msipath>setup.msi /l*v c:\temp\msi.log``` verbose logging isn't going to work. [This site](http://www.experts-exchange.com/Programming/Installation/Q_24757147.html) has the syntax to turn loggin on for a msp however. Funnily enough the .msp that wouldn't install multiple times, installs fine with verbose loggin on elminating the need for the logging... whatever. So its installed and Open Manage still doesn't like our drives. Ugh.

So got on tech chat with Dell, he has me do a dset (dell e support audit thingy) and send it to him. In there the drives are Dell vendor so they are dell drives. He also pokes around and can't find any reason to Open Manage to be rejecting them. Also we try to figure out why Open Manage 7.4.1 won't install. I finally found a [compatibility matrix](http://www.dell.com/support/Manuals/us/en/19/Topic/dell-opnmang-sw-v7.4/OM_SupportMatrix-v1/en-us/GUID-7D7B572E-4397-480F-80C2-691201203FB0) that shows we aren't compatible. Its weird that they did a point release that has such limited compatibility.

##Bad Memory Hunt
We've got a box RMVSH1 that we use for virtual machines. It decided to become basically unusable a couple of weeks ago. After MOB did some poking about he found that the memory was SLOW. So we suspected a bad DIMM. It had 4 DIMMs, one of which was not like the others. So we pulled that one. Eureka! It went back to normal....for about 8 hours. Then it was slow again. So we pulled another DIMM. Its been fine for weeks at this point, but we'd like to put the issue to bed. So we put back in the last DIMM we pulled (presumably bad) and are hoping the machine goes belly up again. 

##Shopping List
So I'm using the [Firebase TodoMVC](http://todomvc.com/examples/firebase-angular/#/) as a base for a shopping list app for me and my wife. We find the iPhone shared reminders to be very unresponsive and annoying. 

####CSS
Dropping the app directly into my github pages based site works like a charm. Everything works it just destroys the CSS already present. As a backend guy I'm taking this opportunity to learn some CSS. I'm using Chrome to inspect the example app and figure out how to get mine to look approximately like theirs. Pretty cool stuff. I think I'm 'getting' most of it. [Mine](http://blog.grummle.com/shoppinglist) 

So I actually had to pull out all the stops and get help from a friend to solve a spacing issue. 

####Jekyll
Jekyll interpolates (or tries to) {%raw%}{{ }}{%endraw%} so via [this](http://stackoverflow.com/questions/24102498/escaping-double-curly-braces-inside-a-markdown-code-block-in-jekyll) you escape it using &#123;%raw%}&#123;%endraw%}

## Commit your changes stupid
Made a Web.config change in production to fix a problem in our admin site. Of course my dumb ass didn't commit it so the next deploy wiped it out. Note to self don't be stupid......


##List Upload
####Email Column Required
Ok, so for figuring out if any of the columns are marked as 'email' I did a relatively hackey ```$scope.$watch``` on the model for columns.
 Then I used the ```$setValidity``` to basically mark the entire form as invalid. This works in this case because we don't need to mark individual form 
elements as being in an error state. I also put the override in there to just mark the form as good if we are using Md5 because 
the column mappings will be generated at post time if it is marked as an MD5 list. Come to think of it I don't even need that in there 
because the column selection portion is just skipped and that form's validity doesn't matter. Deleting some code....

```javascript
    $scope.$watch('options.columns', function (newValue, oldValue) {
        var emailExists = !!_(newValue).find(function (x) {
            return (x.element && x.element.name === 'email')
        });
        $scope.options.emailExists = emailExists;
        $scope.columns.$setValidity('emailColumn', emailExists);
    }, true);
```

####Column Uniqueness
Ok, now we actually have to use ```$setValidity``` how it was probably intended. I have to figure out which elements are in an error state and mark them by passing them in.
 So I have to find them, then get a reference to them. 

