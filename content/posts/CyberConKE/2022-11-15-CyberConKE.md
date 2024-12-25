---
layout: post
title: "CyberConKE Finals CTF"
date: 2022-11-15
categories: [ctf, CyberConKE]
tags: [Ctf, CyberConKE]
---

This past weekend (12th Nov 2022), I got the opportunity to participate in the <a href="https://twitter.com/CyberConKe">#CyberConKE</a> Finals CTF at United States International University (USIU) which was an amazing experience. I played as Team ShellPhish with my teammates <a href="https://twitter.com/mug3njutsu">mug3njutsu</a> and <a href="https://twitter.com/levanto_0">levanto</a>, and we managed to attain first position. The challenges were amazing and I got to learn a thing or two. The challenges were hosted on cyberranges and are not available at the time of writing this writeup since the competition has already ended. Therefore, In this writeup,I will be covering some of the osint challenges that I can still remember and how I solved them. Below is the final scoreboard from the competition

![images](/images/CyberConKE/scoreboard.png)

`ㅤ`

# Catalog
* [Challenge 1](#challenge-1)
* [Challenge 2](#challenge-2)
* [Challenge 3](#challenge-3)
* [Challenge 4](#challenge-4)
* [Challenge 5](#challenge-5)
* [Challenge 6](#challenge-6)
* [Summary](#challenge-6)

#### Challenge 1
* `Challenge prompt`
	* An attacker called and SIGINT located the GSM cellphone tower it connected to. Using the details below, find the GPS coordinates of the cell tower and the closest institution to the cell tower. Format ccke{latitude, longitude, institution-initials} – Round off latitude and longitude to four decimal places.
	Cell tower details – MCC – 639, MNC – 2, LAC – 4087, cell ID – 12882

* `solution`
	* For this challenge, we are given some information related to a cell tower and we are supposed to find the GPS coordinates based on the information provided. We are given the MCC ,MNC, LAC and Cell Id of the tower. 
	* MCC (Mobile Country Codes) are used in Wireless Telephone Networks (GSM, CDMA, UMTS, etc.) to identify the country which a mobile subscriber belongs to. A Mobile Country Code identifies the country. For example, in China MCC is equal to 460, in USA - 310, Hungary - 216, Belorussia - 257, Kenya - 639.
	* MNC on the other hand stands for Mobile Network Code. This code identifies the mobile operator eg Safaricom or AT&T. The detailed table with MCC and MNC codes is available [here](https://cellidfinder.com/mcc-mnc).
	* Location Area Code (LAC) is a unique number of current location area. It's used by carriers to identify a location area where the cell towers are located. However it's not unique, different countries and carriers could have the same LAC. In general, if you know MCC, MNC and LAC, you could know the approximate location of a cell phone. 
	* CellID (CID) is used to identify a base transceiver station within an area identified by LAC. A cell tower could have more than one (1) cell ID. You know a Cell ID, you know the location of a cell phone. You know more than 3 cell towers, you can pinpoint a cell phone.
	* Using this information, we can convert these parameters to estimated coordinates and place a location of GSM base station on the Google map. There are some websites that allows us to convert these parameters to coordinates e.g [opencellid](https://opencellid.org).We can enter the parameters into the website as follows

	![image](/images/CyberConKE/opencellid-1.jpg)

	* The coordinates of the cell tower can be seen in the url `https://opencellid.org/#zoom=18&lat=-1.213303&lon=36.879043` and the closest institution to the cell tower is `usiu`. Therefore the flag is `ccke{-1.2133,36.8790,USIU}` 
	* Another tool one can use is `https://cellphonetrackers.org/gsm/gsm-tracker.php`. You can enter the values in the form and it will determine the location as shown below.
	
	![image](/images/CyberConKE/2.jpg)

##### References
- https://opencellid.org/#zoom=18&lat=-1.213303&lon=36.879043
- https://cellidfinder.com/mcc-mnc


#### Challenge 2
* `Challenge prompt`
	* Based on the previous question, identify the radio type for the cell tower, what are the two frequencies it operates at?

* `solution`
	* GSM frequency bands or frequency ranges are the cellular frequencies designated by the International Telecommunication Union for the operation of GSM mobile phones and other mobile devices.
	* Luckily , there is a website that contains countries and the frequencies that are mostly used in each of them. The table can be found [here](https://www.worldtimezone.com/gsm.html)

	![image](/images/CyberConKE/3.jpg)

	* From the image above, we can see that the common frequency bands are `900` and `1800`. Hence the flag is `ccke{900,1800}`


##### References
- https://en.wikipedia.org/wiki/GSM_frequency_bands
- https://www.worldtimezone.com/gsm.html



#### Challenge 3
* `Challenge prompt`
	* What is the name of the vessel with IMO: 9441130, MMSI 224402000?

* `solution`
	* The first thing I did here was to do a google search for `"IMO: 9441130, MMSI 224402000"`. Based on the google results, I discovered that the `IMO` and `MMSI` provided in the challenge description were for a ship. I also came across the following [website](https://www.myshiptracking.com/vessels/abel-matutes-mmsi-224402000-imo-9441130) that allows someone to track a ship based on the `IMO` and `MMSI`

	![image](/images/CyberConKE/4.jpg)

	* From the image above, we can see that the vessel is a `Passenger Ship` by the name `ABEL MATUTES`. Therefore the flag for the challenge was `ccke{ABELMATUTES}`

##### References
- https://www.google.com/search?q=%22IMO%3A+9441130%2C+MMSI+224402000%22
- https://www.myshiptracking.com/vessels/abel-matutes-mmsi-224402000-imo-9441130


#### Challenge 4
* `Challenge prompt`
	* Track the voyager 1 probe with your OSINT skills. What will be the distance in Km of the voyager 1 from earth on 18th November 2022 at 00:00hrs? Format: ccke{float}.

* `solution`
	* Voyager 1 is a space probe launched by NASA on September 5, 1977, as part of the Voyager program to study the outer Solar System and interstellar space beyond the Sun's heliosphere. To be able to know the distance in km of the voyager 1 from earth on `18th November 2022`, we need to find a website that allows us to track its movement. After googling for a while, I came across this [site](https://theskylive.com/voyager1-info) which does exactly that.

	![image](/images/CyberConKE/5.jpg)

	* Now we have to set the date and time to `18th November 2022` and `00:00hrs` respectively

	![image](/images/CyberConKE/6.jpg)

	* From the image above, we can see that the distance from earth in km is `23769.13 Million Km` hence the flag is `ccke{23769.13}`


##### References
- https://en.wikipedia.org/wiki/Voyager_1
- https://theskylive.com/planetarium?obj=voyager1&date=2022-11-18&h=00&m=00


#### Challenge 5
* `Challenge prompt`
	* Using the image below, find the following details: serial number and mode s code for the aircraft. Format: ccke{serialno,mode-s}
	![image](/images/CyberConKE/boeing.jpg)

* `solution`
	* Looking closely at the plane image, we can see the text `N881BK` written on it.
	* After performing a google search on the text, I found out that the plane is a `American Airlines Boeing 787-8 Dreamliner` and information related to the plane can be found using the following [link](https://pt.flightaware.com/resources/registration/N881BK) that contains the plane's information

	![image](/images/CyberConKE/8.jpg)

	* From the image above , we can see that the serial number is `66001` and mode-s is `AC21AF`

##### References
- https://pt.flightaware.com/resources/registration/N881BK
- https://www.flightradar24.com/data/aircraft/n881bk


#### Challenge 6
* `Challenge prompt`
	* Which airport was the image above first taken. Format ccke{firstname}
	
	![image](/images/CyberConKE/10.jpg)

	* The image above gives us some basic information about the plane and also contains somewhat the original image for the plane. Visiting the original image using the link `https://www.jetphotos.com/photo/10340292#modal-exif`, we can see the photo location at the bottom (`Charleston`) 

	![image](/images/CyberConKE/9.jpg)

##### References
- https://www.flightradar24.com/data/aircraft/n881bk
- https://www.jetphotos.com/photo/10340292#modal-exif


#### Summary
* As much as I would have loved to do a writeup on the memory forensics category, the challenges are no longer available on the platform.
* However , the author of the memory forensics challenges has a writeup here : [Oste's Blog](https://05t3.github.io/posts/CyberConFinals/)


[![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Femiliohulbert.github.io%2FCyberConKE%2F&count_bg=%2379C83D&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=false)](https://hits.seeyoufarm.com)