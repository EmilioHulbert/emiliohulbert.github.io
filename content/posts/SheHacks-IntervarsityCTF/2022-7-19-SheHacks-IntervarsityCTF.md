---
layout: post
title: "SheHacks-IntervarsityCTF"
date: 2022-07-19
categories: [ctf, shehacks]
tags: [Ctf]
---


This past weekend I got the opportunity to participate in the #ShehacksKEintervasityCTF which was amazing. I played as Team BitClan with my teammates and we managed to scoop second position. The challenges were amazing and I got to learn a thing or two.In this writeup,I will be explaining how I solved the challenges. Note that this is the approach I took to solve the challenges and not necessarily how they are supposed to be solved.

![image](/images/kcactf/1.png)

# Catalog
* [Osint](#osint)
* [Mobile App](#mobile-app)
* [Reverse Engineering](#re)
* [Binary Exploitation](#pwn)
* [Forensics](#forensics)
* [Web](#web)
* [Lessons Learnt](#lessons-learnt)



## OSINT
I only managed to solve two challenges in this category.

### Hunter 1

![image](/images/kcactf/2.png)

The first challenge (Hunter 1) tells that the team uncovered a malicious document with the sha256 hash of `bcff17a1885a330497594ca80b29257313195e3b065f674ad2d205b7c63b919d`. The first thing would be to check the hash of the file in google.

![image](/images/kcactf/3.png)

Among the results we found on google, we can see the hash among the result with the title of `2018-04-10 - Malspam pushing Gandcrab ransomware`. Since we are being asked the name of the ransomware behind the Malicious Office Document, we get our flag which is flag{gandcrab}, which is the name of the ransomware

### Last Hunt
This is one of the challenges i first blooded.It is an easy osint challenge that does not rely on solving the previous challenges in this category.This challenge is asking which other method of payment gandcrab ransomware requested other than BitCoin (BTC). We can search for this in google. Since we know that most threat actors use cryptocurrency as a form of payment, we can search using the following key terms.

![image](/images/kcactf/4.png)

From the second and third articles, we can see that gandcrab ransomware demanded for payment in a cryptocurrency known as DASH, hence the flag for the challenge was flag{dash}


### References
I only managed to solve the above two challenges in this category. Below are some references which can provide more information about solving challenges similar to the above

- https://hackmd.io/@tahaafarooq/usiu-ctf-2022-blockhain
- https://www.h4k-it.com/urchinsec-ctf-competition-2022/


## Mobile App
The next set of challenges were in the Mobile Category.

![image](/images/kcactf/5.png)

### FixMe
The first challenge in this category hints that there is a trick employed to disguise malware. We are also given a file named fixme.so . At first, I thought that this a shared library but after checking the type using the file command, I found that the file has an archive format. From there we can create a temporary folder where we can extract the contents of the file(/tmp/fixme), then extract the contents of the file to that folder using the unzip command as shown below

![image](/images/kcactf/7.png)

Once the contents are extracted, we can then change to the directory and try to view the contents of the files.

![image](/images/kcactf/8.png)

We find a FixmeKt.class file which contains the flag : flag{i_knew_youd_fix_me}

### Logd
In this challenge , we are told something about developers forgetting to turn off logging in their applications. We are also given an android application named logga.apk . Since I do not know much about reversing android applications, I used this online tool(`https://www.decompiler.com/`) which I found to be very nifty when it comes to decompiling android applications(You can also use other tools such as JADX & Frida to reverse engineer android apps). The tool allows you to upload an android apk and gives you the ability to download the decompiled files as a zip.

![image](/images/kcactf/9.png)

Once we download and extract the contents of the archive, we can view the directory structure and files using code editors such as sublime.
We get two folders (resources and sources), here we choose the source folder the navigate to (com/chalie/logga/MainActivity.java) file.

![image](/images/kcactf/10.png)

Looking at the source code above, we see that there is a notFlag string defined which is merged with another string(line 30) and logged when the enteredSecret string is not empty. The merged string will be the notFlag string i.e MZWGCZ33MFWHOYLZONPWG3 (line 13)
 plus the hardcoded string i.e DFMFXF6ZDFMJ2WOX3MN5TXG7IK (line 30) which results to `MZWGCZ33MFWHOYLZONPWG3DFMFXF6ZDFMJ2WOX3MN5TXG7IK`. This string looks like an encoding ,however, we don't know what encoding is used on the string. We can try to use the `magic` feature in `https://gchq.github.io/CyberChef/` which tells us that the encoding is base32 and gives us the flag : flag{always_clean_debug_logs}.

![image](/images/kcactf/11.png)

### Buits and Bytes

![image](/images/kcactf/6.png)

For this challenge, we are given an apk file and told that developers use a byte array trick to hide authentication and api keys and other sensitive information. We can decompile the apk using the same method as before and open the extracted files using sublime as shown

![image](/images/kcactf/12.png)

The MainActivity.java file did not seem to have anything interesting. This challenge took me a while to solve because I have little to no experience with android apps😅😅. Before despairing, I decided to check the application resources and files. After several minutes of checking the resource files, I found an interesting file at (byte.apk_Decompiler.com/resources/res/raw/oauth.txt)

![image](/images/kcactf/13.png)


Opening the file we get some numbers which look like ascii numbers, again we can convert them using `https://gchq.github.io/CyberChef/` which gives us the flag : flag{lol_we_use_byte_arrays_to_hide_stuff}

![image](/images/kcactf/14.png)


### Firirida
For this challenge, we are given an apk file which we are supposed to find the flag in. We can decompile the apk using the same method as before and open the extracted files using sublime as shown

![image](/images/kcactf/15.png)

Checking the `MainActivity.java` file, we see an interesting line at 15 which seems to load a library called firirida i.e `System.loadLibrary("firirida");` . Since the name is entered as a parameter using double quotes, i had the thought that this library can be found among the resource files.
Just like I had suspected, i found the library at (resources/lib/armeabi-v7a/libfiririda.so). Even though I used ghidra to obtain the flag, below is an easier way to find the flag.

![image](/images/kcactf/16.png)


### References
- https://httptoolkit.tech/blog/android-reverse-engineering/

## RE
There were two challenges in this category of which I only managed to solve the first one with help from @trustie

![image](/images/kcactf/17.png)


### Keylet
In this challenge we are given an android application and a prompt that says `I started writing Java for android , It doesn't really work as it's supposed to be !` . Just like the other android challenges , used `https://www.decompiler.com/` to decompile the application.

Once the apk is decompiled, you can open it with the code editor of your choice, for me I chose sublime. You can traverse to the `MainActivitiy.java` file as shown in the image below.
![image](/images/kcactf/18.png)

From the source code above, we see that the application accepts  user input, removes the text `'flag{'` string from the input (line 22) then checks the secret using the checkSecret function (line 26). If the secret is invalid, it will show 'Access Denied'. 

![image](/images/kcactf/19.png)

Looking at the checkSecret function, we can see that it compares the input string byte by byte with the flagBytes array in (line 38). The flagBytes array contains numbers that look like a Decimal representation of a string. We can try to decode the values using `https://gchq.github.io/CyberChef/`.To understand more about octal and decimal representation of characters check this page (https://jbwyatt.com/ascii.html)


![image](/images/kcactf/20.png)

After decoding, we get the string, we get the result `pe~eIS_Fun_Z`. The first 4 characters seem to be different from the last characters which are all uppercase. This could hint that the encoding on the first 4 characters is different from the last characters. I then tried to decode the first 4 strings as octal using cyberchef and got the following result.

![image](/images/kcactf/21.png)

We can now merge the two results to get the flag{JAVAIS_FUN_Z}



## Pwn
There were four challenges in this category of which I only managed to solve the first two 
![image](/images/kcactf/22.png)



### Overflow
The first challenge was quite simple. We get a prompt `'Overflow into the buffer to get the flag!'`. The first thing I did was to enter many A's on the program's user input to see how it would react.

![image](/images/kcactf/23.png)

Entering many A's gave us the flag ie. flag{buffer_overfl0w_into_m3m0ry}


## Forensics
The solutions for this category can be found on the author's blog here
- https://05t3.github.io/posts/SheHacksInterUniCTF/

and here:
- https://medium.com/@zemelusa/first-steps-to-volatile-memory-analysis-dcbd4d2d56a1

## PwnBox
The solution for this challenge can be found on the author's blog here
- https://hackmd.io/@tahaafarooq/kcau-bootcamp-pwnbox



## Web
The solution for this challenge can be found on the author's blog here
- https://trevorsaudi.com/posts/2022-06-13_zipslip-vulnerability-justctf2022/




[![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Femiliohulbert.github.io%2FShehacksKEintervasityCTF-Kca%2F&count_bg=%2379C83D&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=views&edge_flat=false)](https://hits.seeyoufarm.com)



<!-- https://hits.seeyoufarm.com/#badge -->