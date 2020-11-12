---
layout: post
title: APIs and Spark Cores
---
<h2>Preparing for the Weather</h2>
Today I decided to start a new project that would let me practice using APIs.

<p>The background: For my job, I have to work outside occasionally. When it's a nice day, it's great to be outdoors. However, it helps to be prepared when the weather takes a turn for the worse.<br>
The idea is simple: Look inside the calendar to see if I'm scheduled to go anywhere today, then notify me of the day's weather for that location listed. This will then make working outside a bit more tolerable if I am prepared for it.
This also gives me a good way to get my hands dirty exploring APIs with Python, particularly the Google Calendar and Wunderground weather APIs.</p>

<p>Setting up took a bit of legwork. The Google API process was slightly cumbersome as you had to go through the process of creating a new client ID, but as long as you stuck with their quickstart guide https://developers.google.com/google-apps/calendar/quickstart/python, you'll be in good hands. The Wunderground API process was simpler, as all you have to do was register your account, and grab your API key.</p>

<p><b>Friendly reminder:</b> NEVER share your API keys! If you're pushing your project to your Github public repository without removing the API key, things may not turn out well for you. One way to prevent that is to store your API keys in a separate file (such as .yml), and put that extension in your .gitignore. This way, you can continue to work, commit, and push your files to your repo without compromising your keys so long as you don't accidentally add them to the actual program file you're working on.</p>

<p>After the keys, the setup was pretty straightforward. Copy the examples provided and run them and you can see the immediate results. The Google Calendar example called the service.events method to provide the next ten events listed in your calendar. The Wunderground API lists the current temperature in Cedar Rapids, which today is 66.2 degrees using the geolocation method.

All in all, it's pretty straightforward. Next time, I'll work on combining the two of them together. For the future, the idea is to have the program running on my Raspberry Pi, which I plan to have as the brains for my future Internet of Things system.

<h2>Spark Core</h2>
<p>What's also exciting was setting up the Spark Core today by what used to be known as Spark, which has recently rebranded itself as Particle.<br>

![Alt text](https://raw.githubusercontent.com/quickbrownfox319/quickbrownfox319.github.io/master/images/20150628/spark-core.jpg)</br>
For the uninformed, the Spark Core is a Wifi capable microcontroller. The programming is the same as the Arduino, making the transition very easy. However, what sets the Core apart is the beautiful web interface they have set up. Within minutes, I was able to get connected to the Wifi network and begin flashing code over the air so to speak. Their Android app is also very simple yet beautifully designed, with the ability to control the individual pins. You can see their website [here](https://www.particle.io/?redirected=true).
This device will be another component to my planned IoT system.

That's it for today. Hopefully next time I will have more substantial progress going on.
