---
layout: basic
title: "Baby Pictures" 
---

So since Norah was born I've been wrestling with finding a way to have all of our pictures of her available to us and our family/friends, while also giving our family and friends a way to send us their pictures of Norah. The other requirement is that they have to be "backed up".

##Keep Calm and Back It Up:
 I lost about 5 years worth of photos to a hard drive failure several years go. Murphy was a real ass about it, he waited to fail the hard drive till the day that I had the hard drive I intended to use for software RAID1 (Mirror) in hand.
 
###Dropbox: The Contender
My wife and I currently use Dropbox. I remap the default folders on her Mac and mine to point to folders in our Dropboxes. This seems to work suprisingly well. There are all the other side benefits of the files being available via web and iphone as well. I also already pay for this service so if I could keep my baby picture solution in dropbox that'd be *great*...

###Quick and Dirty
As a stop gap I used a app called [DBInbox](http://dbinbox.com/) to let people upload images to us and I did a simple share in Dropbox so that people could see the images. The issue I ran into with DBInbox is that it creates a folder in the App folder of Dropbox and DB doesn't allow you to share those folder. So My wife doesn't have direct access to the photos. Ideally she'd be able to drag and drop files on her computer to the shared folder and have them be visible to everyone.

###Everything looks like a nail...
One of the current hammers in my tool belt is node.js. DB has a javascript api.... you see where this is going. Insanely complicated Rube Golberg machine coming right up. My initial concept was to allow uploads to a site that would use node.js and the DB JS SDK to upload them to dropbox and serve them up using DB public url's. I figured it'd be pretty cool to take the byte stream that came in from friends and family and pipe it to DB. Alas I couldn't find a good way to serve media out from dropbox. They use some goofy scheme for their public pages which I'm guessing is to stop just that sort of thing.

##R&D: Rip off and duplicate

###Dropbox
So I stared poking around to see what was out there and came across [dropbox-sync-js](https://github.com/Phlaphead/dropbox-sync-js) by [Phlaphead](https://github.com/Phlaphead). Its basically an implementation of the dripbox application in javascript. From a quick perusal of the code it seems well built and easy to follow. It has a couple quirks that need to be addressed and I'm hoping to get do a pull request to fix some issues it has with synching non-app folders. 

So now I've got my synching strategy worked out.

###Thumbnails
Pictuers, lots and lots of pictures. Large Pictures. Need thumbnails. I don't want to write it.... So I was happy when I stumbled on [quickthumb](https://github.com/zivester/node-quickthumb). And...done.

###Exif
I have a crazy idea that I'm going to stash the majority of the data I care about concerning these pictures in the exif data. Stuff like data uploaded, and by who. I *believe* that I can do this and it looks like the project [Exiv2node](https://github.com/dberesford/exiv2node)

##Family and Friends as Users
So the point of this exercise is to make an easy place to see and put all pictures of Norah. Like anything else if it sucks no one will use it. Well the stop gap site that I setup sucks. I've already seen people stop using it. The biggest complaint I've seen is that dropbox organizes the images based on their names. Which as you can imagine is just about the most useless way you could arrange images. 
-  Photo 1.jpg		
-  Photo 2.jpg		//oldest
-  DSCIM_11232.jpg	//newest  

Like most things these days it seems like my 'users' want a quick fix and to go on with their day. So if I could do things like show them the new pictures since their last visit that'd be *great...*

###Another Hammer
I've goat a real man crush on AngualrJS at the moment. Its really a little creepy, but I do like me some good tools. So I'm figuring I'll get me some json data about the images available and use Angular to present it in what ever way the user wants.

##Multiple File Upload Sucks
Support for multiple file upload in browsers sucks. I've looked at several different solutions and I've been horrified by some of the stuff that they have to do to make it work. I will not tolerate Flash. No Sir I Don't Like It. I looked at [jQuery upload](http://blueimp.github.io/jQuery-File-Upload/) initially but I really didn't like the vibe I got from it. So I'm giving a [dropzone](http://www.dropzonejs.com/) a go.

##All together now
So Lets bash all these together and see if we can get anywhere.

