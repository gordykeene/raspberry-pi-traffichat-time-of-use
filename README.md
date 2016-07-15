# Time of Use Plan Indicator

This Time of Use Plan Indicator is a [Node-RED][nodered] Flow for the [Raspberry Pi][raspberrypi] and [Ryanteck][ryanteck] TrafficHAT board that indicates active and upcoming [Time-based pricing][time-based-pricing] electric rates.

### Background

My local power company offers several pricing plans that offer discounted rates for most of the day in exchange for accepting higher rates during other times of the day. This plan can work to everyone's advantage when users minimize power consumption during [peak times][srp-tou]. 

As a household, we have already done many of the obvious improvements (LED lighting, pool pump timer only runs at night and solar hot water). Yet two large consumers remain: the oven and the clothes dryer. The oven is mostly self-correcting because it makes the house uncomfortably warm, however the clothes dryer is a little tricky and this project strives to simplify it. 

It's not just a question of comparing the current time to the active winter or summer plan. It also involves ensuring power-consumer equipment isn't started such that it continues to run well into the next on-peak period.

### How it Works

The goal is to blink the green LED during off-peak times and blink the red LED during on-peak times. Also, during the hour immediately prior to the start of an on-peak period, the yellow LED will also blink.

The instructions to users are simply only use the dryer when the green LED is enabled. If the yellow LED is also enabled, ensure the load will complete before the top of the hour.

# Required Equipment

### TrafficHAT

The Ryanteck [TrafficHAT][ryanteck-traffichat] provides green, yellow and red LEDs, as well as a buzzer and a momentary switch. For this application, neither the buzzer nor the switch are necessary.

Raspberry Pi GPIO pins each have three names (for more information about that, see [Appendix 1 here][raspberrypi-40pin]). This table shows how the pins on the TrafficHAT are arranged:

| I/O       | GPI O               |
|-----------|:-------------------:|
| Green     | 15 - GPIO3  - BCM22 |
| Yellow    | 16 - GPIO4  - BCM23 |
| Red       | 18 - GPIO5  - BCM24 |
| Button(*) | 22 - GPIO6  - BCM25 |
| Buzzer    | 29 - GPIO21 - BCM5  |

(*) Suggest using the Raspberry Pi's internal pull up resistor.

### Raspberry Pi

A [Raspberry Pi][raspberrypi] model with [40 pins][raspberrypi-40pin] is also required. Like many others, I've not been able to get my hands on one to test, however, this should be a reasonable application of the [Raspberry Pi Zero][raspberrypi-zero].

Additionally, all the normal Raspberry Pi accessories (SD card, 5Vdc supply) will be needed. 

# Software Setup

The ultimate goal for these instructions is to end up with a [Raspberry Pi][raspberrypi] running [Node-RED][nodered]. The following outlines the process I used to setup the software for this application. There are many ways to accomplish this, and many users may already have a Raspberry Pi already setup and ready to go. For those who might not, or want a quick start guide, this should get you there.

### Setup Raspbian Jessie Lite

1) Download the latest [Raspbian Jessie Lite][raspbian-jessie-lite]. The non-lite version can be used as well, if you have the need to use a display. Personally, the thought of have to use a video cable on this doesn't really fit the Nixie aesthetic.
2) Write the image to an SD card. This process is well documented elsewhere, here are some examples for [MAC][setup-mac] and [PC][setup-pc].

### Setup Node-RED

The previous step shows how to open a terminal shell into the Raspberry Pi. Continuing in that shell, the following instructions will get Node-RED started.

Depending on the path that lead to this point, it is possible Node-RED is already installed. Running the following command is a simple way to find out:
```sh
node-red-stop && node-red-start
```
If a message indicating `Start Node-RED` appears, then Node-RED is already installed and the rest of this section can be skipped. However if a message indicating `command not found` appears, then Node-RED will have to be installed. The following instructions outline one way to accomplish that.

Instructions appropriate for a Raspberry Pi are found on the [Node-RED Raspberry Pi][nodered-raspberrypi] getting started guide. In many cases, it just boils down to pasting this block of text in to a terminal window.

```sh
node-red-stop && \
sudo apt-get update && \
sudo apt-get install nodered && \
node-red-start
```

### Opening Node-RED

