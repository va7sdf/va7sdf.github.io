---
layout: post
title: "Raspberry Pi Real-Time Flight Tracker Updated"
comments: true
---
<!--- Had to use the HTML anchor for FlightRadar24.com per https://github.com/MalcolmRobb/dump1090/pull/71 -->
One of the many cool things you can do with your Raspberry Pi is to add a specific $20-$30 [DVB-T](https://en.wikipedia.org/wiki/DVB-T) USB dongle and install some software to turn your Pi into a real-time flight virtual radar \(for radarspotting\) or data feeder to <a href="https://www.flightradar24.com/" rel="noreferrer">FlightRadar24.com</a> \(FR24\), [FlightAware.com](https://flightaware.com/) \(FA\), and [PlaneFinder.net](https://planefinder.net/) \(PF\). In return, all sites offer enhanced memberships while you maintain your feed to them. A real bonus for aviation enthusiasts!

---

## Overview

<figure class="ink-image">
	<a href="{{ site.baseurl }}/assets/images/2018/04/13/raspberry-pi-real--time-flight-tracker-updated/Diagram.png">
		<img src="{{ site.baseurl }}/assets/images/2018/04/13/raspberry-pi-real--time-flight-tracker-updated/Diagram.png" alt="Overview Diagram">
	</a>
</figure>

When an aircraft is in flight, it transmits data at regular intervals to ground stations either by an [ADS-B](https://en.wikipedia.org/wiki/Automatic_dependent_surveillance_–_broadcast) or a [MODE-S](https://en.wikipedia.org/wiki/Aviation_transponder_interrogation_modes#Mode_S) transponder at a frequency of 1090 MHz. If the aircraft is equipped with an ADS-B transponder, it reports its GPS position and air speed, in addition to identifying information and altitude. MODE-S transponders only identify the aircraft and its altitude. FR24, FA, and PF then derive the position of these aircraft using [MLAT \(Multilateration\)](https://en.wikipedia.org/wiki/Multilateration), where possible.

With a DVB-T USB dongle attached and the [librtlsdr0](https://osmocom.org/projects/sdr/wiki/rtl-sdr) and [dump1090](https://github.com/mutability/dump1090/) packages installed, your Pi will be able to receive and decode the ADS-B and MODE-S transmissions. This is achieved by librtlsdr0, a software defined radio \(SDR\) receiver, tuning the dongle to the frequency and dump1090 decoding the data. FR24, FA, and PF feeders then take the output and forward it to their respective servers. This crowd-sourced data helps augment the data these companies receive from official channels, such as the [FAA](https://en.wikipedia.org/wiki/Federal_Aviation_Administration).

You can then install a webserver and configure it to deliver a local virtual radar representation of your dump1090 feed. FR24, FA, and PF also offer more comprehensive virtual radar maps on their feature rich sites.

---

## Requirements

+ DVB-T USB dongle based on the Realtek RTL2832U with the Rafael Micro R820T or R820T2 tuner. \(The [NooElec NESDR Mini 2](https://www.amazon.ca/gp/product/B00PAGS0HO/) is recommended. I currently use the older [NooElec TV28Tv2](https://www.amazon.ca/gp/product/B00CM3LNMM/).\)
+ One USB 2.0 port. \(Keep in mind the USB dongle is quite large and can interfere with access to other ports.\)
+ Raspbian Jessie or Stretch \(full or lite\) either installed via NOOBS or from an image is fine.
+ Location for the antenna with no obstructions &mdash; outside or in a window is best.
+ \[optional\] Account\(s\) with <a href="https://www.flightradar24.com/" rel="noreferrer">FlightRadar24.com</a>, [FlightAware.com](https://flightaware.com/), and [PlaneFinder.net](https://planefinder.net/), if you plan to feed data. \(Setting up the accounts and requesting feed authorization is beyond the scope of this post.\)
+ \[optional\] Latitude, longitude, and elevation of antenna position, if you plan to use MLAT. \(Use your smartphone's built-in GPS or a third-party app to get these coordinates.)
+ \[optional\] SSH access, if you plan to connect to a headless system.
+ \[optional\] Static IP, if you want a fixed address when accessing the dump1090 webserver or connecting via SSH.

---

## Installation

Much has changed since my [original post](http://gordon.celesta.me/2016/02/10/raspberry-pi-real-time-flight-tracker.html).  The FR feeder no longer bundles its own version of dump1090 so, we will start by installing dump1090-mutability, the fork recommended by FR, and then the feeders.

With some instructions, I've included screenshots of sample output for your reference. Keep in mind that your output might differ depending on your system's current state.

#### dump1090-mutability Installation

These manual instructions are based on the those found within the FR24 forum post, ["New Flightradar24 feeding software for Raspberry Pie"](http://forum.flightradar24.com/threads/8908-New-Flightradar24-feeding-software-for-Raspberry-Pie?p=66479#post66479) \[*sic*\].

1.  Install the dependency, librtlsdr0 (software defined radio receiver for Realtek RTL2832U)

		sudo apt-get install librtlsdr0

1.	Add udev rules

		sudo wget -O /etc/udev/rules.d/rtl-sdr.rules https://raw.githubusercontent.com/osmocom/rtl-sdr/master/rtl-sdr.rules

1.	Download the dump1090-mutability package

		wget https://github.com/mutability/dump1090/releases/download/v1.14/dump1090-mutability_1.14_armhf.deb

1.	Install the dump1090-mutability package

		sudo dpkg -i dump1090-mutability_1.14_armhf.deb

	When prompted "Start dump1090 automatically?", press __enter__ to accept the default, "Yes".

1.	Reboot your Pi

1.	Verify dump1090-mutability is running, by

	1.	using the service command, or

			service dump1090-mutability status

		[screenshot]

	1.	using nc (netcat) to view the comma delimited SBS-format output

			nc localhost 30003

		[screenshot]

#### [Optional] lighttpd Installation

dump1090 can be configured to work with a standalone webserver to display a virtual radar web page over your local network.  The authors of dump1090 recommend using [lighttpd](https://www.lighttpd.net/) rather than webserver the dump1090 built-in webserver.

1.	Install lighttpd

		sudo apt-get install lighttpd

1.	Enable dump1090 configuration

		sudo lighty-enable-mod dump1090

1.	Restart lighttpd

		sudo service lighttpd force-reload

See the section, **External webserver configuration**, on ["dump1090-mutability Debian/Raspbian packages"](https://github.com/mutability/dump1090/#external-webserver-configuration) for more information.

#### FlightRadar24.com Feeder Installation

These manual instructions are based on the those found within the FR24 forum post ["New Flightradar24 feeding software for Raspberry Pie"](http://forum.flightradar24.com/threads/8908-New-Flightradar24-feeding-software-for-Raspberry-Pie?p=66479#post66479) \[*sic*\].

1.	Install the dependency, dirmngr (GNU privacy guard - network certificate management service)

		sudo apt-get install dirmngr

1.	Import FR24 signing key

		gpg --keyserver pool.sks-keyservers.net --recv-keys 40C430F5

	[screenshot]({{ site.baseurl }}/assets/images/2018/04/13/raspberry-pi-real--time-flight-tracker-updated/fr24feed/01-Import_Key.png)

1.	Add new key to list of trusted keys

		gpg --armor --export 40C430F5 | sudo apt-key add -

	[screenshot]({{ site.baseurl }}/assets/images/2018/04/13/raspberry-pi-real--time-flight-tracker-updated/fr24feed/02-Trust_Key.png)

1.	Add FR24 repository to sources

		sudo sh -c "echo 'deb http://repo.feed.flightradar24.com flightradar24 raspberrypi-stable' >> /etc/apt/sources.list"

1.	Update the cache to include the new repository

		sudo apt-get update

	[screenshot]({{ site.baseurl }}/assets/images/2018/04/13/raspberry-pi-real--time-flight-tracker-updated/fr24feed/04-Update_Sources.png) \(new sources highlighted for your reference\)

1.	Install fr24feed feeder

		sudo apt-get install fr24feed

1.	Run the sign-up wizard \(even if you already have a sharing key\)

		fr24feed --signup

	The actual signup process is quite verbose, so I've attached an example walk-through in this [screenshot]({{ site.baseurl }}/assets/images/2018/04/13/raspberry-pi-real--time-flight-tracker-updated/fr24feed/06-Configure_fr24feed.png).

	Note: *Step 6A* and *6B* disable logging. dump1090, with debugging on, writes a lot of information to the logs and SD cards have a finite number of writes before they are damaged.

1.	Restart the feeder

		sudo service fr24feed restart

1.	Verify the feeder service is running by

	1.	using the service command, or

			service fr24feed status

		[screenshot]({{ site.baseurl }}/assets/images/2018/04/13/raspberry-pi-real--time-flight-tracker-updated/fr24feed/08-fr24feed_Service_Status.png)

	1.	using the built-in feeder status

			fr24feed-status

		[screenshot]

#### FlightAware Feeder Installation

These instructions follow those in section two of, ["PiAware - dump1090 ADS-B integration with FlightAware ✈ FlightAware"](https://flightaware.com/adsb/piaware/install#2).

1.	Download the PiAware feeder package

		wget http://flightaware.com/adsb/piaware/files/packages/pool/piaware/p/piaware-support/piaware-repository_3.5.3_all.deb

1.	Install the piaware package

		sudo dpkg -i piaware-repository_3.5.3_all.deb

1.	Update the cache to include the new repository

		sudo apt-get update

	[screenshot] \(new sources highlighted for your reference\)

1.	Install piaware feeder

		sudo apt-get install piaware

1.	Enable automatic and manual PiAware software updates

		sudo piaware-config allow-auto-updates yes; sudo piaware-config allow-manual-updates yes

	[screenshot]({{ site.baseurl }}/assets/images/2018/04/13/raspberry-pi-real--time-flight-tracker-updated/piaware/04-Configure_piaware_1.png)

	Note: Manual updates are performed via your FA profile page.

1.	Restart the feeder

		sudo service piaware restart

1.	Verify the feeder service is running

		sudo service piaware status

	[screenshot]({{ site.baseurl }}/assets/images/2018/04/13/raspberry-pi-real--time-flight-tracker-updated/piaware/07-piaware_Service_Status.png)

1.	Claim your feeder online at [https://flightaware.com/adsb/piaware/claim](https://flightaware.com/adsb/piaware/claim)

#### PlaneFinder Feeder Installation

These instructions follow those in sections **Client Installation** and **Pi/RTL** under the heading **Installation &amp; Configuration** in, ["Microsoft Word - Plane-Finder-Debian-Client.docx"](https://876e4dd3654715fa151c-71f8796d2abe5094889e30919e12901d.ssl.cf3.rackcdn.com/docs/Plane-Finder-Debian-Client.pdf)

1.	Download the pfclient feeder package

		wget http://client.planefinder.net/pfclient_3.7.40_armhf.deb

1.	Install the pfclient feeder package

		sudo dpkg -i pfclient_3.7.40_armhf.deb

	Note: You don't need to update the cache or install the package using apt-get since this package doesn't add a repository to the sources list.

1.	Configure your feeder at [http://192.168.1.31:30053/](http://192.168.1.31:30053/) \(Remember to substitute 192.168.1.31 with the IP of your Pi.\)

	1.	On the first page, type the:

		+	email address associated with your account, and
		+	latitude and longitude of your antenna

	1.	Click the *Create a new sharecode* button

	1.	On the second page, type:

		+	**127.0.0.1** in the *IP address* text box, and
		+	**30005** in the *Port number* text box

	1.	Click the *Complete configuration* button

#### OpenSky-Network Feeder Installation

See [OpenSky Feeder for Dump1090 (Raspberry Pi-based)](https://opensky-network.org/community/projects/30-dump1090-feeder) for detailed instructions.

1.	Download the opensky-feeder package

		wget https://opensky-network.org/files/firmware/opensky-feeder_latest_armhf.deb

	[screenshot]

1.	Install the opensky-feeder package

		sudo dpkg -i opensky-feeder_latest_armhf.deb

	[screenshot]

1.	The opensky-feeder install will ask for the following information on separate ncurses screens:

	+	latitude of the antenna
	+	longitude of the antenna
	+	altitude of the antenna
	+   your OpenSky-Network username
	+	serial number of your existing receiver, which you will leave blank for a new receiver
	+	port for the output from dump1090, type **30005**, and
	+	host of the system running dump1090, type **localhost**

---

## dump1090 Virtual Radar

Visit http://192.168.1.31:8080/ in your browser to view the dump1090 virtual radar. \(Remember to substitute 192.168.1.31 with the IP of your Pi.\) You should see a Google Map of Europe. Once you pan and zoom the map to your location, it should look similar to the screenshot below and, if there's any air traffic in the area your receiver can pick up, those flights will be displayed.

Keep in mind that only flights transmitting ADS-B data are represented on the map because MODE-S doesn't send position information. All flights detected will be listed in the right-hand side.

<figure class="ink-image">
	<a href="{{ site.baseurl }}/assets/images/2018/04/13/raspberry-pi-real--time-flight-tracker-updated/Boeing_737_Max-dump1090.png">
		<img src="{{ site.baseurl }}/assets/images/2018/04/13/raspberry-pi-real--time-flight-tracker-updated/Boeing_737_Max-dump1090.png" alt="Boeing 737 MAX First Flight (January 29, 2016): dump1090 Web Page">
	</a>
	<figcaption class="over-bottom">
		Boeing 737 MAX First Flight (January 29, 2016): dump1090 Web Page
	</figcaption>
</figure>

---

## FlightRadar24.com, FlightAware.com, and PlaneFinder.net Profiles

Each website provides a profile page with some statistics about your feed. FA is far more comprehensive, and you can even send commands to your Pi from your profile page. FA is also configured by default to alert you by email if your feeder is down.

The directions to the profile page for each site are as follows:

**FlightRadar24.com**

+	Visit <a href="https://www.flightradar24.com/premium/" rel="noreferrer">https://www.flightradar24.com/premium/</a>
+	Click on the *Premium* button in the upper right
+	Sign in with your credentials
+	Click on the *Your Feeds* in the collection of buttons to the right of the *Your Account* heading
+	Click *More info*

**FlightAware.com**

+	Visit [https://flightaware.com/](https://flightaware.com/)
+	Click the *Login* link near the upper left
+	Sign in with your credentials
+	Click the *My ADS-B* link near the upper left

**PlaneFinder.net**

+	Visit [https://planefinder.net/sharing/login](https://planefinder.net/sharing/login)
+	Sign in with your credentials
+	Click your profile avatar, located on the right of the top navigation bar, and then choose *Account Settings*.
+	In the section, **Your receivers**, click on the link that corresponds to the *share code* of the receiver you want to view.

---

## Conclusion

A Raspberry Pi with DVB-T USB dongle and open source software installed, works out to be a more affordable option for hobbyists when compared to the expensive higher-end ADS-B virtual radars such as the [Radarcape](http://modesbeast.com/radarcape.html) or [AirNav RadarBox](https://www.airnavsystems.com/RadarBox/index.html) or dedicated decoders like the [Mode-S Beast](http://modesbeast.com/scope.html) or [Kinetic SBS](http://www.kinetic.co.uk/index.php) models.  Plus, your costs are effectively offset by the premium memberships within a short period of time. This basic configuration can also be extended by adding a high gain antenna and filter to improve reception as well.

---

## Screenshot Gallery

<figure class="ink-image">
	<a href="{{ site.baseurl }}/assets/images/2018/04/13/raspberry-pi-real--time-flight-tracker-updated/Boeing_737_Max-flightradar24.com+live_stream.png">
		<img src="{{ site.baseurl }}/assets/images/2018/04/13/raspberry-pi-real--time-flight-tracker-updated/Boeing_737_Max-flightradar24.com+live_stream.png" alt="Boeing 737 MAX First Flight (January 29, 2016): FlightRadar24.com Track and Live Stream Landing">
	</a>
	<figcaption>
		Boeing 737 MAX First Flight (January 29, 2016): FlightRadar24.com Track and Live Stream Landing
	</figcaption>
</figure>

<div class="column-group horizontal-gutters">
	<div class="xlarge-50 large-50 medium-50 small-100 tiny-100">
		<figure class="ink-image">
			<a href="{{ site.baseurl }}/assets/images/2018/04/13/raspberry-pi-real--time-flight-tracker-updated/RESEARCH9.png">
				<img src="{{ site.baseurl }}/assets/images/2018/04/13/raspberry-pi-real--time-flight-tracker-updated/RESEARCH9.png" alt="FlightRadar24.com Screenshot">
			</a>
			<figcaption>
				FlightRadar24.com Screenshot
			</figcaption>
		</figure>
	</div>
	<div class="xlarge-50 large-50 medium-50 small-100 tiny-100">
		<figure class="ink-image">
			<a href="{{ site.baseurl }}/assets/images/2018/04/13/raspberry-pi-real--time-flight-tracker-updated/flightaware.com.png">
				<img src="{{ site.baseurl }}/assets/images/2018/04/13/raspberry-pi-real--time-flight-tracker-updated/flightaware.com.png" alt="FlightAware.com Screenshot">
			</a>
			<figcaption>
				FlightAware.com Screenshot
			</figcaption>
		</figure>
	</div>
</div>

---

## Additional Resources

+	[FlightAware: ADS-B Network News newsletter archive](https://us14.campaign-archive.com/home/?u=7bd8986a8ad54991c01e23939&id=a7b1182a9d)
+	["Fr24feed software Additional FAQs and Info"](http://forum.flightradar24.com/threads/9866-Fr24feed-software-Additional-FAQs-and-Info)
+	[flightaware/piaware/PiAware-Best-Practices.mediawiki](https://github.com/flightaware/piaware/blob/master/PiAware-Best-Practices.mediawiki)
+	[Virtual Radar from a Digital TV Dongle](http://www.arrl.org/files/file/QST/This%20Month%20in%20QST/January%202014/VirtualRadarJan2013QST.pdf)
+	[RTL-SDR.COM](http://www.rtl-sdr.com/)
+	[rtl-sdr – OsmoSDR](http://sdr.osmocom.org/trac/wiki/rtl-sdr)
