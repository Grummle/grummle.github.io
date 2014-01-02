---
layout: blog 
title: "RabbitMQ for Errors in .Net" 
---

So we've been trying to dip our feet into messaging for some time now. We have one 'module' in our application that currently uses RabbitMQ to pipeline message across several different languages and processes. Overall that experience has been very positive.

<!--more-->

##Errors...Lots of Errors
Error handling in our app is rudimentary. Actually is sucks. Unhandled exceptions in .Net and classic ASP are written to a local XML file and the windows Event Store. Then an email is sent to a group address. Obviously ideally speaking there should be very few errors, but sadly this isn't the case. Of course when we break something it only compounds the problem. 13K+ emails is not a good way to be alerted to problems in the application (especially when gmail's web interface only lets you delete 100 at a time)

##We can rebuild it, we have...blah,blah
So lets dump them in a fanout or routeing exchange and do some fun stuff. Initial things I'd Klike to do:
- Dump them into [ElasticSearch](http://www.elasticsearch.org/)
- Alert on error rate
- Alert on configurable paramaters
- Not use .Net
