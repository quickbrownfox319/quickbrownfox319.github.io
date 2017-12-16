---
layout: post
title: Projects Updates
---

<h1>LocationWeather Update</h1>
Good news everyone! A new blog post yay! -soft cheering-
<p>I've been pretty busy, but I am happy to say that it was productive business in the past several months since my last post. I've "finished" my LocationWeather project a while ago to a mostly-functional state, where it pushes updates to my devices via Pushbullet. Definitely some hiccups along the way, but I am pretty pleased with its result being my first project using APIs.</p>

<h2>Functionality at a (very) High Level</h2>
<p>Essentially, what it does is it looks a few days into the future in my Google calendar and finds all events. It then takes those event locations and searches Wunderground for that location's forecast in two days (Just realized I forgot to actually implement adding the event date to query Wunderground's forecast date...). Then, it pushes the forecast as a Pushbullet note to the user account. The script is hosted on a raspberry pi 2, and is run via a bash script and cronjob every day at midnight.
</p>
![Alt text](https://raw.githubusercontent.com/quickbrownfox319/quickbrownfox319.github.io/master/images/weatherlocation-1.jpg)


<br>If you'd like to see the entire code, you can visit the page 
<a href="https://github.com/quickbrownfox319/LocationWeather/blob/master/locationweather.py">here.</a>

<h2>Bugs, Tweaks, and Other Things to Work On</h2>
Although I consider this a success, there are a few things I'd like to improve upon as I get better at working with APIs and coding.

<p>
<h3>1) Consistent use of libraries</h3>
For the moment, I have left Google's example code intact implementing oauth2. I am ashamed to say that it's pretty ugly and redundant right now, as I'm using both urllib2, httlib2, and requests library all at the same time, mostly for the fact that Google's oauth2 uses urllib2/httplib2, while the rest of my code uses the requests library (yes, I am that afraid to touch it right now, please don't laugh too hard). Definitely one of the things I want to change in the next iteration of this project.

<h3>2) Querying Event Date from Wunderground</h3>
So like I mentioned earlier, I forgot to query Wunderground for date of the event. At the moment, I have a hackish job of looking only for the forecast for tomorrow for that event location, while in the meantime I pull events in the upcoming week. What I just realized this does is it's doing two separate operations that are unrelated. One operation is pulling events for the upcoming 7 days, while the other is querying tomorrow's forecast for those locations. What this does then is give inaccurate event forecast until the event is tomorrow. I guess one quick solution is to change the events I pull from 7 days to just tomorrow, which will align the two separate processes. However, the better way to do it is to figure out how to parse the JSON response for that specific event's forecasted date. It shouldn't be that much work, but something I'll need to keep in mind.

<h3>3) Fix Location</h3>
At the moment, the way I get the event's location is by using regex to find the zip code in the Google Calendar event description, then plug that into Wunderground's URL endpoint for searching in 'geolookup'. However, this is not a very good way to do it as I've found. I assumed that when I typed in my address in that section, Google will automatically fill in the zip code for me. This is not always true, which as a result calls the default location I set as backup.

I'll be honest here - I have no idea what I'm doing when it comes to regex. In order to get this simple zip code regex function to work, I had to copy someone else's regex from Stack Overflow and finangle it to work. How it works right now is beyond me other than a witch's brew of spells and poorly understood code (The same could be said about Google's oauth2 function I mentioned earlier). To sidestep this issue (momentarily of course!), I think a more robust way would be to pass the address to Google's Location API and pull usable location data that way. Hacky? Yes. But if it works, it works...until I force-feed myself a book on how to use Regex. 

<h3>4) Reduce Frequency of Pushes</h3>
Remember how I said that my rpi2 pushes me these notifications at midnight every day?
I'd like a way to not get so many of these. I'm not exactly sure how I'll implement it, but it'll mean that somehow I'll be able to dismiss the notification, and that should send a response to my code which would say which event to ignore. Seems pretty complicated, but I guess I'll find out. Not a high priority for this project at the moment.

<h1>Wrap Up</h1>
That's about it for the LocationWeather updates! Right now it's sitting on my desk pushing these weather notifications to me, which is pretty nice and I'm pretty pleased with what I have so far. It's not the prettiest in terms of functionality or cleanliness, but it's working to an extent and I built it myself for once, which I am proud of.

<h3>Gas Monitor Mini-Update</h3>
In other news, I have also made progress with my remote gas monitor, which currently is being hosted on a RPi2 and Arduino Nano.
</p>

<br>![Alt text](https://raw.githubusercontent.com/quickbrownfox319/quickbrownfox319.github.io/master/images/gasmonitor-setup.jpg)

<p>
I have tried using the Particle Photon which has built in wifi, but due to complications related to the site, I had to scrap it. The gas monitor so far uses the pi to read in serial data gathered by the Nano, and stores it as a csv locally as well as writes it to a Google Sheets doc, which I thought was pretty nifty.
</p>
<br>![Alt text](https://raw.githubusercontent.com/quickbrownfox319/quickbrownfox319.github.io/master/images/gasmonitor-pi.jpg)
<br>![Alt text](https://raw.githubusercontent.com/quickbrownfox319/quickbrownfox319.github.io/master/images/gasmonitor-nano.jpg)
<p>
There are still a lot of rough edges that I need to work on regarding connectivity, but it kind of works as intended. You'll have to wait for my next post about that one though, so hold on to the edge of your seats! Hopefully the next post won't be in another several months.  
</p>
