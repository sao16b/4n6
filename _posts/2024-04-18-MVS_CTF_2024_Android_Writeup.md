---
title: Magnet Virtual Summit CTF 2024 - Android Writeup
description: Writeup for the Android portion of the MVS 2024 CTF.
author: sosborn
date: 2024-04-23 10:00:00 -0400
categories: [CTF]
tags: [magnet, android]

---

### Info

**Link**: <https://www.magnetforensics.com/blog/2024-magnet-virtual-summit-ctf-winners-and-another-chance-to-play/>

**Problem**: There's an Android logical image to analyze and a series of questions to answer.

### 1. Press x to Respawn

**Prompt**: On what platform did Rocco share his Call of Duty Username?

The Call of Duty folder is in `/data/data/com.activision.callofduty.shooter`{: .filepath}. Under `shared_prefs`{: .filepath}, there's an XML file `__hs_lite_sdk_store.xml`{: .filepath} that identifies Rocco's username as OkClick5789.

![CoD Username](/assets/img/2024-04-18/1_1.png){: width="500" }

A quick keyword search in Autopsy gives a hit in `/data/data/com.twitter.android/databases/1719897971716685824-dm.db-wal`{: .filepath}; Rocco shared his username in a **Twitter** DM.

![Twitter CoD Username](/assets/img/2024-04-18/1_2.png){: width="500" }

### 2. Warm Up

**Prompt**: What Southern state’s sports team did Rocco search up?

In Autopsy, under Data Artifacts -> Web History, Chrome History reveals Rocco searched up the **Louisiana Ragin' Cajuns**.

![Chrome History](/assets/img/2024-04-18/2_1.png){: width="600" }

### 3. Can you Handle this

**Prompt**: What was Rocco’s Twitter account name?

Back in the Twitter databases (`/data/data/com.twitter.android/databases`{: .filepath}), I found Rocco's username in the `users`{: .filepath} table of the `1719897971716685824-66.db`{: .filepath} database: **RoccoSachs96775**.

![Twitter Username](/assets/img/2024-04-18/3_1.png){: width="400" }

### 4. Need to reach those heights

**Prompt**: What is the SIM operator name?

SIM information can be found in the `telephony.db`{: .filepath} database, which I found in the device encrypted storage at `/data/user_de/0/com.android.providers.telephony/databases`{: .filepath}. The `siminfo`{: .filepath} table contains three entries with two carriers: T-Mobile and **Boost Mobile**. Considering that Boost Mobile has the data_roaming column set to 1, and the iso_country column set to 'us', that's probably the primary SIM operator.

![telephony.db](/assets/img/2024-04-18/4_1.png){: width="600" }

### 5. Not to be basic but...

**Prompt**: What is the default Internet Browser?

According to the Autopsy Data Artifacts -> Web History findings, it seems that **Chrome** is the most prevalent browser that was used.

![Chrome History](/assets/img/2024-04-18/5_1.png){: width="600"}

New artifact alert! It seems you can find out default applications from the `roles.xml`{: .filepath} file, which can be found on this phone at `/data/misc_de/0/apexdata/com.android.permission`{: .filepath}. This file lists Chrome as the default browser application.

![roles.xml](/assets/img/2024-04-18/5_2.png){: width="400"}

### 6. Survival Mode Activated

**Prompt**: What conference did Rocco show interest in?

In Autopsy, under Data Artifacts -> Web History, the Chrome History artifact indicates Rocco was interested in **Preppercon**.

![Preppercon](/assets/img/2024-04-18/6_1.png){: width="600"}

### 7. Sign me up!

**Prompt**: What email is associated with the device?

Android account information can be found in `/system_ce/0/accounts_ce.db`{: .filepath} or `/system_de/0/accounts_de.db`{: .filepath}. In both of these databases, there's a table `accounts`{: .filepath}, and both contain the email address **roccotsachs@gmail.com**.

![Email Address](/assets/img/2024-04-18/7_1.png){: width="400"}

### 8. Not so popular

**Prompt**: How many messages were sent from Rocco in Twitter Direct Messages?

In the Twitter database (`/data/data/com.twitter.android/databases/1719897971716685824-66.db`{: .filepath}), there's a table `conversation_entries`{: .filepath} that contains direct messages and user ids. From the filename, and from the `users`{: .filepath} table from [task 3](#3-can-you-handle-this), Rocco's user id seems to be 1719897971716685824. 

In the `conversation_entries`{: .filepath} table, it's just a matter of summing up the DMs corresponding to that ID, which comes out to be **8**.

![Twitter DMs](/assets/img/2024-04-18/8_1.png){: width="600"}

### 9. No two cents about them

**Prompt**: According to exCHANGEs in discord with Chad, what did Chad want back from Rocco?

