---
layout: blog
title: "The day after...."
---

Time to vomit out some more basic todo list type stuff to see if we can keep Mr. Dillon....SQUIRREL! on track.

<!--more-->

##TODO

* ~~Deposit Norah's xmas monies~~
* ~~Post 13" MBP Retina to Craigslist~~
* ~~order brake pads for my bike~~
* ~~Install new firmware for SSDs in RMSQL3~~
* ~~Get DiffAll and Beyond Compare setup on new machine.~~
* Review and push Current list upload stuff to master
* take a look at filtering Path to long errors.
* take a look at reproducing Angular loop on login page 
* figure out strikethrouhg jekyll/github pages

## Post Mac Book
Posting this first. Quickest task I have probably and it'd be good if it got up there as quick as possible.
After some consultation with my resident pricing expert and checking out whats available already end up [here](http://chicago.craigslist.org/chc/sys/4819845581.html) 
  
Ah Craigslist how we love you. 


## Brake pads
So figure out this morning that the sound coming from my brakes where the screws under the pad rubbing on the rim. Doh. Amazon Done.  


## Update firmware on SSD Drives in RMSQL3
So we bought some SSD drives from ebay after talking to the guy and him telling us they are dell legit. Well the dell controller is all 'WTF are these things' So we are now jumping through
the sellers hoops to see if yes inded they are legit or just bullshit.

So I've got to download a bootable utility and see if I can get them updated. I tried downloading this to the actual server and it looks like it creates
a bootable usb drive. Problem being that it wouldn't recognize the usb drive that I'd left in the server. So we'll try it here locally and see if we can't get it to create
the drive at least. Then I can run it over to the datacenter and give it a whirl. At this point I really don't have very high hopes this is actually going to work.

Downloading........ZZZZZZZzzzzzzzz
Ok no matter what I do the MakeUsb utility isn't playing ball. It won't find the usb key.  
Ok so I used [rufus](https://rufus.akeo.ie/) to make a usb key bootable and copied the files onto that.
Just to make sure it works before I walk all the way I rebooted the mac with the key in. We are golden.

Ok be back in a bit after lunch, firmware install, and hitting the bank.

Ok, Norh's richer then I am....  

I tried to use the lifecycle controller to update the drive firmware, but no luck. Did update quite a few other things though. Took FOREVER.  
After that finished I got the drive update going. Only 2 of the 12 SSD drives I care about needed to be updated.
All the drives still show up with a caution symbol in the server manager. So waste of my time I think.

## Get DiffAll and Beyond Compare going.
DiffAll is a script that works with git that creates two temporary folders one with the what ever revision you givit and the current master revision. So it lets you use tools like beyond comapare 3 to compare the entire folder structure of your project. Its in a word awsome. After working on a branch and before I merge in I always pull pull up the full diff this way and make sure I'm not shooting my past self or someone else in the face with my commit.  

Anyway I had it all setup on my old behmoth dell but not on the new MBP so time to pull that stuff off and see if I can get it working properly. And yay its done. Just had to copy the scipt over from the old machine to git/bin

##List Upload

So after some finicky rebasing I've got the branch updated with master.
