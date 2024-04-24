---
title: "BelkaCTF #2: Drug Dealer Case Writeup"
description: Writeup for Belkasoft's Drug Dealer Case CTF.
author: sosborn
date: 2023-07-11 13:30:00 -0400
categories: [CTF]
tags: [belkasoft, android]
---

### Info

**Link**: <https://belkasoft.com/ctf_may/>

**Problem**: A suspicious-looking man was detained, and traces of drugs were found on him. An image of his Android phone was taken, and we're tasked with finding out if this man is connected to the local drug ring. My first foray into phone forensics!

### 1. Owner (Baby - 160 points)

**Prompt**: What is the full name of the phone owner?

After a good deal of navigating around the file system, I came across a file `com.whatsapp_preferences_light.xml`{: .filepath} in `/data/data/com.whatsapp/shared_prefs`{: .filepath}. In here was a key called `push_name`{: .filepath} with string value **Derek Hor**.

![Whatsapp Name](/assets/img/2023-07-11/1_1.png){: width="400" }

In the SUBJECT's google history (found in `/data/data/com.android.chrome/app_chrome/Default/History`{: .filepath}) there are a few searches relevant to knee pain.

![Chrome History](/assets/img/2023-07-11/1_2.png){: width="700" }

There's an X-ray image of someone's knees in `/data/media/0/DCIM/Camera`{: .filepath}. The patient's name, Derek Hor is in the top left.

![X-ray](/assets/img/2023-07-11/1_3.png){: width="500" }

### 2. Supervisor (Warmup - 135 points)

**Prompt**: What is the phone number he reported to about drug delivery?

In `/data/data/com.android.providers.contacts/databases`{: .filepath} there's a database `contacts2.db`{: .filepath}. In that database there's a table called `raw_contacts`{: .filepath} which contains the names and numbers of Derek's contacts, one of whom is named Boss with associated number **12395104974**.

![Contacts2.db](/assets/img/2023-07-11/2_1.png){: width="600" }

### 3. Destinations (Tricky - 829 points)

**Prompt**: What were the suspect's delivery locations on the night of arrest?

At first I think exiftool may be of some help; in my searching I came across some photos of the SUBJECT holding cash, possibly after just completing a drug deal. I export the photos from `/data/media/0/DCIM/Camera`{: .filepath}, run exiftool on the latest ones from April 17th, and pipe that into a grep for 'GPS Position' (`exiftool . | grep 'GPS Position'`), getting a batch of coordinates back.

![Exiftool Results](/assets/img/2023-07-11/3_1.png){: width="700" }

However, April 17th was not the date of arrest, as there are artifacts on the phone dating April 18th. So I keep digging.

On April 18th, Derek sent a WhatsApp message to Boss with a link to a photo album, likely containing pics of the drug deals.

![Whatsapp Message](/assets/img/2023-07-11/3_2.png)

This sends me to the `/data/data/com.android.chrome/app_chrome/Default`{: .filepath} folder to look at his web history, which is in a file called `History`{: .filepath}. There looks to be a few requests to Google Maps in the table called `urls`{: .filepath}, and there's also a few requests to that album.

![Google Maps History](/assets/img/2023-07-11/3_3.png){: width="700" }

![Album History](/assets/img/2023-07-11/3_4.png){: width="700" }

There's another table called `visits`{: .filepath} which contains timestamps that link back to the urls on the `id`{: .filepath} column. Unfortunately, Autopsy doesn't do this for me, so I export both tables and open them in Excel. On the `visits`{: .filepath} file, I sort by descending timestamp, and look for url id 154, which corresponds to the album URL. Any Google Maps url visited just before that is probably a location we're looking for.

![Visits Table](/assets/img/2023-07-11/3_5.png){: width="300" }