Combing through the Discord folders, I stumble on a database at `/data/data/com.discord/files/kv-storage/@account.1185636389107273799/a`{: .filepath} that contains Discord messages in the table `messages0`{: .filepath}. In this table, there are records between Chad and Rocco that indicate Chad wanted **money** back from Rocco after he scammed him.

![Discord Message](/assets/img/2024-04-18/9_1.png){: width="600"}

### 10. You can never be too ready

**Prompt**: How many additional survival tips were provided in the $9 book Rocco was looking into?

In Autopsy, under Data Artifacts -> Web History, there are a couple Amazon entries.

![Amazon Entries](/assets/img/2024-04-18/10_1.png){: width="700"}

Following the URLs takes me to a page for a ~9 dollar book: How to Fight a Bear...and Win: And **72** Other Real Survival Tips We Hope You'll Never Need.

![How To Fight a Bear and Win](/assets/img/2024-04-18/10_2.png){: width="500"}

Rocco also took a photo of the book with a $9 sticker on it; it's located in `/data/media/0/DCIM/Camera/PXL_20231215_202654750.jpg`{: .filepath}.

![Book Picture](/assets/img/2024-04-18/10_3.png){: width="400"}

### 11. Tag your’re it!

**Prompt**: What city was the user in when they identified an AirTag on them?

In the Discord chat messages, there's one message from Rocco that was sent on 2023-12-27 indicating they had become aware of the AirTag. 

![Discord AirTag Message](/assets/img/2024-04-18/11_1.png){: width="500"} 

Luckily, there are some photos that were taken on 2023-12-27 with GPS metadata, which I used exiftool to extract (`exiftool PXL_20231227_163049844.jpg | grep GPS`).

![Exiftool](/assets/img/2024-04-18/11_2.png){: width="500"}

Plugging these coordinates in to Google Maps reveals Rocco was in **Windsor, ON**.

![Windsor, ON](/assets/img/2024-04-18/11_3.png){: width="500"}

### 12. A game of Cat and Mouse

**Prompt**: What game did two beloved cartoon charachters promote in an Ad?

