---
title: "BelkaCTF #5: Party Girl - MISSING Writeup"
description: Writeup for Belkasoft's Party Girl - MISSING CTF.
author: sosborn
date: 2023-06-27 13:35:00 -0400
categories: [CTF]
tags: [belkasoft, windows]
---

**Link**: <https://belkasoft.com/ctf_july_2022/index.php>

**Problem**: A 17-year old girl has gone missing after a party. We're tasked with finding out what happened, armed only with open-source tools and an E01 image of her laptop.

### 1. User (Baby - 150 points) - Who is the laptop user?

I opened the E01 in AccessData FTK Imager. The `SOFTWARE`{: .filepath} registry file contains information about user profiles, so I exported the file from `C:\Windows\config`{: .filepath} and stuck it into RegRipper. The ProfileList key reveals the `C:\Users\maria`{: .filepath} path is associated with SID **S-1-5-21-1751165416-2537581589-3438259779-1001**.

![RegRipper Results](/assets/img/2023-06-27_1/1_1.png){: width="300" }

Alternatively (and perhaps more easily), one can load the image into Autopsy and navigate to OS Accounts.

![OS Accounts](/assets/img/2023-06-27_1/1_2.png)

### 2. Missing stuff (Warmup - 200 points) - What phone model did the girl use?

Maybe we can use EXIF data from photos she took. A quick glance around the image shows a promising `Pictures`{: .filepath} folder in her OneDrive (`C:\Users\maria\OneDrive\Pictures`{: .filepath}). I exported the folder to my local machine and spun up Ubuntu. I ran the following command in the directory: `exiftool -r . | grep -B 20 'Model'`. This command runs exiftool recursively on the folder and pipes it into a grep command, which searches for the keyword 'Model' and prints out results with 20 prepending lines. The very first hit is on `C:\Users\maria\OneDrive\Pictures\Meh\1bb3ed5218a057d9e1798f05064854e5.jpg`{: .filepath}. Exiftool says the Artist is Maria, and the Camera Model is **iPhone 8**.

![Exiftool Results](/assets/img/2023-06-27_1/2_1.png){: width="500" }

### 3. Going further (Warmup - 200 points) - What is the version number of the girl's email software?

I thought diving back into the registry might be a good idea for this one. `NTUSER.DAT`{: .filepath} contains information regarding recent applications. If Maria used email frequently maybe we'd find something there. I exported the file from `C:\Users\maria`{: .filepath} and ran RegRipper on it. Searching for Mail shows the Start Menu Mail app points to Mozilla Thunderbird.

![RegRipper Start Menu](/assets/img/2023-06-27_1/3_1.png){: width="300" }

I get lucky with a search for Thunderbird in the same file and find version **91.10.0** in Maria's `Downloads`{: .filepath} folder.

![RegRipper Thunderbird](/assets/img/2023-06-27_1/3_2.png){: width="300" }

### 4. Boyfriend? (Baby - 150 points) - Who has recently invited the girl to meet up?

In Autopsy, I navigated to Data Artifacts -> E-Mail Messages -> Default -> Default to see Maria's local inbox. Sorting the results by Date Received shows an email from **jamegkmail@protonmail.com** on July 25th inviting Maria to a 'Unique Place.'

![Unique Place Email](/assets/img/2023-06-27_1/4_1.png)

### 5. Final! (Hard - 700 points) - What is the chat password?

First I need to find a chat application. Looking back through the `NTUSER.DAT`{: .filepath} file, I see an application called WickrMe that looks unfamiliar.

![RegRipper WickrMe](/assets/img/2023-06-27_1/5_1.png){: width="300" }

Lo and behold, it's a messaging app! Maybe I can find something in Maria's `AppData`{: .filepath} folder. I pull up Autopsy and navigate to `C:\Users\maria\AppData`{: .filepath}. I decide to look at the `Roaming`{: .filepath} folder first. There's a folder for `Wicker, LLC`{: .filepath} and in that folder is a file called `Wakemeup.txt`{: .filepath} containing a long string of characters. Strange.

![Wakemeup.txt](/assets/img/2023-06-27_1/5_2.png)

Here was the point where the unkempt troll in the back of my brain started chanting something familiar. "Base64, Base64, Base64," he rasped. I exported the file to my local machine and uploaded the file to good old <https://www.base64decode.org>.

![Base64decode Results](/assets/img/2023-06-27_1/5_3.png){: width="600" }

Hm. Still looks like Base64. I uploaded the "decoded" file to <https://www.base64decode.org>. Same result: more Base64. I did this 6 or 7 times before I got tired and decided to write a script to speed things up.

