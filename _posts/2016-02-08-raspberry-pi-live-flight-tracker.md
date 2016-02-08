---
layout: post
title:  "Raspberry Pi Live Flight Tracker"
---
NOTES:

+ abbreviate FR24

## Objectives

## Hardware

## Raspberry Pi Configuration

+ Static IP: Recommended so you can access the built-in dump1090 web interface; use network tools, such as netcat (nc) to view CSV output for debugging; or, use dynamic dns.

## Requirements

+ One USB 2.0 port.  (Keep in mind the USB dongle is quite large and can interfere with access to other ports.)
+ Raspbian either installed via NOOBS or from an image is fine.
+ Latitude, Longitude and altitude of antenna position - if you plan to use MLAT.  (If your smartphones' built-in GPS only shows latitude and longitude, you can use a third-party app to get the altitude.  Another option is to use FreeMapTools [Elevation Finder](https://www.freemaptools.com/elevation-finder.htm) or similar to obtain your latitude, longitude, and elevation.  If you use the website, remember to then add the height of your antenna from the ground to get the altitude.)

---

## FlightRadar24.com Feeder Installation

We will set up the FR24 feeder first.  This package includes a fork of [dump1090](https://github.com/MalcolmRobb/dump1090) with MLAT support, which we'll use to provide data to both FR24 and PiAware feeders.

### tl;dr

jhis one-liner will download, install, and setup the FlightRadar24.com feeder and will then initiate the user configuration.  What happens automatically is effectively the same as the steps followed during a manual install so you're free to use either method.

	$ sudo bash -c "$(wget -O - http://repo.feed.flightradar24.com/install_fr24_rpi.sh)"

A concern I have with the script is that, on line 29, it changes the permissions on the user configuration file (fr24feed.ini) so that it is readable and writable by all users.  (During the manual install the permissions restrict writing to only by the user.)

However, given that a typical Raspbian install is only configured with one login user and the fr24feed.ini contains little sensitive data, this issue is trival.  If you feel inclined, you can correct the permissions after the install is complete, by using the following:

	$ sudo chmod go-w /etc/fr24feed.ini

### Manual Install

These instructions are based on the instruction found within the FR24 forum post [New Flightradar24 feeding software for Raspberry Pie](http://forum.flightradar24.com/threads/8908-New-Flightradar24-feeding-software-for-Raspberry-Pie?p=66479#post66479)

Based on already having an account with FR.

I've included screenshots for each instruction demonstrating the expected output.

1.	Import FR24 signing key

		$ gpg --keyserver pgp.mit.edu --recv-keys 40C430F5

	[screenshot]({{ site.url }}/assets/images/2016/02/08/raspberry-pi-live-flight-tracker/fr24feed/01-Import_Key.png)

1.	Add new key to list of trusted keys

		$ gpg --armor --export 40C430F5 | sudo apt-key add -

	[screenshot]({{ site.url }}/assets/images/2016/02/08/raspberry-pi-live-flight-tracker/fr24feed/02-Trust_Key.png)

1.	Add repository to sources

		$ sudo sh -c "echo 'deb http://repo.feed.flightradar24.com flightradar24 raspberrypi-stable' >> /etc/apt/sources.list"

	[screenshot]({{ site.url }}/assets/images/2016/02/08/raspberry-pi-live-flight-tracker/fr24feed/03-Include_Source.png)

1.	Update the cache

		$ sudo apt-get update

	[screenshot]({{ site.url }}/assets/images/2016/02/08/raspberry-pi-live-flight-tracker/fr24feed/04-Update_Sources.png)

1.	Install FR feeder

		$ sudo apt-get install fr24feed

	[screenshot]({{ site.url }}/assets/images/2016/02/08/raspberry-pi-live-flight-tracker/fr24feed/05-Install_fr24feed.png)

1.	Run the sign-up wizard (even if you already have a sharing key)

		$ fr24feed --signup

	The actual signup process is quite verbose so I've attached an example walk-through in this [screenshot]({{ site.url }}/assets/images/2016/02/08/raspberry-pi-live-flight-tracker/fr24feed/06-Configure_fr24feed.png).

	Things to mention about the configuration process:

	+	*Step 1.2* ???
	+	*Step 4.3* type the values `--net --net-bi-port 30104`.  The first argument `--net` will instruct dump1090 to enable the built-in web interface and `--net-bi-port 30104` is required by the PiAware feeder.  You may omit the first argument if you don't plan on using the interface.
	+	*Step 6A and 6B* Unless you are debugging an issue with dump1090, I would suggest disabling logging.  dump1090 writes a lot of information to the logs and SD cards have a finite number of writes before they are damaged.

1.	Restart the FR24 feeder

		$ sudo service fr24feed restart

	[screenshot]({{ site.url }}/assets/images/2016/02/08/raspberry-pi-live-flight-tracker/fr24feed/07-Restart_fr24feed_Service.png)

1.	Test the FR24 feeder status

		$ sudo service fr24feed status

	[screenshot]({{ site.url }}/assets/images/2016/02/08/raspberry-pi-live-flight-tracker/fr24feed/08-fr24feed_Service_Status.png)

## Housekeeping

