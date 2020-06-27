---
layout: post
title: "ADS-B Multiple Feeds"
comments: true
---
<style>
	strong { font-size: medium; }
</style>
As a continuation of the [Raspberry Pi Real-Time Flight Tracker Updated](http://archdwm.local:4000/2018/04/13/raspberry-pi-real-time-flight-tracker-updated.html) post, in which I demonstrated how to install and configure your Raspberry Pi to turn it into a data feeder for Flightradar24 and FlightAware, this post will:

* review the hardware and other requirements;
* introduce five other feeder sites - ADS-B Exchange, ADSBHub, AirNav RadarBox, Plane Finder, and OpenSky Network;
* recommend the installation order for all the feeders - including Flightradar24 and FlightAware;
* revisit installing FlightAware's feeder;
* discuss any issues with installing the other feeders; and,
* feature the online virtual radar views.

## Introduction
When an aircraft is in flight, it transmits data at regular intervals to ground stations either by an [ADS-B](https://en.wikipedia.org/wiki/Automatic_dependent_surveillance_%E2%80%93_broadcast "ADS-B") (Automatic Dependant Surveillance Broadcast) or a [MODE-S ](https://en.wikipedia.org/wiki/Aviation_transponder_interrogation_modes#Mode_S "MODE-S")transponder at a frequency of 1090 MHz.  If the aircraft is equipped with an ADS-B transponder, it reports its GPS position and air speed, in addition to identifying information and altitude. MODE-S transponders only identify the aircraft and its altitude. Some of the feeders then derive the position of these aircraft using MLAT (Multilateration) software.

With a DVB-T USB dongle attached and the [librtlsdr0](https://osmocom.org/projects/rtl-sdr/wiki/Rtl-sdr "librtlsdr0") and [dump1090](https://github.com/flightaware/dump1090 "dump1090") packages installed, your Pi will be able to receive and decode the ADS-B and MODE-S transmissions. This is achieved by librtlsdr0, a software defined radio (SDR) receiver, tuning the dongle to the frequency and dump1090 decoding the data. The feeders then take the output and forward it to their respective servers. This crowd-sourced data helps augment the data the commercial companies receive from official channels, such as the FAA, and other crowd-sourced initiatives.

### Requirements

* DVB-T USB dongle based on the Realtek RTL2832U with the Rafael Micro R820T or R820T2 tuner.
* One USB 2.0 port. (Keep in mind the USB dongle is quite large and can interfere with access to other ports.)
* Raspbian (full or lite) either installed via NOOBS or from an image is fine.
* Location for the antenna with no obstructions &mdash; outside or in a window is best.
* [optional] Account(s) with the respective service, if you plan to feed data.  (Setting up the accounts and requesting feed authorization is beyond the scope of this post.)
* [optional] Latitude, longitude, and elevation of antenna position, if you plan to use MLAT. (Use your smartphone's built-in GPS or a third-party app to get these coordinates.)
* [optional] SSH access, if you plan to connect to a headless system.
* [optional] Static IP, if you want a fixed address when accessing the dump1090 webserver or connecting via SSH.

## Overview

In addition to the physical requirements you need to install a dump1090 and then one or more of the feeders.  (librtlsdr0 is installed as a dependency of dump1090.)

### dump1090

The original dump1090 was written by [antirez](https://github.com/antirez "antirez") and, when I first installed dump1090, the recommended version was the fork by [MalcolmRobb](https://github.com/MalcolmRobb "MalcolmRobb").  Later installation instructions would recommend the [mutability](https://github.com/mutability "mutability") fork.  Now [flightaware](https://github.com/flightaware "flightaware") has their own fork, from the mutability branch, and [Flightradar24](https://github.com/Flightradar24 "Flightradar24") forks from the MalcolmRobb branch.

Since Flightradar24 and FlightAware have their own versions of dump1090 &mdash; in addition to the 975 other forks  &mdash; and the other services simply require dump1090 be installed, which should you choose?  I recommend installing FlightAware's because it's based off of mutability, which is recommended by two of the four other services.  (The other services aren't picky about which version is used and Flightradar24's feeder hasn't complained about using it either.)

### Feeders

##### ADS-B Exchange (2015? - United States)
ADS-B Exchange (AE) is, "the worlds largest co-op of ADS-B/Mode S/MLAT feeders, and the world's largest public source of unfiltered flight data."<sup>[1](#footnote1)</sup>  Having unfiltered data means that military and certain private aircraft are still visible on their virtual radar.  Convenient if you want to figure out the details of the mystery aircraft that is flying around your area and doesn't appear on the commercial services' virtual radars.

<br />
**Website: [https://www.adsbexchange.com/](https://www.adsbexchange.com/)**  
<br />

##### ADSBHub (2017 - Germany)
ADSBHub (AH) is a strictly crowd-sourced service and anyone who feeds data to it is permitted to access the aggregate data.

**Website: [https://www.adsbhub.org/](https://www.adsbhub.org/)**  
<br />

##### AirNav RadarBox (1996 - United States)
AirNav RadarBox (RB) was one of the earliest commercial flight trackers.  Until recently you had to purchase their "RadarBox" hardware to decode the ADS-B and MODE-S signals, which could then be viewed on their virtual radar software or submitted to their service for discounted access to the online view.  Now they permit feeds from ARM Linux distros and provide free membership to contributors.

**Website: [https://www.radarbox.com/](https://www.radarbox.com/)**  
<br />

##### Flightradar24 (2006 - Sweden)
Flightradar24 (FR) is a commercial services and, possibly, the most well known as screenshots of their virtual radar are often used by the media when reporting on incidents.  They offer free Business tier membership for feeding data.  Their online virtual radar is the best of the bunch and the 3D view is an impressive feature.

**Website: [https://www.flightradar24.com/](https://www.flightradar24.com/)**
<br />

##### FlightAware (2005 - United States)
FlightAware (FA), another commercial service, is more focused on flight tracking and notification than providing a virtual radar.  The free Enterprise membership, in return for supplying data, allows you to create unlimited flight alerts and view eight months of historical flight data, in addition to a slightly more comprehensive virtual radar.

**Website: [https://flightaware.com/](https://flightaware.com/)**  
<br />

##### OpenSky Network (2015 - Switzerland)
The OpenSky Network (OS) is another service that crowd-sources data for displaying in real-time and archiving for researchers.  It started in 2012 as a research project between armasuisse (Switzerland), University of Kaiserslautern (Germany), and University of Oxford (UK).

**Website: [https://opensky-network.org/](https://opensky-network.org/)**  
<br />

##### Plane Finder (2009 - United Kingdom)
Plane Finder (PF) seems a lesser quality clone of Flightradar24, without a 3D view.  They provide the "Gifted Sharer Subscription" for active contributors.

**Website: [https://planefinder.net/](https://planefinder.net/)**  
<br />

## Installation

The order I tend to follow these days is:

1. FlightAware: [https://flightaware.com/adsb/piaware/install](https://flightaware.com/adsb/piaware/install)
2. Flightradar24: [https://www.flightradar24.com/share-your-data](https://www.flightradar24.com/share-your-data)
3. Plane Finder: [https://forum.planefinder.net/threads/raspberry-pi-b-zero-rpi2-rpi3-rpi4-installation-instructions-for-raspbian-dump1090-data-feeder.241/](https://forum.planefinder.net/threads/raspberry-pi-b-zero-rpi2-rpi3-rpi4-installation-instructions-for-raspbian-dump1090-data-feeder.241/)
4. AirNav RadarBox: [https://www.radarbox.com/sharing-data](https://www.radarbox.com/sharing-data)
5. OpenSky Network: [https://opensky-network.org/community/projects/30-dump1090-feeder](https://opensky-network.org/community/projects/30-dump1090-feeder)
6. ADS-B Exchange: [https://www.adsbexchange.com/how-to-feed/](https://www.adsbexchange.com/how-to-feed/)
7. ADSBHub: [https://www.adsbhub.org/howtofeed.php](https://www.adsbhub.org/howtofeed.php)

I install FlightAware first so that I have the mutability fork of dump1090 installed.  Feeders two through six can be installed in any order &mdash; remembering to configure them to use the already installed dump1090 instead of installing their own.

ADSBHub operates differently.  Rather than installing any dedicated software, their servers connect to the SBS output of dump1090 on port 30003 (TCP).  This requires you to port forward 30003 to your Pi.

## Virtual Radars

1. FlightAware: [https://flightaware.com/live/](https://flightaware.com/live/)
2. Flightradar24: [https://www.flightradar24.com/48.43,-123.37/](https://www.flightradar24.com/48.43,-123.37/)
3. Plane Finder: [https://planefinder.net/](https://planefinder.net/)
4. AirNav RadarBox: [https://www.radarbox.com/@48.15347,-123.49554,z9](https://www.radarbox.com/@48.15347,-123.49554,z9)
5. OpenSky Network: [https://opensky-network.org/network/explorer](https://opensky-network.org/network/explorer)
6. ADS-B Exchange: [https://tar1090.adsbexchange.com/](https://tar1090.adsbexchange.com/)
7. ADSBHub: [https://www.adsbhub.org/coverage.php](https://www.adsbhub.org/coverage.php)

Many of the feeders include their own web server or use NGINX for local data viewing.  Aircraft your receiver gets messages from will be plotted on a map with varying levels of detail.  (Pi-hole has a tendency to intercept these requests and display a page with the following message, "Pi-hole: Your black hole for Internet advertisements
Did you mean to go to the admin panel?"  I haven't taken the time to fix this issue and will likely install Pi-hole on another device as my solution.)

You can also install [Virtual Radar Server](http://www.virtualradarserver.co.uk/Download.aspx) on Windows / Mac / Linux.  It will serve up a virtual radar web page from your own computer.  This is another option for anyone with Pi-hole installed on the same device as the feeders.

## Further Reading
Craig ([http://www.makikiweb.com/](http://www.makikiweb.com/)) emailed me the article, "The Datawake ADS-B PiAware Receiver" [(https://microship.com/datawake-ads-b-piaware-receiver/)](https://microship.com/datawake-ads-b-piaware-receiver/).  It dives deeper into the hardware side of ADS-B feeders.

---
1. [https://www.adsbexchange.com/](https://www.adsbexchange.com/ "ADS-B Exchange")<a name="footnote1"></a>