This one is a ridiculous challenge, and I ended up consulting [Kevin Pagaro's writeup](https://www.stark4n6.com/2024/03/magnet-virtual-summit-2024-ctf-android.html) for this one. The "ad" is actually part of a tutorial located in the Android tips directory at `/data/data/com.google.android.apps.tips/files/download/asset/83c4649ef9ea3b1825f2ee682accc363a31a0e5d`{: .filepath}. Autopsy doesn't display it, so I extracted it and tacked on a .mp4 to the filename. 3/4 of the way through, the game **Tom and Jerry: Chase** is shown.

![Tom and Jerry](/assets/img/2024-04-18/12_1.png){: width="500"}

I think this would take a while to find, but a way to find it would be to look through Autopsy's files through File Views -> File Types -> By MIME Type -> video.

### 13. Always achieving new heights

**Prompt**: What was the new score achieved on the video game Rocco watched on Youtube?

There's a `statuses`{: .filepath} table in the Twitter database (`/data/data/com.twitter.android/databases/1719897971716685824-66.db`{: .filepath}) that contains tweets that Rocco viewed, posted, and retweeted. One of these tweets links a YouTube video of a new Subway Surfers high score.

![YouTube Tweet](/assets/img/2024-04-18/13_1.png){: width="700"}

The high score achieved in the video is **5187**.

![High Score](/assets/img/2024-04-18/13_2.png){: width="500"}

### 14. LIVE your life

**Prompt**: What two sports did Rocco capture in a photo?

In the photos directory (`/data/media/0/DCIM/Camera`{: .filepath}) there's a live photo `PXL_20231218_020011968.jpg`{: .filepath} where I can just barely make out **golfing** and **skiing**.

![Trivia](/assets/img/2024-04-18/14_1.png){: width="500"}

### 15. Remember your floaties

**Prompt**: What fun outdoor activity location was searched for?

In Autopsy, under Data Artifacts -> Web History, there's an entry from Chrome History for **Big Water Campgrounds** in **Timmins, Ontario**. 

![Big Water Campgrounds](/assets/img/2024-04-18/15_1.png){: width="600"}

### 16. R-E-J-E-C-T-E-D Rejected

**Prompt**: When was the last shutdown that was initiated by Rocco? (YYYY-MM-DD HH:MM:SS) UTC 24 hour time.

There's an interesting folder `/data/system/shutdown-checkpoints`{: .filepath} that contains files with information about shutdowns, including those requested by the user. One file, `checkpoints-1703807249418`{: .filepath} contains the latest timestamp for a user-requested shutdown at **2023-12-28 23:47:29 UTC**.

![Shutdown Request](/assets/img/2024-04-18/16_1.png){: width="500"}

### 17. Out of Stock

**Prompt**: What is the most recent score in Subway Surfer?

It sure is a hit to the confidence to have to consult Kevin's writeup again, but I need to keep reminding myself that I'm still learning. And now I know a little more about Android than I did before!

Android keeps track of recent activity in `/data/system_ce/0/recent_tasks`{: .filepath}. In here, there's a file `256_task.xml`{: .filepath} that references a Subway Surfer session and a snapshot.

![256_task.xml](/assets/img/2024-04-18/17_1.png){: width="400"}

One directory up, there's a folder named `snapshots`{: .filepath}. In here, there's a file `256.jpg`{: .filepath} that is seemingly linked to the `256_task.xml`{: .filepath} file, and reveals the score achieved in the latest session: **1899**.

![256.jpg](/assets/img/2024-04-18/17_2.png){: width="500"}

### 18. So Salty!

**Prompt**: What is the handle of the person who is talking about how upset they are with  Rocco?

I recall seeing in the Discord messages with Chad an exchange about a Twitter account that was causing Rocco grief.

![Chat 1](/assets/img/2024-04-18/18_1.png){: width="600"}

![Chat 2](/assets/img/2024-04-18/18_2.png){: width="600"}

I find a few screenshots of the tweets in `/data/media/0/Pictures/Screenshots`{: .filepath} that gives the handle: **@larissajenna9**.

![Screenshot](/assets/img/2024-04-18/18_3.png){: width="300"}

### 19. Don’t let them see you down

**Prompt**: What was added using Photoshop?

In `/data/media/0/Pictures`{: .filepath} there's a folder `Photoshop Express`{: .filepath} with four screenshots of a Connections game. It seems Rocco might have removed the "Next Time!" text.

![Screenshot 1](/assets/img/2024-04-18/19_1.png){: width="300"}

![Screenshot 2](/assets/img/2024-04-18/19_2.png){: width="300"}

It doesn't seem like Connections would say "Next Time!" when he assumedly won the game. I navigate to the `Screenshots`{: .filepath} folder in `/data/media/0/Pictures`{: .filepath} and find the original screenshot, without the **Success** sticker and "Good Job!" text, which he added.

![Original screenshot](/assets/img/2024-04-18/19_3.png){: width="300"}

### 20. It’s the eye of the tiger

**Prompt**: When is Rocco’s Bday? (YYYY-MM-DD)

I know Facebook keeps track of birthdays, and luckily, we have a Facebook archive. I find Rocco's birthday in `/facebook-61554919820462-2024-01-06-49fzodcA/personal_information/profile_information/profile_information.html`{: .filepath}: **1974-09-29**.

![Facebook Birthday](/assets/img/2024-04-18/20_1.png){: width="600"}

### 21. Secrets Secrets are no Fun

**Prompt**: What did Rocco search in the App Store to download the app used to hide photos?

Google Play data can be found in `/data/data/com.android.vending`{: .filepath}. Under `databases`{: .filepath}, I find a database `suggestions.db`{: .filepath} that contains "queries". One of these is **calculator vault**, which is an app that is used to hide photos.

![Calculator Vault](/assets/img/2024-04-18/21_1.png){: width="400"}

### 22. Stalker Alert

**Prompt**: Shortly after logging into Facebook with IP address 72.38.231.98, a photo was taken. Where was this photo taken?

The Facebook archive stores information about logins/logouts at `/facebook-61554919820462-2024-01-06-49fzodcA/security_and_login_information/logins_and_logouts.html`{: .filepath}. The login from 72.38.231.98 took place on 2023-12-27 16:16:01 UTC. 

![Facebook Logins](/assets/img/2024-04-18/22_1.png){: width="600"}

In Rocco's photos (`/data/media/0/DCIM/Camera`{: .filepath}) the photo with the closest timestamp is `PXL_20231227_163049844.jpg`{: .filepath}. Running a grep for GPS on the EXIF data (`exiftool PXL_20231227_163049844.jpg | grep GPS`) gives the coordinates.

![GPS Coords](/assets/img/2024-04-18/22_2.png){: width="500"}

Plugging these coordinates in to Google Maps shows the photo was taken at **Devonshire Mall, 3100 Howard Ave Unit B7, Windsor, ON N8X 3Y8, Canada**. 

![Google Maps](/assets/img/2024-04-18/22_3.png){: width="500"}


### Debrief

It was nice to get back into Android for a change. There's a lot more to the OS than I figured, and I learned a few new artifacts that I think will come extremely in handy for future challenges. I hope to participate live at next year's summit, and see if I can hold my own alongside the DF greats.
