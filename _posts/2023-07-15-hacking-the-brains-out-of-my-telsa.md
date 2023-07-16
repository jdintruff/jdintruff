---
title: "Hacking The Brains Out Of My Telsa"
date: 2023-07-15
layout: post
image: ./images/PXL_20230713_204411068.jpg
excerpt_separator: <!--more-->
---

Once I upgraded to a more modern car a couple years ago, I became very curious about what data I could collect from it. I ended up creating a data pipeline to store and process video from its 4 cameras as well as vehicle telemetry (speed, acceleration, etc).

<!--more-->

This project started as a way to automatically copy and backup video footage if there was ever an incident while I was driving. It ended up turning into a data hoarding adventure that is still evolving. Currently there are two major components of this project: video backup/processing, and telemetry collection. 

### Overview and Hardware

The car was a 2021 Tesla Model S Plaid that I bought a couple of years ago. It was right after a "refresh" where a number of hardware changes were made to the new Model S so it was challenging to collect reliable information on where everything was.

The car has a USB port in the glovebox where users can plug in a flash drive to allow the car to store videos from its cameras. An example use case is if I honk my horn, it automatically saves the footage before and after that moment. 

There's a great project called [teslausb](https://github.com/marcone/teslausb) which runs on a Raspberry Pi 4 Model B that's plugged into that USB port and emulates a storage device. When the car saves video data onto this emulated storage device, it's saved until the device is within range of my home WiFi network, at which point it copies that footage to my backup server. It even let me set up a Discord webhook so that whenever it finished uploading footage it would send me a message/notification. Very cool project, simple to setup and configure, major props to [marcone](https://github.com/marcone) for his works.

This is what it looks like in my messy glovebox, please ignore the insanely expired registration.

![Glovebox computer](../../../../../images/PXL_20230713_204643923%20(1).jpg)


The telemetry side wasn't nearly as simple. At a high level, the Raspberry Pi connected to a bluetooth device that was plugged into my car's OBD-II port in order to gather, parse, and filter some of the unobfuscated telemetry that the car emits.

### Brain surgery

Unfortunately Tesla doesn't adhere to common standards for OBD-II ports so in order to get it to play nice with a standard [OBD-II CAN bus reader](https://www.obdlink.com/products/obdlink-mxp/), I needed to acquire or build a cable that remapped the pins on the interface. Ultimately I ended up buying one [here](https://evoffer.com/product/model-s-x-can-diagnostic-cable/). 

There was a great [YouTube video](https://www.youtube.com/watch?v=OKzPa2HdsvA) that showed me how to easily access the OBD-II port on my specific model of car which made things a lot less stressful (I was very anxious about disassembling my brand new car).

![Stuffing a bluetooth device into my car's brains](../../../../../images/PXL_20230713_204411068.jpg)



Once the car's brains were exposed I plugged the cable into the OBD-II port and plugged my OBDLink MX+ into that cable. I had a hard time getting my Raspberry Pi to consistently pair with that Bluetooth device every time it powered on and ended up having to disassemble the front console of my car multiple more times just to push the pair button, which made me feel like an absolute homunculus. Eventually after updating the firmware on the OBDLink MX+ it would consistently connect by running this sequence of commands automatically on reboot:

```bash
# Connect using the MAC address of my Bluetooth device
rfcomm connect hci0 00:XX:XX:XX:XX:XX 1

# Attach the line discipline
ldattach --debug --speed 500000 --eightbits --noparity \
  --onestopbit --iflag -ICRNL,INLCR,-IXOFF 29 /dev/rfcomm0

# Set up the CAN bus
ip link set can0 type can bitrate 500000
ip link set can0 up
```

After that I was able to dump the CAN bus to stdout by just running

```bash
candump can0
```

I ended up writing a Python script that parsed this data and emitted neatly formatted JSON that looked like this:

![Pretty JSON from the CAN bus](../../../../../images/Screenshot 2023-07-15 164358.png)

Just to validate the data I went on a drive and plotted some of the different values that were produced. This first one was the angle of the steering wheel in degrees:

![Steering wheel angle over time](../../../../../images/quick-drive-steering-wheel.png)

Next I plotted the speed, which defaulted to km/h. Notice that the speed is negative while I'm backing out of my driveway:

![Speed vs time](../../../../../images/quick-drive-speed.png)



### Future Work

Ugh there's SOOOO much more that I want to do here... 

Eventually I want to make a plugin for the aforementioned teslausb project that will display the vehicle telemetry alongside the video footage so that you can see the state of the steering wheel, speed, turn signals, etc. all synced up with the video. There's plenty of room for it in the UI too (see the screenshot below)

![teslausb UI](../../../../../images/Screenshot 2023-07-15 173520.png)

It would be better if I filtered out the events I don't care about when interacting with the CAN bus instead of filtering them out later. 

If I could serve this data through a web page that the in-car browser could access that would be cool. A couple barriers exist there, the big one being that the internet browser in Teslas is unable to access anything on a local network, it ***has*** to be on the open internet.

There's a project I'm working on right now related to doing automated license plate parsing on some of this footage but I need to do a bit more due diligence on that one before I can write it up in a public blog post.
