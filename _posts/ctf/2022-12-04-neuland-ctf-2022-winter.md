---
title: Neuland CTF 2022 Winter Writeup
date: 2022-12-04 00:00:00 +8000
categories: [Writeup, CTF]
tags: [neuland, ctf, writeup, reversing, web, pwn, osint, forensics, crypto, misc]
image:
  path: /assets/img/post/nlandctf2022/neuland-ctf-2022-winter-thumbnail.png
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Neuland CTF 2022 Winter
---
 
In this first writeup, I will describe how I found 15 flags in my first CTF. 


## About the CTF
On December 3, 2022, the second CTF of [THI's](https://thi.de/) computer science club [Neuland-Ingolstadt](https://neuland-ingolstadt.de/) took place.  
  
Our task in this jeopardy-style ctf was to solve challenges from the categories web, steganography, forensics, cryptography, open source intelligence, reverse engineering, pwn/binary exploitation, and miscellaneous. The goal was to find a so-called flag. (Flag pattern: `nland{.*}`)
  
## Preparation phase
Since this was my first CTF, I have been preparing for it for a few days
  
A few weeks ago, I started to compile all the information, findings, and tools I learned into a toolbox. I will continuously expand this toolbox during my studies, other CTF challenges, and my private projects in the next few years.  
So far, I have created documentation for more than 60 tools. I am also providing explanations about different vulnerabilities.  
![Screenshot of the hacker toolbox](/assets/img/post/nlandctf2022/toolbox-screenshot.png)  
  
Another useful preparation was to implement a downloading script, for the often-used CTF Dashboard [ctfd](https://ctfd.io/). With this simple script our team could download all challenges and files well organized in our git repository and was ready to hack only a few seconds after the start of the CTF.

## Writeups
* [Reversing](#reversing)
* * [Strings](#strings)
* * [Tracer](#tracer)
* [Web](#web)
* * [Latency](#latency)
* * [XML is stupid](#xml-is-stupid)
* [PWN](#pwn)
* * [Password Guessing](#password-guessing)
* [OSINT](#osint)
* * [Time Machine](#time-machine)
* * [Lost Connection](#lost-connection)
* [Forensics](#forensics)
* * [Based meme](#based-meme)
* * [Friend or foe](#friend-or-foe)
* [Cryptography](#cryptography)
* * [Bitwise Operator](#bitwise-operator)
* * [RSA](#rsa)
* * [Blackbox](#blackbox)
* [Misc](#misc)
* * [For the Gains](#for-the-gains)

## Reversing
### Strings
The first reverse engineering task was quite simple. The [strings](https://linux.die.net/man/1/strings) command of Linux can be used to print all printable characters in a file.  
![Get nland flag from strings binary](/assets/img/post/nlandctf2022/rev-strings.png)
This is usually one of the first things I try with binaries to get information about it.  
  
Flag: `nland{f0und_y0u}`
### Tracer
The task of this reverse engineering challenge was to find out the admin password (`nland{admin-password}`).  
  
Another try with strings resulted only in many useless 9 characters long strings.  
That's why I decided to load the program in Ghidra:
![Get nland flag from strings binary](/assets/img/post/nlandctf2022/rev-tracer-ghidra.png)
On the right side, you can see the decompiled code.  
But the important part started after about 5000 lines with variables with random hexadecimal values.  
You can see in line 5020 that the user input is compared with the admin password.
![Inspect program in gdb](/assets/img/post/nlandctf2022/rev-tracer-gdb.png)
So I started the program in my modified GDB.  I was able to find the strcmp function quite quickly and get the parameter values. (`asdf`(my input) and `42ceec6b744d41bc8044fee516003183`(the admin password))
  
Flag: `nland{42ceec6b744d41bc8044fee516003183}`
## Web
### Latency
Latency was a very simple web challenge.  
There was an input field for the IP address which could be used to send an ICMP request to the specified address. I immediately thought of `Command Injection`. So I added a `whoami` after the IP address with a semicolon (`127.0.0.1; whoami`) to see if the server returns something. And indeed the username was returned!  
  
Next, I printed the whole folder content with the `ls` command (`127.0.0.1; ls`) and saw that a `flag.txt` file existed. With a simple `127.0.0.1; cat flag.txt` I could output the flag.  
  
Flag: `nland{5h377s-4r3-3v3rywh3r3}`
### XML is stupid
The challenge `XML is stupid` was very simple. But out of stupidity and bad luck, I wasted more than half an hour on this five-minute task.  
  
But to explain the challenge: There was a web interface with a file upload for XML files. I immediately knew from the name of the challenge that it had to be an `XXE` attack (XML External Entity).  
The description also said that the flag was placed in the `/opt/next/flag.txt` location.  
Since I had already documented the XXE attack in detail in my toolbox, I already had a XXE exploit script. I adapted it to the according file path:  
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///opt/next/flag.txt"> ]>

<root>
  <first-name>&xxe;</first-name>
  <last-name>Mustermann</last-name>
</root>
```
When you uploaded the XML file, the flag was displayed in the browser.  
  
Now briefly to my problem described earlier. I have tried this variant and many other XML attack possibilities. However, when submitting, the page with the flag was not loaded and the page was frozen.  
After some time I asked the organizers for help. The result after a  few minutes of error handling was, that our campus network is  filtering malicious requests. So I had to switch to a public network to get the flag.  
  
Flag: `nland{y0u-sh0u7dn7-us3-xm7}`  
## PWN
### Password Guessing
Our first PWN task, which I solved with a team colleague, was `Password Guessing`. The task was to guess a constantly regenerating password.
![Password Guessing Code](/assets/img/post/nlandctf2022/pwn-pg-code.png)
The first thing I noticed when looking at the code was that the random seed for generating the random password in line 14 is always set to the current time.  
This means that the same "random" number is always output at the same time, both locally and on the server.
![Password generating code](/assets/img/post/nlandctf2022/pwn-pg-code-short.png)
I just shortened the code so that only the password was displayed on the console. Then I could pipe it to the specified Netcat service, which gave me the flag.
```shell
./pw2 | nc summit.informatik.sexy 8083
```
  
Flag: `nland{51mpl3_45_7h47}`
## OSINT
### Time Machine
A The hint in the first OSINT Challenge was that a few weeks ago, the searched flag was placed accidentally in the CTF registration page ([https://ctf.neuland-ingolstadt.de/](https://ctf.neuland-ingolstadt.de/)). However, this flag was immediately removed again.  
  
I immediately got the idea of the [Wayback Machine](https://web.archive.org/web/) and after a few seconds I was able to find a [backup with the flag](https://web.archive.org/web/20221024120345/https://ctf.neuland-ingolstadt.de/).  
![Wayback Machine](/assets/img/post/nlandctf2022/osint-time-machine.png)
  
Flag: `nland{7h3_1n732n37_n3v32_f029375}`
### Lost Connection
However, `Lost Connection` was a difficult task in contrast to `Time Machine`. The only information given was the email `elisabeth.hacker1337@gmail.com` and that the person was last seen in London. The goal was to find out the city of residence (Flag format: `nland{city-name}`).  
  
I searched all possible search engines for the email and the name. However, without even any success.  
Fortunately, a team colleague found a very helpful article about [Gmail OSINT](https://medium.com/hacking-info-sec/how-to-gmail-osint-like-a-boss-1ca4f55f55e2). This blog post described how to find out more information about a person via Gmail contacts.  
In the network tab, in the developer tools you can find a request which contains a userid (`114037517009879435623`) for `elisabeth.hacker1337@gmail.com`.    
![Gmail Contacts](/assets/img/post/nlandctf2022/osint-lc-gmail.png)
You can now attach this to Google Maps and get the recession which was written in `Oxford`. ([https://www.google.com/maps/contrib/114037517009879435623](https://www.google.com/maps/contrib/114037517009879435623))
![Gmail Maps](/assets/img/post/nlandctf2022/osint-lc-gm.png)
   
Flag: `nland{Oxford}`
## Forensics
### Based meme
The task was to restore the corrupted attachment of an old email.  
A quick look at the file showed that it had to be an image in Base64 encoding.
![Based Meme Email](/assets/img/post/nlandctf2022/forensics-based-meme-email.png)
I didn't wait long and copied the Base64 code to [CyberChef](https://cyberchef.org/) and selected the `Render Image` option. With the input format `Base64`, CyberChef gave me the rendered image with the meme:  
> Das Internet ist für uns alle Neuland. - Angela M.

![Base64 Cyberchef](/assets/img/post/nlandctf2022/forensics-based-meme-cyberchef.png)
This is not only the origin of the name for the Neuland club but was also the flag we were looking for in this challenge.  
  
Flag: `nland{Das_Internet_ist_für_uns_alle_NEULAND_}`
### Friend or foe
The `friend or foe` challenge was also one of the easier tasks.  
The goal was to clone a given Physical Access Card.
![RFID Card](/assets/img/post/nlandctf2022/forensics-friend-or-foe-rfid.jpeg)
I quickly realized that it was an RFID card, which I could easily read using the [NFC Tools App](https://www.wakdev.com/en/apps/nfc-tools-android.html) on my smartphone. The flag was stored in plain text in `Record 1`.  
![NFC Tools](/assets/img/post/nlandctf2022/forensics-friend-or-foe-nfc.jpg)
  
Flag: `nland{1ff_m42k_2}`
## Cryptography
### Bitwise Operator
In this Cryptography Challenge the hint `The key to happiness is love.` and a secret message was given:
```text
`bo`juc:7?m:?Q6?9y?;=Q>~=<:9><s.
```
The title suggested that it could be a bitwise operation. And so I decided to open [CyberChef](https://cyberchef.org/) and select the `XOR` operation.  
Since a key is now required for the XOR operation, I noticed that the description draws attention to `love` as the key.  
Luckily this was true and I was able to find the flag.
![CyberChef](/assets/img/post/nlandctf2022/crypto-bitwise.png)

Flag: `nland{m491c41_817w153_0p324702}`
### RSA 
In the `RSA` challenge the three values given were `n`, `c`, and `e`.  
Since `n` had a relatively small value, it was possible to calculate the factors q and p, which can be used to recover the private key.  
However, I simply used the rather old-looking page [dcode](https://www.dcode.fr/rsa-cipher) for calculating and decrypting this cipher.
![RSA](/assets/img/post/nlandctf2022/crypto-rsa.png)
Flag: `nland{7h15_15_f1n3}`
### Blackbox
Given was a black box with a blinking light. In addition, the information was provided that the sender sent his name.
![Blackbox](/assets/img/post/nlandctf2022/crypto-blackbox.jpeg)
It quickly became clear that this had to be Morse code: `... .- --`.  
Translated it means `SAM`, which was the name of the sender and in this case also the flag.

Flag: `nland{SAM}`
## Misc
### For the Gains
For the Gains was a fitting miscellaneous challenge.  
The task was to convert the nucleotide sequence into an amino acid sequence.  (Flag format: `nland{amino-acid-sequence}`)
  
After some googling, I found the [Github Repository transeq](https://github.com/glarue/transeq). This CLI can convert the nucleotide sequence into an amino acid sequence. 
![Blackbox](/assets/img/post/nlandctf2022/misc-for-the-gains.png)

Flag: `nland{MASKVQLVFLFLFLCAMWASPSAASRDEPNDPMMKRFEEWMAEYGRVYKDDDEKMRRFQIFKNNVKHIETFNSRNENSYTLGINQFTDMTKSEFVAQYTGVSLPLNIEREPVVSFDDVNISAVPQSIDWRDYGAVNEVKNQNPCGSCWSFAAIATVEGIYKIKTGYLVSLSEQEVLDCAVSYGCKGGWVNKAYDFIISNNGVTTEENYPYLAYQGTCNANSFPNSAYITGYSYVRRNDERSMMYAVSNQPIAALIDASENFQYYNGGVFSGPCGTSLNHAITIIGYGQDSSGTKYWIVRNSWGSSWGEGGYVRMARGVSSSSGVCGIAMAPLFPTLQSGANAEVIKMVSET}`


## Conclusion
For my first CTF, the competition from Neuland went well. We were in the lead for a long time. 
![CTF Scoreboard Chart](/assets/img/post/nlandctf2022/conclusion-scoreboard-chart.png)
(Our Team was `H4CK3R5` with the red color)  
  
However, since this was our first CTF, we got our priorities wrong and didn't have time to solve the more difficult tasks at the end to stay in the lead. Nevertheless, we got enough points for fifth place out of about twenty teams.  
In my opinion, this is quite a good result for a CTF newcomer team. 
  
---
  
**Neuland-Ingolstadt Links:**
- [Website](https://neuland-ingolstadt.de/) 
- [Github](https://github.com/neuland-ingolstadt) 
- [Instagram](https://www.instagram.com/neuland_ingolstadt/)
- [LinkedIn](https://www.linkedin.com/company/neuland-ingolstadt)