Once everything is assembled and Node-RED is running, you should be able to open it from a web browser. For a default installation, that will be `http://<raspberry-pi-IP-address>:1880/`. If that doesn't work, check that you have entered the correct IP address for the Raspberry PI. Also, inspect the `setting.js` file and locate the `uiPort` entry. Either of the following commands should help.
```sh
cat /home/pi/.node-red/settings.js | grep uiPort
```
or
```sh
sudo nano /home/pi/.node-red/settings.js
```

### Set the time zone

As this application concerns time, it is appropriate to ensure the local time zone is correctly established. The following command should guide that process. Once complete, it may be necessary to restart Node-RED or reboot the Raspberry Pi for the setting to take affect.
```sh
sudo dpkg-reconfigure tzdata
```


### Using TOU in Node-RED

With Node-RED open and an empty Flow tab visible, select the hamburger button in the upper right to open the menu. From the menu, select Import > Clipboard. Open the `time-of-use-traffic-hat` file, then select and copy it's contents. Return to Node-RED, paste the text and select Ok. Once the new Flow appears under the mouse, move it towards the upper left corner and click once. Select Deploy.

If your hat has a buzzer, it will chirp once when the flow is started.

# Customization for Other Electric Pricing Plans

Most likely your power company's pricing plan is will be different then mine, so feel fork this project and make it your own!

The meat of the project exists in the `Initialize Flow` node where it creates and adds the `findTouState` function. The primary products of this function are returned as the two boolean attributes `isOnPeak` and `isPeakAboutToStart` of `msg.payload`.

As date related calculations are known to be a significant opportunity for introducing errors, unit tests are your friend. Update the `findTouState unit tests` node to reflect changes to the Time Based Pricing structure, then trigger the unit tests. A summary of the results will be displayed in Node-RED's debug tab. 

![Node-RED unit test](/images/tou-unit-test-node.png)

### Holidays

My utility does waives the on-peak rates for certain holidays. Initially I considered a solution for calculating when we observe these holidays, but the rules are non-trivial and would have required significant testing. Then I looked into calling a web service, but initially I couldn't find any reasonable options, plus that would have required all instances to have a continual connection to the Internet and possibly some more complicated than necessary caching.

Ultimately, the simplest solution won out: just precalculate these dates for the expected lifetime of the application. Currently, up until the year 2030 the application will work as expected. In 2031 and beyond, the application will treat holidays as peak days.

The specific holidays used by your utility may differ than the ones embedded into this flow. To make changes, open the Initialize Flow node and locate the tou.observedHolidays declaration.

Holidays are gleaned from the excellent [Holiday API][holiday-api] webservice with calls similar to the following. (It is my understanding it may now be required to request and add a API key.) 
```
http://holidayapi.com/v1/holidays?country=us&year=2016&month=07
```

### Notes
Before saving the .json file, it was run through the excellent [JSONLint][jsonlint] tool. I have found doing so makes identifying changes from the repository significantly easier than raw Node-RED output.

### Version
1.0.0

### Picture
![Node-RED driving the display hardware](/images/tou-sample1.jpg)

### Tech
This project benefited from other projects:

* [Dillinger] - Made with "Dillinger, the Last Markdown Editor ever."

[//]: # (http://stackoverflow.com/questions/4823468/store-comments-in-markdown-syntax)

[ryanteck]: <https://ryanteck.uk/>
[ryanteck-traffichat]: <https://ryanteck.uk/hats/1-traffichat-0635648607122.html>
[dillinger]: <http://dillinger.io/>
[dillinger-gh]: <https://github.com/joemccann/dillinger>
[holiday-api]: <https://holidayapi.com/>
[jsonlint]: <http://jsonlint.com/>
[nodered]: <http://nodered.org/>
[nodered-raspberrypi]: <http://nodered.org/docs/hardware/raspberrypi>
[raspberrypi]: <https://www.raspberrypi.org/>
[raspberrypi-40pin]: <https://www.raspberrypi.org/documentation/usage/gpio-plus-and-raspi2/>
[raspberrypi-zero]: <https://www.raspberrypi.org/products/pi-zero/>
[raspbian-jessie-lite]: <https://www.raspberrypi.org/downloads/raspbian/>
[setup-mac]: <https://www.raspberrypi.org/forums/viewtopic.php?t=74176>
[setup-pc]: <http://www.circuitbasics.com/raspberry-pi-basics-setup-without-monitor-keyboard-headless-mode/>
[srp-tou]: <http://www.srpnet.com/prices/home/tou.aspx>
[time-based-pricing]: <https://en.wikipedia.org/wiki/Time-based_pricing>
