---
layout: blog 
title: "Ninja Blocks I am disappoint" 
---

The HVAC system at our current house *sucks*. The place is three stories and the furnace doesn't heat the building evenly or even linearly for that matter. I intended to use some Ninja Block components to rig up my own custom thermostat but that was a pipe dream.

<!--more-->

##Problem
We have a new born who's 4 months old. So keeping her room a stable temperature is a priority. However a standard thermostat does a pretty poor job of that (its not in the room, how does it know the temp). Also the design and implementation of this places HVAC setup is less then optimal. The HVAC does not heat the floors with any simple easily discernible relationship to each other. Adding two degrees to the thermostat on the first floor can equal 10-20+ degrees for the second and basically makes the 3rd floor unlivable.

##Real question
I think the relationship that most houses have with heating/cooling is much simpler then our current setup. So without zoning what can we do? If we knew the temp in each room and told the thermostat which room we cared about (living room during the day, baby's room at night) the thermostat would have an easier time of it.

##Enter [Ninja Blocks][nb]
I already had a Raspberry Pi and when I saw that [Pi Crust][picrust] I thought it'd be a perfect fit. The crust allows a raspberry pi to interact with the 433MHz class of devices that Ninja Blocks offers. One of those devices was a [$15 wireless battery operated thermometer/humidity sensor][tempsensor]. Perfect! So for $184 I got 1x [Pi Crust][picrust], 6x [temp sensors][tempsensor] and 3x [motion sensors][motionsensor]. I was in heaven. It took awhile for them to show up, but I was *uber* excited to play with them. I've always figured that I was going to have to dig through the source in order to make my solution internal (no reliance on Ninja Blocks cloud service) but to make sure everything worked I setup as directed by NB. Ran into an issue, but easily fixed with help from the forum and then we hit 'the' snag. Temp sensor range was about 3 foot from the pi.

###Pi Crust
![pi crust](/images/picrust.png)  
My understanding is that the Pi Curst is an ardunio compatible board that can be used with the Raspberry Pi. Basically mating an arduino on top. This allows you use arduino shields and also the NB 'cape' to provide 433MHz capability to the Raspberry Pi. The problem seems to be that the antenna is simply a wire thats coiled. The antenna is also the wrong length. 433MHz 1/4 should be 16.5cm but the one I received was 17.5cm. Apparently this makes a difference and seems to have in testing as well. I was able to get 15ft or so reliably after trimming my antenna to the specified length. Still wasn't reliable or useful past that distance however. The direction that the sensor was pointed also seemed to matter significantly.

###Temperature Sensor  
![temp sensor](/images/tempsensor.jpg)  
The sensors themselves are small sturdy feeling attractive units. They display temp and humidity data on a small LCD. I liked them immediately. However it appears that of all the components available on NB's website the temp sensors have the worst range. Multiple complaints on the forums. Most people had difficulty getting the suggested 10m range. I initially had an issue getting 3ft. 

#Seriously?
Why would you release something like this? How is it useful? The forums seem to indicate that the other devices that NB sells have much improved range, but the thermometers are what I need to use. So I've posted the NB devices for sale and hope to recoup my expenditure. The Proteus Wifi Thermometer looks promising, though I'll have to reduce the number of sensors I was planning to deploy.

[nb]: http://ninjablocks.com/ "Ninja Blocks"
[picrust]: http://ninjablocks.com/pages/picrust "Pi Crust"
[tempsensor]: http://ninjablocks.com/products/temperature-and-humidity-sensor "Temp Sensor"
[motionsensor]: http://ninjablocks.com/products/motion-sensor "Motion Sensor"


