---
layout: post
title: Environmental Gas Sensor - Part 2
---

# I'm Back!

Sorry it's been a while. I've since been back in school to focus on electrical and computer engineering, and I have learned a LOT, so stay tuned for posts that will be more relevant to hardware, programming, and security! However, I'll finally cover the inner workings on the gas sensor I developed back in 2015. I haven't really worked on it in a long time (2-3 years as of writing this post), so some packages may be out of date. I was also a very newb programmer at the time so some things are definitely not as Pythonic or programmatic as they should be. However, here's what I did and if you have an improvement, feel free to email me or implement it in your project.


# The Details
## Basic components

I tried to keep this simple using off-the shelf hardware you can find on Amazon or through a local electronics store such as Microcenter.

* Raspberry Pi (any version should be okay)
* Arduino Nano
* Sainsmart MQ135 sensor

The Raspberry Pi can be thought of as the "brains" or the controller of the system. It handles the script running the data collection, API calls, connectivity, and storage of data. I would have used the Raspberry Pi alone, however there's a problem. Raspberry Pi's have GPIO pins which are digital in nature. The MQ135 sensor used has analog output of data. This means basically that digital data return `1` or `0`, whereas analog is a continuous stream of data. The difference between digital and analog data is the difference between finite and continuous data, in which digital data is composed of discrete values, and analog are a continuous set of values. One way to visualize it is that digital is like a set of steps, in which you can be on any step between the top and the bottom of the stair, but you can't be standing inbetween those steps. Analog is more like a ramp, where you can be at any elevation between the top and the bottom of the ramp. You can read more from this [Sparkfun post](https://learn.sparkfun.com/tutorials/analog-vs-digital) explaining the difference. To resolve this issue, I used an Arduino Nano for its analog pins. Then using the Python `pyserial` library in my script, I was able to listen and extract data via USB connection.


## Code
### Overview
The Raspberry Pi runs a combination of different scripts to make sure everything works. A Python script using the [gspread wrapper](https://github.com/burnash/gspread) writes collected data from the MQ135 sensor via the Google Sheets API. Once establishing a connection to Google Sheets, it creates a connection to the Arduino via PySerial. The Arduino has its own script where it takes a measurement every few minutes. Finally, data is written to a CSV file that's stored directly on the Pi itself. The Pi also logs any errors that might have occured for later debugging. The way it's accessed is through SSH once the Pi establishes a connection.




### _`pi_log_data_py2.py`_

This is the Python script used to run everything including serial connection, API calls, and data collection.

```python
import os, sys, traceback
import time as t

import yaml
import simplejson as json
from sseclient import SSEClient
import gspread
from oauth2client.client import SignedJwtAssertionCredentials
from datetime import datetime
import requests
import serial

#rpi serial port with arduino
serialPort = '/dev/ttyUSB0'

errorLogFile = "error_log.txt"
filename = "data_log.csv"

# Manipulate data in Google Sheets
def google_sheets(gc, data, date, time):
    try:
        wks = gc.open("GasSensorData").sheet1
        print "wkb okay"

        dataToAdd = [date, time, data, "{0} {1}".format(date, time)]
        wks.append_row(dataToAdd)
    
    except Exception as e:
        print "Google Sheets error, ", e
        exc_type, exc_value, exc_traceback = sys.exc_info()
        lines = traceback.format_exception(exc_type, exc_value, exc_traceback)
        error_log("Google Sheets", lines)
        raise e

def error_log(type, lines):
    with open(errorLogFile, 'a') as logOutput:
        logOutput.write("{0}\n{1}\n{2}\n\n".format(datetime.now(), type, str(lines)))

def main():

    while(True):

        # Google oauth
        try:
            #Google sheets API via gspread wrapper
            json_key = json.load(open('client_secret_work.json'))
            scope = ['https://spreadsheets.google.com/feeds']
            credentials = SignedJwtAssertionCredentials(json_key['client_email'], bytes(json_key['private_key']), scope)

            gc = gspread.authorize(credentials) #create oauth object
            print "Google oauth2 okay"

        except Exception as e:
            print "Google login error ", e
            exc_type, exc_value, exc_traceback = sys.exc_info()
            lines = traceback.format_exception(exc_type, exc_value, exc_traceback)

        #serial connection to arduino
        serialConnection = serial.Serial(serialPort, timeout = 2.0)
        
        try:
            while True:
                #get serial data from arduino
                data = serialConnection.readline().strip()
                if data:
                    #write and format data to Google sheets
                    date = datetime.now().strftime("%m/%d/%y")
                    time = datetime.now().strftime("%H:%M:%S")
                    print data, date, time
                    with open(filename, 'a') as outputFile:
                        outputFile.write(date + ',' + time  + ',' + data + '\n')
                    google_sheets(gc, data, date, time)
                    t.sleep(30)

        except Exception as e:
            print "Sensor Error: ", e
            exc_type, exc_value, exc_traceback = sys.exc_info()
            lines = traceback.format_exception(exc_type, exc_value, exc_traceback)
            error_log("Sensor", lines)


if __name__ == '__main__':
    main()

```



### _`run_gas_log`_

Here's the Bash script run every 24 hours using a cron-job. This was necessary because for some reason, the Python script would crash about every 2 days. This script navigates to the folder where the main Python script is located, runs it until it crashes, log the error, and restart the script again. Not an elegant solution, but it worked well enough.

```bash
#!/bin/bash
sleep 11
cd /
cd /home/pi/gassensor/
until sudo python pi_log_data_py2.py >> error.txt; do
    echo "'pi_log_data_py2.py' crashed with exit code $?. Restarting..." >&2
    sleep 1
done
cd /
```



## _`nano.ino`_

This is the Arduino code that collects a reading from the MQ135 sensor about every 10 minutes. Now, this should be replaced by an interrupt rather than messing around with trying to time everything precisely. The Pi could remotely program the Arduino Nano using [Platformio](http://platformio.org/).


```C++
#define ANALOG_IN A0

int STATUS_LED = 13;

void setup() {
    
    Serial.begin(9600);
    
    pinMode(ANALOG_IN, INPUT);
    pinMode(STATUS_LED, OUTPUT);
    
}

void loop() {
    
    digitalWrite(STATUS_LED, HIGH);
    Serial.println(analogRead(ANALOG_IN));
    delay(2000);
    digitalWrite(STATUS_LED, LOW);
    delay(598000);
    //delay set for 10 minutes
}
```

## Images and Setup
I posted these images in a previous post (10/27/15), but should be of more value now that you have context behind them.

![Alt text](https://raw.githubusercontent.com/quickbrownfox319/quickbrownfox319.github.io/master/images/20151027/gasmonitor-pi.jpg "Raspberry Pi 2 controller")

![Alt text](https://raw.githubusercontent.com/quickbrownfox319/quickbrownfox319.github.io/master/images/20151027/gasmonitor-nano.jpg "Arduino Nano with MQ135 gas sensor")

# Final Thoughts
This project is by no means perfect or even very good. However, it means a lot to me because it allowed me to realize that I wanted to pursue a career in computer & electrical engineering. Working through how to code on hardware platforms to using APIs showed me how much there is to learn out there, and gave me the confidence and desire to be a part of this fast-paced industry. I believe everyone who is curious should pick up a project of their own to work on to completion and feel the same sense of accomplishment. So effectively, this project and post simultaneously closes my environmental engineering chapter and opens up my ECE chapter. I hope you join me as I dive into this perpetual quest of learning.