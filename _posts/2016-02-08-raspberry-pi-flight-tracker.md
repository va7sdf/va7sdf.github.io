---
layout: post
title: "Raspberry Pi Flight Tracker"
---
One of the many cool things you can do with your Raspberry Pi is to add a special $30 [DVB-T](https://en.wikipedia.org/wiki/DVB-T) USB dongle and install some software to turn your Pi into a flight data feeder for <a href="http://flightradar24.com/" rel="noreferrer">FlightRadar24</a> and [FlightAware](https://flightaware.com).
<!--- Had to use the HTML anchor for FlightRadar24.com per https://github.com/MalcolmRobb/dump1090/pull/71 -->

A Pi feeder is an more affordable hobbyist option when compared to the expensive higher-end [ADS-B](https://en.wikipedia.org/wiki/Automatic_dependent_surveillance_–_broadcast) receivers such as the [Radarcape](http://modesbeast.com/radarcape.html), [Mode-S Beast](http://modesbeast.com/scope.html), or [Kinetic SBS Models](http://www.kinetic.co.uk/index.php).

## DVB-T USB Dongle

As long as you purchase a DVB-T dongle based on the Realtek RTL2832U with the Rafael Micro R820T or R820T2 tuner, the Mode S decoder software, dump1090, can interface with it.

The [NooElec NESDR Mini 2](http://www.amazon.ca/gp/product/B00PAGS0HO/) is recommended. I currently use the older [NooElec TV28Tv2](http://www.amazon.ca/gp/product/B00CM3LNMM/)

## Raspberry Pi Configuration

+ Static IP: Recommended so you can access the built-in dump1090 web interface; use network tools, such as netcat (nc) to view CSV output for debugging; or, use dynamic dns.

## Requirements

+ One USB 2.0 port. (Keep in mind the USB dongle is quite large and can interfere with access to other ports.)
+ Raspbian (Wheezy or Jessie) either installed via NOOBS or from an image is fine.
+ Latitude, Longitude and altitude of antenna position - if you plan to use MLAT. (If your smartphones' built-in GPS only shows latitude and longitude, you can use a third-party app to get the altitude. Another option is to use FreeMapTools [Elevation Finder](https://www.freemaptools.com/elevation-finder.htm) or similar to obtain your latitude, longitude, and elevation. If you use the website, remember to then add the height of your antenna from the ground to get the altitude.)

---

## Installation

We will set up the FR24 feeder first. This package includes a fork of [dump1090](https://github.com/MalcolmRobb/dump1090) with MLAT support, which we'll use to provide data to both FR24 and PiAware feeders.

With each command line, I've included screenshots of sample output for your reference. Keep in mind that your output might differ depending on your system's configuration and current state.

### FlightRadar Feeder Installation


#### tl;dr

jhis one-liner will download, install, and setup the FlightRadar24.com feeder and will then initiate the user configuration. What happens automatically is effectively the same as the steps followed during a manual install so you're free to use either method.

	sudo bash -c "$(wget -O - http://repo.feed.flightradar24.com/install_fr24_rpi.sh)"

A concern I have with the script is that, on line 29, it changes the permissions on the user configuration file (fr24feed.ini) so that it is readable and writable by all users. (During the manual install the permissions restrict writing to only by the user.)

However, given that a typical Raspbian install is only configured with one login user and the fr24feed.ini contains little sensitive data, this issue is trival. If you feel inclined, you can correct the permissions after the install is complete, by using the following:

	sudo chmod go-w /etc/fr24feed.ini

#### Manual Install

These instructions are based on the instruction found within the FR24 forum post [New Flightradar24 feeding software for Raspberry Pie](http://forum.flightradar24.com/threads/8908-New-Flightradar24-feeding-software-for-Raspberry-Pie?p=66479#post66479) [*sic*].

Based on already having an account with FR.

I've included screenshots for each instruction demonstrating the expected output.

1.	Import FR24 signing key

		gpg --keyserver pgp.mit.edu --recv-keys 40C430F5

	[screenshot]({{ site.url }}/assets/images/2016/02/08/raspberry-pi-live-flight-tracker/fr24feed/01-Import_Key.png)

1.	Add new key to list of trusted keys

		gpg --armor --export 40C430F5 | sudo apt-key add -

	[screenshot]({{ site.url }}/assets/images/2016/02/08/raspberry-pi-live-flight-tracker/fr24feed/02-Trust_Key.png)

1.	Add repository to sources

		sudo sh -c "echo 'deb http://repo.feed.flightradar24.com flightradar24 raspberrypi-stable' >> /etc/apt/sources.list"

	[screenshot]({{ site.url }}/assets/images/2016/02/08/raspberry-pi-live-flight-tracker/fr24feed/03-Include_Source.png)

1.	Update the cache

		sudo apt-get update

	[screenshot]({{ site.url }}/assets/images/2016/02/08/raspberry-pi-live-flight-tracker/fr24feed/04-Update_Sources.png) (new sources highlighted for your reference)

1.	Install FR feeder

		sudo apt-get install fr24feed

	[screenshot]({{ site.url }}/assets/images/2016/02/08/raspberry-pi-live-flight-tracker/fr24feed/05-Install_fr24feed.png)

1.	Run the sign-up wizard (even if you already have a sharing key)

		fr24feed --signup

	The actual signup process is quite verbose so I've attached an example walk-through in this [screenshot]({{ site.url }}/assets/images/2016/02/08/raspberry-pi-live-flight-tracker/fr24feed/06-Configure_fr24feed.png).

	Things to mention about the configuration process:

	+	*Step 1.2* ???
	+	*Step 4.3* type the values `--net --net-bi-port 30104`. The first argument `--net` will instruct dump1090 to enable the built-in web interface and `--net-bi-port 30104` is required by the PiAware feeder. You may omit the first argument if you don't plan on using the interface.
	+	*Step 6A and 6B* Unless you are debugging an issue with dump1090, I would suggest disabling logging. dump1090 writes a lot of information to the logs and SD cards have a finite number of writes before they are damaged.

1.	Restart the FR24 feeder

		sudo service fr24feed restart

	[screenshot]({{ site.url }}/assets/images/2016/02/08/raspberry-pi-live-flight-tracker/fr24feed/07-Restart_fr24feed_Service.png)

1.	Test the FR24 feeder status

		sudo service fr24feed status

	[screenshot]({{ site.url }}/assets/images/2016/02/08/raspberry-pi-live-flight-tracker/fr24feed/08-fr24feed_Service_Status.png)

### FlightAware Feeder Installation

With dump1090 installed and configured as part of the FR24 feeder installation, we can proceed to install the PiAware feeder.

These instructions follow those in sections two through four on [PiAware - dump1090 ADS-B integration with FlightAware ✈ FlightAware](https://flightaware.com/adsb/piaware/install#2)

1.	Download the PiAware package

		wget http://flightaware.com/adsb/piaware/files/piaware_2.1-5_armhf.deb

	[screenshot]({{ site.url }}/assets/images/2016/02/08/raspberry-pi-live-flight-tracker/piaware/01-Download_piaware_Package.png)

1.	Install the PiAware package

		sudo dpkg -i piaware_2.1-5_armhf.deb

	[screenshot]({{ site.url }}/assets/images/2016/02/08/raspberry-pi-live-flight-tracker/piaware/02-Install_piaware_Package.png)

1.	Install the required dependencies

		sudo apt-get install -fy

	[screenshot]({{ site.url }}/assets/images/2016/02/08/raspberry-pi-live-flight-tracker/piaware/03-Install_Dependencies.png)

1.	Enable automatic and manual PiAware software updates

		sudo piaware-config -autoUpdate 1 -manualUpdate 1

	[screenshot]({{ site.url }}/assets/images/2016/02/08/raspberry-pi-live-flight-tracker/piaware/04-Configure_piaware_1.png)

	Note: Manual updates are performed via your FA profile page [LINK]

1.	Configure PiAware with your FA account

		sudo piaware-config -user <username> -password

	[screenshot]({{ site.url }}/assets/images/2016/02/08/raspberry-pi-live-flight-tracker/piaware/05-Configure_piaware_2.png)

	Note: Substitute <username> with your own FA username. After issuing the command, you will be asked for your FA password.

1.	Restart the PiAware feeder

		sudo /etc/init.d/piaware restart

	[screenshot]({{ site.url }}/assets/images/2016/02/08/raspberry-pi-live-flight-tracker/piaware/06-Restart_piaware_Service.png)

1.	Test the PiAware feeder status

		sudo service piaware status

	[screenshot]({{ site.url }}/assets/images/2016/02/08/raspberry-pi-live-flight-tracker/piaware/07-piaware_Service_Status.png)

## Testing

### Netcat

To see 
[Animated GIF]({{ site.url }}/assets/images/2016/02/08/raspberry-pi-live-flight-tracker/netcat_output.gif)

## Housekeeping