```python
import base64

f = open("Wakemeup.txt", "r").read()
f = f.encode('ascii')
while(' '.encode('ascii') not in f):
	f = base64.b64decode(f)
print(f)
```

On a hunch I thought the decoded text might contain a space, which Base64 does not. So I looped over the decoded text until it found a space in the text, then printed out the result. Sure enough, running the script printed out the password: **julie@2000**.

![Script Results](/assets/img/2023-06-27_1/5_4.png){: width="600" }

### 6. Not final (Hard - 700 points) - What software was mentioned in the invitation?

I wasn't able to decrypt the `wickr_sqlite.db`{: .filepath} file the previous task referenced, but it turns out I didn't need to. In my perusal of `AppData`{: .filepath} I checked the `Local`{: .filepath} folder for good measure. There's a `Wickr, LLC`{: .filepath} folder containing a `WickrMe`{: .filepath} folder. Looking through the contents of the `WickrMe`{: .filepath} folder led me to the following path: `C:\Users\maria\AppData\Local\Wickr, LLC\WickrMe\temp\attachments`{: .filepath}. In that folder is an `invitation.ics`{: .filepath} file, only it's not a standard ics file. Autopsy shows that it actually contains two items: `eventpass.txt`{: .filepath} and `invitation.wav`{: .filepath}.

![Alleged ics File](/assets/img/2023-06-27_1/6_1.png)

I listened to the `invitation.wav`{: .filepath} file. Morse code! I exported the file and uploaded it to a Morse code decoder to reveal **SilentEye** as the software in question.

![Decoded Morse Code](/assets/img/2023-06-27_1/6_2.png){: width="400"}

### 7. BelkaRestaurant (Tricky - 500 points) - Where was Maria invited to?

Looking up SilentEye shows it's a steganography application. I downloaded it and loaded the `invitation.wav`{: .filepath} file into it to decode. Out pops `cup of coffees7eg.jpg`{: .filepath}.

![SilentEye](/assets/img/2023-06-27_1/7_1.png){: width="700"}

I looked through the EXIF data and tried a reverse Google image search but nothing fit as a possible answer. I hemmed and hawed for a little while before consulting the official writeup for a hint. It pointed me to two places: Maria's search history and the `eventpass.txt`{: .filepath} file I hadn't used yet, which contained the simple string 'specialevent.'

Back to Autopsy to look through Maria's search history (Data Artifacts -> Web History). I notice she visited <https://futureboy.us>, which is a site hosting a steganography tool! I uploaded the image and passed in 'specialevent' as the password, and we have our location: **Special Belkunir Pashe Restaurant**.

![Futureboy](/assets/img/2023-06-27_1/7_2.png){: width="300"}

### 8. Open to call (Baby - 150 points) - What is the restaurant phone number?

No results for Special Belkunir Pashe Restaurant anywhere I could find. I went back to the official writeup to learn I needed to use Open Street Map. But putting it into Open Street Map returned no results for me. Needed to be there, I guess.

### 9. Backup (Warmup - 200 points) - What kind of second factors can I use to download the girl's iCloud backup?

For this one we have four options:

- Maria's iPad, which is set as trusted. It is locked, but shows notifications with text
- Maria's iPhone
- Jame's iPhone, which is set as trusted
- Jame's iPad, which is logged as a family member for Maria

The site gives a link describing iCloud acquisition and analysis. It says there's only two types of second factors: SMS codes to a phone number linked to an iCloud account, and a code sent to a trusted device. That leaves options 1, 2, and 3, but we don't have Maria's phone, so the answers are **1** and **3**.

### 10. Mystery unveiled (Baby - 150 points) - Can you figure out where the girl is?

For this step the site linked a database file called `consolidated.db`{: .filepath} that was obtained from the iCloud backup. Opening it in DBBrowser for SQLite shows a series of tables.

![consolidated.db](/assets/img/2023-06-27_1/10_1.png){: width="300"}

GeoFences looks promising. Maybe I can get a lat/long? Browsing the table shows two records with lat/long information and timestamps. The most recent entry points to 47.5398556132234, 19.0302273633114.

![GeoFences Table](/assets/img/2023-06-27_1/10_2.png){: width="700"}

Sticking those coordinates into Google Maps gives us Maria's last known location: Saint Margaret Hospital in **Budapest, Hungary**.

![Maria's Last Location](/assets/img/2023-06-27_1/10_3.png){: width="500"}

### Debrief

I think this CTF was a perfect difficulty for a novice digital forensic examiner. None of the tasks felt too difficult, though I did consult the official writeup a couple of times when I got stuck, which I'm determined not to do anymore. I wouldn't get that luxury in a live CTF or real-life scenario, after all. This CTF also netted me some good experience with Autopsy, which I had barely touched. I'm feeling confident going into the next CTF.

