---
layout: post
title: Environmental Gas Sensor Using the Raspberry Pi & Arduino - Part 1
---

Hey all! Apologies all around for being so late, but a lot of exciting stuff has happened in the...half year since my last post, so I will try to fill in the gaps.
This series of posts is going to cover a project I did for a company where I made a remote environmental monitoring system using a Raspberry Pi, Arduino, and lots of Internet help.

## Environmental Gas Sensor (High Level) ##

### Background ###

As an environmental engineer, I helped design and construct a Soil Vapor Extraction system (SVE) that had petroleum semivolatile organic compounds (SVOCs). Rather than bore you with the dreadful details of fluid dynamics, chemistry, and - worst of all - environmental regulations surrounding its design, I'll tl;dr it for you. Essentially, it's a giant vacuum cleaner with a carbon filter at the end. However, just like your Brita filter or the nasty air filter you have in your car, the carbon's adsorption properties deplete over time as more stuff (read: semivolatile organic compounds) collect on it. And just like your woefully undersized car filter, this carbon filter would allow more 
crap to pass through it, effectively rendering it useless and contaminating the air with hazardous volatiles.
Obviously, we didn't want this to happen, so I would drive out to the site once a week with a photoionization detector and write down measurements. By comparing the initial influent concentration with the effluent concentration, we were able to determine how effective the carbon filtration system was working. This method worked, but had a few important limitations.

1. Delay Between Readings

   Because we were not doing real-time readings, there was the potential for the filter to hit breakthrough inbetween readings, leaving several days of unfiltered contaminations to pass through. This could be a concern as it exposes people working or living nearby to the chemicals that were meant to be filtered.
	
2. Labor and Time Intensive

   The most expensive cost to the client was the time and labor cost as compared to material costs. Typically, it took an hour's drive each direction to the site from the office, and the work itself took approximately 10-20 minutes to complete. On top of this, if there was a field staff of only 4-5 people the workload had to be shifted in order to allow a single person to take these readings.

3. Sparse Readings

   Tangential to Point 1 earlier. Because data points were taken once a week during site visits, there wasn't a clear visual of how the system responded during various environmental changes such as climate, which could have a potentialy large impact on the system's efficiency and performance.


### The IoT Solution & Future ###

The Internet of Things (IoT) is a a growing movement to network devices to the Internet for remote monitor and control. For our situation, the ideal use case would be a remote monitoring system where users could observe the health of the carbon filters and begin the process of purchasing new filters prior to breakthrough to reduce down-time. With more collected data points, users could also determine the most ideal environmental conditions based on performance and optimize the system to conserve labor, materials, and energy needed to run the system. This concept could be extended to remotely control the system. The blower and valves to extraction wells could be operated to adjust the flow rates to a level of precision and control not feasible through manual operation. That leaves the question of what if the entire operation could be automated without any user interaction? Using some programming and machine learning, the user would only need to remotely monitor and adjust the system to "teach" the system how to respond to conditions during startup. Eventually, the system could perform autonomously, adjusting controls in response to environmental factors and even preventively through weather reports and order new carbon filters when it detects the beginning of breakthrough.

Getting to that point on my own is still in the future, but I was able to develop a remote monitoring system using a Raspberry Pi SBC, an Arduino microcontroller, and Python to write SVE exhaust data to a Google Doc sheet.
So stay tuned for the next post where I will go into more of the technical details of what I used and how I made it!