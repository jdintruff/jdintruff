---
title: "Getting Pretty Pictures From Geostationary Orbit"
date: 2023-07-11
layout: post
excerpt_separator: <!--more-->
image: ./images/GOES18_M1_FC_20230706.gif
---

I've always been super interested in space, and once I started getting into radio stuff I knew I'd eventually want to receive a signal from an interesting satellite. Fortunately, my tax dollars paid for a very expensive satellite called GOES-18, which is currently in geostationary orbit above me. I managed to pull high resolution images from this satellite and wrote a script to produce gifs out of those images.

<!--more-->

## Background

![GOES transmission architecture diagram](../../../../../images/Screenshot%202023-07-11%20063449.png)

The GOES series of satellites are environmental monitoring satellites that observe Earth using several wavelengths of light. They also observe the sun but unfortunately this data isn't as easily accessible (I have a future project planned around receiving that data). 

These satellites beam raw data down to one of three stations on the East coast of the US that are operated by NOAA where it's processed and turned into images and data that are sent back up to the satellite for retransmission. The processed images are transmitted by its High Rate Information Transfer (HRIT) system. This is the information I was aiming to capture.

## Hardware

![All the hardware involved in this project](../../../../../images/PXL_20230625_041218617.jpg)

Starting from the antenna side, I used a pretty standard 1-meter parabolic grid antenna but with a modified feedhorn and reflector. This was to optimize for receiving linearly polarized signals at \~1690 MHz. I haven't done much testing on the antenna itself to measure its characteristics and radiation pattern, but it seems to do the job.

That antenna was connected to a Nooelec SMArTee SDR. Its optimal frequency range was \~500 MHz to 1500 MHz but I found that it was more than capable of making it up to the frequency I needed.

Next I plugged that SDR into an LNA specifically tuned for this frequency range, the Nooelec SAWbird+ GOES which has a center frequency of 1688 MHz. I'm not entirely sure this was necessary as I didn't test the signal without it, but oh well.

This was all connected to a Raspberry Pi 4B with 8GB of memory running Ubuntu server.

## Software

There's a great project called [goestools](https://github.com/pietern/goestools) which does most of the heavy lifting. After installing and configuring `goesrecv` and `goesproc` I was able to run two commands to pull images.

This first one uses the SDR to stream data from the satellite.

```bash
goesrecv -v -i 1 -c /home/jack/goesrecv.conf
```

Running the first command will start outputting some information about the stream it's capturing, and it took a little fiddling to get the viterbi index below \~400 so that I could get an error-free stream. I downloaded an app to vaguely align the antenna and then dialed it in by watching the output of the `goesrecv` command and making small adjustments. 

![goesrecv output](../../../../../images/Screenshot%202023-07-11%20065130.png)

This second command processes the stream and parses out individual images.

```bash
goesproc -c /usr/share/goestools/goesproc-goesr.conf \
  -f -m packet --subscribe tcp://127.0.0.1:5004 --out .
```

This produces files in the following directory structure:

```bash
$ find . -maxdepth 3 -path ./nws -prune -o -print | sort
.
./goes16
...
./goes18
./goes18/fd
./goes18/fd/ch02
./goes18/fd/ch07
./goes18/fd/ch08
./goes18/fd/ch09
./goes18/fd/ch13
./goes18/fd/ch14
./goes18/fd/ch15
./goes18/fd/fc
./goes18/m1
./goes18/m1/ch02
./goes18/m1/ch07
./goes18/m1/ch13
./goes18/m1/fc
./goes18/m2
./goes18/m2/ch02
./goes18/m2/ch07
./goes18/m2/ch13
./goes18/m2/fc
./himawari8
...
```

The directories are generally of the form `<satellite>/<region>/<channel>/<date>` where:

`satellite` = the satellite the data originally came from

`region` = one of a series of region codes where `fd` is the full-disk image

`channel` = the channels representing different wavelengths of light (explained [here](https://www.goes-r.gov/mission/ABI-bands-quick-info.html))

GOES-16 is also known as GOES-East, and is responsible for capturing images from the East coast of the US. Himawari 8 is a Japanese satellite which captures data above Japan.

Once I had used `goestools` to capture these images, it was very straightforward to create a Python script that would accept a few parameters to produce a .gif from them. By inputting the above parameters plus a date range it would spit out a .gif by running a command from ImageMagick like this:

```bash
convert -delay 20 -loop 0 -resize 50% \
  ./goes18/m1/fd/fc/2023-07-05/*.jpg timelapse.gif
```

## Final Product

![The final product, a gif from about 24 hours of images](../../../../../images/GOES18_M1_FC_20230706.gif)

![Infrared images enhanced to detect major weather patterns](../../../../../images/timelapse-ch13_enhanced-2023-07-11.gif)

![Full-color image](../../../../../images/timelapse-fc-2023-07-11.gif)

#### References

[https://www.goes-r.gov/downloads/resources/documents/GOES-RSeriesDataBook.pdf]()

[https://space.oscar.wmo.int/satellites/view/goes_18]()

[https://www.rtl-sdr.com/rtl-sdr-com-goes-16-17-and-gk-2a-weather-satellite-reception-comprehensive-tutorial/]()

[https://usradioguy.com/goes-satellite-imagery-reception/]()