(url 97 corresponds to the SUBJECT's homepage on the album-hosting site)

Cross-referencing with the `urls`{: .filepath} table gives us:

![URLs Table](/assets/img/2023-07-11/3_6.png)

We can pull out the lat/longs from the URLs to get the answers:

- **33.4845,-111.877215**
- **33.374304,-112.1035501**
- **33.5542226,-111.9340928**

### 4. Job experience (Baby - 348 points)

**Prompt**: How long has the suspect been acting as a drug dealer?

Going back to the WhatsApp history, there's a few interchanges between Derek and a friend. Derek mentions an interview on June 20, 2020, seemingly indicating that he got a new job.

![Interview Message](/assets/img/2023-07-11/4_1.png){: width="400" }

April 18, 2021 - June 20, 2020 = **303 days**.

### 5. Salary (Tricky - 1000 points)

**Prompt**: From what Bitcoin wallet did he get paid the last time for his job?

Looking at Derek's WhatsApp messages, the latest bitcoin transaction from his boss was 0.0913 on April 15, 2021 at around 13:30 EDT, or 17:30 UTC.

![Bitcoin Message](/assets/img/2023-07-11/5_1.png)

A few messages later, Derek remarks he sees a little less than that amount.

![Complaint Message](/assets/img/2023-07-11/5_2.png)

Lucky for us, we can track bitcoin transactions, and we have enough information to do so. Using <https://blockchair.com>, we can filter down by date (April 15th, 2021) and amount (around 0.913). One transaction seems to line up with the time frame and amount.

![Bitcoin Transaction](/assets/img/2023-07-11/5_3.png){: width="600" }

Examining the transaction and exporting the receipt gives the source wallet: **113JqY3CqsQPT7EN6wj5tRAVKftEP9rQC**.

![Bitcoin Invoice](/assets/img/2023-07-11/5_4.png){: width="500" }

### 6. Supplier (Hard - 444 points)

**Prompt**: What is the phone number of the drug supplier?

I recall seeing a backup for the Signal application in `/data/media/0/Signal/Backups`{: .filepath}, and a photo in `/data/media/0/Pictures/Screenshots`{: .filepath} called `Screenshot_20201220-210242_Signal.png`{: .filepath} that shows the backup password partially obscured, and the word keep scrawled above it.

![Keep Screenshot](/assets/img/2023-07-11/6_1.png){: width="500" }

I also remember seeing a folder containing the word keep...ah, yes, `com.google.android.keep`{: .filepath} in `/data/data`{: .filepath}. In the `databases`{: .filepath} folder within that folder, there's a database called `keep.db`{: .filepath}. In the table `list_item`{: .filepath}, there's a field called `text`{: .filepath}, and one of the records has 6 groups of 5 digits which seem to line up with the password to the backup.

![Backup Password](/assets/img/2023-07-11/6_2.png){: width="700" }

I export the Signal backup and download a tool called signal-back (<https://github.com/xeals/signal-back>), which I ran on the backup, providing the password at the prompt. I was able to get messages, but no phone numbers when outputting to a CSV format, and I could only get phone numbers to show when formatting results as raw Go struct, which is disgusting to look through. 10 minutes or so of wading through garbage, I find two phone numbers and profile names, +13148346839 associated with Signal profile "horatio0.42k" (Derek) and **+14233767293** associated with our supplier, "X."

![Signal Go Struct](/assets/img/2023-07-11/6_3.png)

### 7. Refill (Tricky - 547 points)

**Prompt**: When was the last time the suspect met his supplier?

The Signal messages with the supplier seem to indicate a meeting was supposed to take place at 33.508136, -112.148462 on 12/17 at 2pm, but was cancelled.

![Signal Messages](/assets/img/2023-07-11/7_1.png){: width="700"}

Maybe they met up later. Navigating to `/data/data/com.android.providers.calendar/databases`{: .filepath} there's a database called `calendar.db`{: .filepath}, and a table within it called `Events`{: .filepath}. There's an event called Pizza delivery with `eventLocation`{: .filepath} set to 33.508146,-112.148462. Typo in latitude aside, this looks pretty familiar, and the timestamp indicates this was supposed to take place 12/17 at 2pm Arizona time.

![Events](/assets/img/2023-07-11/7_2.png){: width="700" }

Looks like there's a few more records for Pizza deliveries, the latest one taking place on **April 10, 2021 at 00:30:00** Arizona time.

![Pizza Delivery Events](/assets/img/2023-07-11/7_3.png){: width="700" }

### 8. IMEI (Warmup - 691 points)

**Prompt**: What is the supplier's phone IMEI identifier?

We're given a link to <https://cellrecordslookup.nsa.fyi/>, which provides IMEIs given latitude, longitude, and date and time (MST). We have five total pizza delivery events, one of which we know was cancelled. We have latitudes/longitudes and timestamps for each of these events. I pull all the IMEIs for those four events and put them in an Excel spreadsheet. Then, I highlight the records that are duplicated.

![IMEI Spreadsheet](/assets/img/2023-07-11/8_1.png){: width="400" }

Only two IMEIs are present across all four events: 350236009513272 (Derek) and **332182208414842** (supplier).

![Identified IMEIs](/assets/img/2023-07-11/8_2.png){: width="400" }

### Debrief

This one was a bit easier than the previous CTF. I still have to figure out my way around Bitcoin, and I could use some more practice navigating phone file systems. I was especially proud of myself for finding the answer to [task 6](#6-supplier-hard---444-points); sometimes grueling directory searching pays off. Speaking of, I'm proud of myself for my consistent (so far) ability to recall files and information I searched through earlier. I think that's come in handy a few times these past CTFs. Anyway, one more to go! And then I'm thinking maybe a big one, like the Magnet Virtual Summit CTF from earlier this year.

