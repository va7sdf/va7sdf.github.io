---
layout: post
title: "Raspberry Pi Real-Time Flight Tracker"
---
<!--- Had to use the HTML anchor for FlightRadar24.com per https://github.com/MalcolmRobb/dump1090/pull/71 -->
One of the many cool things you can do with your Raspberry Pi is to add a specific $20-$30 [DVB-T](https://en.wikipedia.org/wiki/DVB-T) USB dongle and install some software to turn your Pi into a real-time flight virtual radar (for radarspotting) or data feeder to <a href="http://flightradar24.com/" rel="noreferrer">FlightRadar24.com</a> \(FR24\) and [FlightAware](https://flightaware.com/) \(FA\). In return both sites offer enhanced memberships while you maintain your feed to them. A real bonus for aviation enthusiasts!

---

## Overview

<figure class="ink-image">
	<a href="{{ site.baseurl }}/assets/images/2016/02/10/raspberry-pi-real--time-flight-tracker/Diagram.png">
		<img src="{{ site.baseurl }}/assets/images/2016/02/10/raspberry-pi-real--time-flight-tracker/Diagram.png">
	</a>
</figure>

When an aircraft is in flight, it transmits data at regular intervals to ground stations either by an [ADS-B](https://en.wikipedia.org/wiki/Automatic_dependent_surveillance_–_broadcast) or a [MODE-S](https://en.wikipedia.org/wiki/Aviation_transponder_interrogation_modes#Mode_S) transponder at a frequency of 1090 MHz. If the aircraft is equiped with an ADS-B transponder, it reports its GPS position and velocity, in addition to idetifying information and altitude. MODE-S transponders only identify the aircraft and its altitude. Both FR24 and FA then derrive the position of these aircraft using [MLAT \(Multilateration\)](https://en.wikipedia.org/wiki/Multilateration).

With a DVB-T USB dongle attached and [dump1090](https://github.com/MalcolmRobb/dump1090) software installed, your Pi will be able to receive and decode the ADS-B and MODE-S transmissions. This is achieved by dump1090 tuning the dongle to the frequency and then outputting the decoded data. FR24 and FA software then takes this output and forwards it to their respective servers. This crowd-sourced data helps augment the data these companies receive from official channels, such as the [FAA](https://en.wikipedia.org/wiki/Federal_Aviation_Administration).

You can then view a virtual radar representation of your feed via the built-in dump1090 web server. FR24 and FA offer more comprehensive virtual radar maps on their feature rich sites.

Since a typical DVB-T dongle costs $20-$30, a Pi feeder is a more affordable option for hobbyists when compared to the expensive higher-end ADS-B virtual radars such as the [Radarcape](http://modesbeast.com/radarcape.html) or [AirNav RadarBox](https://www.airnavsystems.com/RadarBox/index.html) or dedicated decoders like the [Mode-S Beast](http://modesbeast.com/scope.html) or [Kinetic SBS](http://www.kinetic.co.uk/index.php) models.

---

## Requirements

+ DVB-T USB dongle based on the Realtek RTL2832U with the Rafael Micro R820T or R820T2 tuner. \(The [NooElec NESDR Mini 2](http://www.amazon.ca/gp/product/B00PAGS0HO/) is recommended. I currently use the older [NooElec TV28Tv2](http://www.amazon.ca/gp/product/B00CM3LNMM/).\)
+ One USB 2.0 port. (Keep in mind the USB dongle is quite large and can interfere with access to other ports.)
+ Raspbian Wheezy or Jessie (full or lite) either installed via NOOBS or from an image is fine.
+ Location for the antenna with no obstructions &mdash; outside or in a window is best.
+ \[optional\] Account(s) with <a href="http://flightradar24.com/" rel="noreferrer">FlightRadar24.com</a> and/or [FlightAware](https://flightaware.com/), if you plan to feed data. (Setting up the accounts and requesting feeder keys is beyond the scope of this post.)
+ \[optional\] Latitude, longitude and altitude of antenna position, if you plan to use MLAT. (If your smartphone's built-in GPS only shows latitude and longitude, you can use a third-party app to get the altitude. Another option is to use FreeMapTools [Elevation Finder](https://www.freemaptools.com/elevation-finder.htm) or similar to obtain your approximate latitude, longitude, and elevation. If you use the website, remember to then add the height of your antenna from the ground elevation to get the altitude.)
+ \[optional\] SSH access, if you plan to operate your Pi headless or remotely.
+ \[optional\] Static IP so you can access the built-in dump1090 web server; use network tools, such as netcat (nc) to view CSV output for debugging; or, use dynamic DNS behind your network.

---

## Installation

We will set up the FR24 feeder first. This package includes a fork of dump1090, with MLAT support, which we'll use to provide data to both FR24 and FA feeders.

With each command line, I've included screenshots of sample output for your reference. Keep in mind that your output might differ depending on your system's configuration and current state.

### FlightRadar24.com Feeder Installation

These manual instructions are based on the those found within the FR24 forum post [New Flightradar24 feeding software for Raspberry Pie](http://forum.flightradar24.com/threads/8908-New-Flightradar24-feeding-software-for-Raspberry-Pie?p=66479#post66479) [*sic*].

1.	Import FR24 signing key

		gpg --keyserver pgp.mit.edu --recv-keys 40C430F5

	[screenshot]({{ site.baseurl }}/assets/images/2016/02/10/raspberry-pi-real--time-flight-tracker/fr24feed/01-Import_Key.png)

1.	Add new key to list of trusted keys

		gpg --armor --export 40C430F5 | sudo apt-key add -

	[screenshot]({{ site.baseurl }}/assets/images/2016/02/10/raspberry-pi-real--time-flight-tracker/fr24feed/02-Trust_Key.png)

1.	Add repository to sources

		sudo sh -c "echo 'deb http://repo.feed.flightradar24.com flightradar24 raspberrypi-stable' >> /etc/apt/sources.list"

	[screenshot]({{ site.baseurl }}/assets/images/2016/02/10/raspberry-pi-real--time-flight-tracker/fr24feed/03-Include_Source.png)

1.	Update the cache

		sudo apt-get update

	[screenshot]({{ site.baseurl }}/assets/images/2016/02/10/raspberry-pi-real--time-flight-tracker/fr24feed/04-Update_Sources.png) (new sources highlighted for your reference)

1.	Install FR feeder

		sudo apt-get install fr24feed

	[screenshot]({{ site.baseurl }}/assets/images/2016/02/10/raspberry-pi-real--time-flight-tracker/fr24feed/05-Install_fr24feed.png)

1.	Run the sign-up wizard (even if you already have a sharing key)

		fr24feed --signup

	The actual signup process is quite verbose so I've attached an example walk-through in this [screenshot]({{ site.baseurl }}/assets/images/2016/02/10/raspberry-pi-real--time-flight-tracker/fr24feed/06-Configure_fr24feed.png).

	Things to mention about the configuration process:

	+	*Step 1.2* 
	+	*Step 4.3* type the values `--net --net-bi-port 30104`. The first argument `--net` will instruct dump1090 to enable the built-in web server and `--net-bi-port 30104` is required by the PiAware feeder.
	+	*Step 6A and 6B* Unless you are debugging an issue with dump1090, I would suggest disabling logging. dump1090 writes a lot of information to the logs and SD cards have a finite number of writes before they are damaged.

1.	Restart the FR24 feeder

		sudo service fr24feed restart

	[screenshot]({{ site.baseurl }}/assets/images/2016/02/10/raspberry-pi-real--time-flight-tracker/fr24feed/07-Restart_fr24feed_Service.png)

1.	Test the FR24 feeder status

		sudo service fr24feed status

	[screenshot]({{ site.baseurl }}/assets/images/2016/02/10/raspberry-pi-real--time-flight-tracker/fr24feed/08-fr24feed_Service_Status.png)

### FlightAware Feeder Installation

With dump1090 installed and configured as part of the FR24 feeder installation, we can proceed to install the PiAware feeder.

These instructions follow those in sections two through four on [PiAware - dump1090 ADS-B integration with FlightAware ✈ FlightAware](https://flightaware.com/adsb/piaware/install#2)

1.	Download the PiAware package

		wget http://flightaware.com/adsb/piaware/files/piaware_2.1-5_armhf.deb

	[screenshot]({{ site.baseurl }}/assets/images/2016/02/10/raspberry-pi-real--time-flight-tracker/piaware/01-Download_piaware_Package.png)

1.	Install the PiAware package

		sudo dpkg -i piaware_2.1-5_armhf.deb

	[screenshot]({{ site.baseurl }}/assets/images/2016/02/10/raspberry-pi-real--time-flight-tracker/piaware/02-Install_piaware_Package.png)

1.	Install the required dependencies

		sudo apt-get install -fy

	[screenshot]({{ site.baseurl }}/assets/images/2016/02/10/raspberry-pi-real--time-flight-tracker/piaware/03-Install_Dependencies.png)

1.	Enable automatic and manual PiAware software updates

		sudo piaware-config -autoUpdate 1 -manualUpdate 1

	[screenshot]({{ site.baseurl }}/assets/images/2016/02/10/raspberry-pi-real--time-flight-tracker/piaware/04-Configure_piaware_1.png)

	Note: Manual updates are performed via your FA profile page [LINK]

1.	Configure PiAware with your FA account

		sudo piaware-config -user <username> -password

	[screenshot]({{ site.baseurl }}/assets/images/2016/02/10/raspberry-pi-real--time-flight-tracker/piaware/05-Configure_piaware_2.png)

	Note: Substitute <username> with your own FA username. After issuing the command, you will be asked for your FA password.

1.	Restart the PiAware feeder

		sudo /etc/init.d/piaware restart

	[screenshot]({{ site.baseurl }}/assets/images/2016/02/10/raspberry-pi-real--time-flight-tracker/piaware/06-Restart_piaware_Service.png)

1.	Test the PiAware feeder status

		sudo service piaware status

	[screenshot]({{ site.baseurl }}/assets/images/2016/02/10/raspberry-pi-real--time-flight-tracker/piaware/07-piaware_Service_Status.png)

---

## dump1090 Virtual Radar

With the `--net` argument passed to dump1090, it will also act as a web server on port 8080. To view the virtual radar web page, visit [http://192.168.1.31:8080/](http://192.168.1.31:8080/) using your browser. (Remember to substitute 192.168.1.31 with the IP of your Pi.)

If dump1090 is running, you will see a Google Map featuring Europe. Once you pan and zoom the map to your location, it should look similar to the screenshot below and, if there's any air traffic in the area your receiver can pick up, those flights will be displayed.

Keep in mind that only flights transmitting ADS-B data are represented on the map because MODE-S doen't send position information. All flights detected will be listed in the right hand side.

<figure class="ink-image">
	<a href="{{ site.baseurl }}/assets/images/2016/02/10/raspberry-pi-real--time-flight-tracker/Boeing_737_Max-dump1090.png">
		<img src="{{ site.baseurl }}/assets/images/2016/02/10/raspberry-pi-real--time-flight-tracker/Boeing_737_Max-dump1090.png">
	</a>
	<figcaption class="over-bottom">
		Boeing 737 MAX First Flight (January 29, 2016): dump1090 Web Page
	</figcaption>
</figure>

---

## FlightRadar24.com and FlightAware.com Profiles

Each website provides a profile page with some statistics about your feed. FA is far more comprehensive and you can even send commands to your Pi from your profile page. FA is also configured by default to alert you by email if your feeder is down.

The directions to the profile page for each site are as follows:

**FlightRadar24.com**

+	Visit <a href="http://flightradar24.com/premium/" rel="noreferrer">http://flightradar24.com/premium/</a>
+	Click on the *Premium* button in the upper right
+	Sign in with your credentials
+	Click on the *Your Feeds* in the collection of buttons to the right of *Your Account*
+	Click *More info*

**FlightAware.com**

+	Visit [https://flightaware.com/](https://flightaware.com/)
+	Click the *Login* link near the upper left
+	Sign in with your credentials
+	Click the *My ADS-B* link near the upper left

---

## Conclusion




---

## Screenshot Gallery

<figure class="ink-image">
	<a href="{{ site.baseurl }}/assets/images/2016/02/10/raspberry-pi-real--time-flight-tracker/Boeing_737_Max-flightradar24.com+live_stream.png">
		<img src="{{ site.baseurl }}/assets/images/2016/02/10/raspberry-pi-real--time-flight-tracker/Boeing_737_Max-flightradar24.com+live_stream.png">
	</a>
	<figcaption>
		Boeing 737 MAX First Flight (January 29, 2016): FlightRadar24.com Track and Live Stream Landing
	</figcaption>
</figure>

<div class="column-group horizontal-gutters">
	<div class="xlarge-50 large-50 medium-50 small-100 tiny-100">
		<figure class="ink-image">
			<a href="{{ site.baseurl }}/assets/images/2016/02/10/raspberry-pi-real--time-flight-tracker/RESEARCH9.png">
				<img src="{{ site.baseurl }}/assets/images/2016/02/10/raspberry-pi-real--time-flight-tracker/RESEARCH9.png">
			</a>
			<figcaption>
				FlightRadar24.com Screenshot
			</figcaption>
		</figure>
	</div>
	<div class="xlarge-50 large-50 medium-50 small-100 tiny-100">
		<figure class="ink-image">
			<a href="{{ site.baseurl }}/assets/images/2016/02/10/raspberry-pi-real--time-flight-tracker/flightaware.com.png">
				<img src="{{ site.baseurl }}/assets/images/2016/02/10/raspberry-pi-real--time-flight-tracker/flightaware.com.png">
			</a>
			<figcaption>
				FlightAware.com Screenshot
			</figcaption>
		</figure>
	</div>
</div>

