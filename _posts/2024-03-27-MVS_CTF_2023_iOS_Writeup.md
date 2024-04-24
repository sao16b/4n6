---
title: Magnet Virtual Summit CTF 2023 - iOS Writeup
description: Writeup for the iOS portion of the MVS 2023 CTF.
author: sosborn
date: 2024-03-27 12:27:00 -0400
categories: [CTF]
tags: [magnet, ios]

---

### Info

**Link**: <https://www.magnetforensics.com/blog/announcing-the-mvs-2023-ctf-winners-and-a-new-ctf-challenge/>

**Problem**: There's an iOS logical image to analyze and a series of questions to answer.

### 1. A few too many

**Prompt**: How many different email accounts did the user have?

Perhaps the easiest way of going about this would be by using Autopsy's Keyword List search, which one can use to return strings that match an email regex, as well as the number of files with hits per match. I let it run for a while, then exported the results to CSV. 

![Keyword List Email Results](/assets/img/2024-03-27/1_1.png){: width="600" }

blueisth3best@icloud.com has the most hits, and corresponds to the phone's iCloud account. A little further down is mborchardt@kurvalis.com, which can be found in the Chrome `Preferences`{: .filepath} file and Slack folders. Further down still is borchardtmichael78@gmail.com, and also michaelkborchardt@proton.me, both present in Chrome data, giving a total of **four** significant email addresses.

### 2. autoFill me in on the deets

**Prompt**: Which email, other than their own, was autofilled in Chrome?

In the Chrome data folder (`/private/var/mobile/Containers/Data/Application/0B468A6F-8837-4A85-BF4D-1EF523683946/Library/Google/Chrome/Default`{: .filepath}) there's a database `Web Data`{: .filepath} which contains an `autofill`{: .filepath} table. In this table there's an entry for **tlouis@kurvalis.com**.

![Chrome Web Data autofill Table](/assets/img/2024-03-27/2_1.png){: width="700" }

### 3. 1 fish 2 fish, red fish bluefish

**Prompt**: According to the user’s email accounts, what is his favorite color?

His iCloud email is blueisth3best@icloud.com, so it stands to reason his favorite color is **blue**.

### 4. Chef Boyardee 2.0

**Prompt**: At which market was the user viewing Chef Pasquale tomato sauce?

I found the photo library in `/private/var/mobile/Media/DCIM/100APPLE`{: .filepath}. `IMG_0034.MOV`{: .filepath} shows the Pasquale sauce in question.

![IMG_0034](/assets/img/2024-03-27/4_1.png){: width="500" }

Extracting the file and running exiftool on it gives GPS coordinates (`exiftool IMG_0034.MOV | grep GPS`).

![exiftool results](/assets/img/2024-03-27/4_2.png){: width="700" }

Plugging these into Google Maps shows the photo was taken at the **Marché Atwater** in Quebec.

![Marché Atwater](/assets/img/2024-03-27/4_3.png){: width="600" }

### 5. Staying stylish!

**Prompt**: What color shirt did the user chose to put their snapchat bitmoji in?

On this phone, Snapchat user data is stored in `/private/var/mobile/Containers/Data/Application/A5579AA5-A9D6-48BA-B937-4BFF7742ED88`{: .filepath}. In `Library/Caches/SCCache/com.pinterest.PINDiskCache.snapcodeData`{: .filepath} there's an interesting file named `qr-add%2F780e51cecf2a174138080341e5a3147f`{: .filepath} that contains JSON data, with a field `imageData`{: .filepath} corresponding to a base64ish string.

![Snapchat Cached Image File](/assets/img/2024-03-27/5_1.png){: width="700" }

Putting that string through a base64 to image converter (<https://base64.guru/converter/decode/image>) gives a Snapcode (note: I had to run it through the site's repair function first due to some Unicode characters towards the end of the string).

![Snapcode](/assets/img/2024-03-27/5_2.png){: width="300" }

Scanning the Snapcode with the Snapchat app on my phone reveals Michael's avatar, wearing a **green** sweater.

![Michael's Snapchat](/assets/img/2024-03-27/5_3.jpg){: width="400" }

I could have just looked up Michael's username, m_b227468, which is present in the `Documents/user.plist`{: .filepath} file within the Snapchat data folder, but I thought this roundabout way was cool.

### 6. Picking up Steam

**Prompt**: What server was the user interested in making?

Looking at his Chrome History (`/private/var/mobile/Containers/Data/Application/0B468A6F-8837-4A85-BF4D-1EF523683946/Library/Application Support/Google/Chrome/Default/History`{: .filepath}), Michael was interested in setting up a CS GO or Rust server.

![Chrome History](/assets/img/2024-03-27/6_1.png){: width="500" }

But, looking at some Discord snippets (`/private/var/mobile/Containers/Data/Application/75EA8999-B416-4238-9763-40959A79311B/Library/Caches/com.hammerandchisel.discord/fsCachedData/DC8F0B62-0041-4049-8679-73F597144048`{: .filepath}), Michael seems intent on setting up a **CSGO** server for an alcull945.

![Discord Log](/assets/img/2024-03-27/6_2.png){: width="700" }

### 7. Overlooking Excellence

**Prompt**: What Sports stadium was the user overlooking at Camilien-Houde belvedere?

Going back to Michael's photo library, there's many photos of a city vista, with a sign indicating places of interest. On one photo (`IMG_0023.HEIC`{: .filepath}), one such marked place is the Stade Olympique, or **Olympic Stadium**.

![Stade Olympique in IMG_0023](/assets/img/2024-03-27/7_1.png){: width="400" }

Running exiftool on the image, extracting the GPS coordinates (`exiftool IMG_0023.HEIC | grep GPS`), and putting them into Google Maps verifies Michael was overlooking the Camilien-Houde belvedere.

![Michael's Location](/assets/img/2024-03-27/7_2.png){: width="600" }

### 8. Out of this world

**Prompt**: Which terms and conditions site on Tik Tok is named after a space formation?

I extracted the TikTok data folder (`/private/var/mobile/Containers/Data/Application/4F9E5274-DDB7-422E-8629-234C84D24F4E`{: .filepath}) and ran a grep for TikTok URLs.

```bash
grep -EIroah 'https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&//=]*)' \
| sort \
| uniq \
| grep -E tiktok[\.]com
```

![exiftool Results](/assets/img/2024-03-27/8_1.png)

Combing through the list, there's a URL <https://www.tiktok.com/forest/nebula/ad_legal> that appears to be a Terms and Conditions site, so the space formation in question is **nebula**.

![Nebula ToC](/assets/img/2024-03-27/8_2.png){: width="600" }

### 9. You're going to crush this one!

**Prompt**: What light-hearted game did the user spend the most time on?

The iOS Screen Time database can be found at `/private/var/mobile/Library/Application Support/com.apple.remotemanagementd/RMAdminStore-Local.sqlite`{: .filepath}, and the `ZUSAGETIMEDITEM`{: .filepath} table gives an idea of usage per application. I exported the table to CSV and sorted by the `ZTOTALTIMEINSECONDS`{: .filepath} column. A few rows down is com.midasplayer.apps.candycrushsaga (**Candy Crush Saga**) clocking in at 577 seconds.

![Screen Time Data](/assets/img/2024-03-27/9_1.png){: width="600" }

### 10. Which way?

**Prompt**: Which cardinal direction was the user turning when heading towards RHEINFAHRE?

One of the images in Michael's photo library (`IMG_0068.HEIC`{: .filepath}) features a sign for the RHEINFAHRE in question.

![IMG_0068](/assets/img/2024-03-27/10_1.png){: width="400" }

Running exiftool on the image gives the GPS coordinates (`exiftool IMG_0068.HEIC | grep GPS`). After putting them into Google Maps, we can use street view to see the way to RHEINFAHRE is South.

![Street View](/assets/img/2024-03-27/10_2.png){: width="400" }

### 11. First class seats out of here!

**Prompt**: What 4-star Airline flies the most passengers out of the same terminal our user flew out of in Germany?

Some network history can be found at `/private/var/preferences/com.apple.wifi.known-networks.plist`{: .filepath}. There's an entry for Airport-Frankfurt, added on January 5, 2023 that was accessed as late as 07:21:29 EST, or 12:21:29 CET.

![Known Networks](/assets/img/2024-03-27/11_1.png){: width="700" }

From the `Photos.sqlite`{: .filepath} database (`/private/var/mobile/Media/PhotoData`{: .filepath}), there's an entry in the `ZPHOTOSHIGHLIGHT`{: .filepath} table for Newark Liberty International Airport on December 23, 2022. On and after this date there is activity in Germany.

![Photos.sqlite](/assets/img/2024-03-27/11_2.png)

Using a new (to me) site <https://www.flightera.net>, I'm able to see there were three flights from Frankfurt to Newark Liberty on January 5th.

![Flightera Results](/assets/img/2024-03-27/11_3.png){: width="700" }

All these flights took off from Terminal 1. Whew, that was a lot of work! The Frankfurt Airport Wikipedia says Terminal 1 is mainly used by **Lufthansa**, and a quick Google search verifies its 4-star status.

### 12. Boosting into a new era

**Prompt**: The user was trying to learn German through an application, what promotion featuring a rocket was most commonly shown to the user?

Duolingo application data can be found in `/private/var/mobile/Containers/Data/Application/89A6AE48-C46D-4405-A187-C7FF439873F3`{: .filepath}. Here, I stumbled on a folder `Documents/plus-ad-video`{: .filepath} that contains a few promotional videos. The **Duolingo_NYPromo_2023_EN.mp4** video prominently features a rocket in its first frame.

![Duolingo Promos](/assets/img/2024-03-27/12_1.png){: width="500" }

### 13. Q-uestion

**Prompt**: What Chinese networking website was associated with Linkedin?

I pointed the grep I used in [task 8](#8-out-of-this-world) at the LinkedIn application directory (`/private/var/containers/Bundle/Application/4D867879-C7BD-4906-8865-EAE0AA4E6236`{: .filepath}). Taking into account the question title, I filter down to URLs with the letter q in them.

```bash
grep -EIroah 'https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&//=]*)' \
| sort \
| uniq \
| grep q
```

![grep Results](/assets/img/2024-03-27/13_1.png)

The first URL, **http://user.qzone.qq.com**, seems to be what we're looking for.

![QQZone](/assets/img/2024-03-27/13_2.png){: width="300"}

### 14. You are here

**Prompt**: Which airline lounge was viewed?

I did a keyword search for 'lounge' in Autopsy and came across a file `PPSQLDatabase.db`{: .filepath}, which seems to contain some interesting user personalization data. One such piece of data in the `loc_records`{: .filepath} table names the **Lufthansa Senator Lounge**. The `sources`{: .filepath} table indicates this came from Apple Maps activity.

![PPSQLDatabase.db](/assets/img/2024-03-27/14_1.png)

### 15. A river runs through it

**Prompt**: At which location did the user travel the most meters according to Apple? (City, Country)

I began nosing around in Apple's Health app folder (`/private/var/mobile/Library/Health`{: .filepath}), figuring it would probably be a good place for finding location and distance information. The `healthdb_secure.sqlite`{: .filepath} database turns out to have a lot of forensic value. In this database, it seems the `samples`{: .filepath} table indicates health information of certain types for "samples" of user activity, and the `quantity_samples`{: .filepath} table gives "quantities" corresponding to those activities. A `data_type`{: .filepath} of 8 in the `samples`{: .filepath} table corresponds to distance traveled in meters; those distances can be found in the `quantity_samples`{: .filepath} table. With the help of a SQL query, I was able to parse out some timestamps and meters from the database.

![Distances in Meters](/assets/img/2024-03-27/15_1.png){: width="400"}

So we're looking for location information timestamped between 694186518 and 694187107. There's another database I came across, `Cache.sqlite`{: .filepath} (`/private/var/mobile/Library/Caches/com.apple.routined`{: .filepath}) with a table `ZRTCLLOCATIONMO`{: .filepath} that contains location information and timestamps. Another SQL query later and I have a shortlist.

![Coordinates with Timestamps](/assets/img/2024-03-27/15_2.png){: width="400"}

All these coordinates point to **Eltville, Germany**.

![Eltville](/assets/img/2024-03-27/15_3.png){: width="500"}

### 16. Lo siento, its going to be a cold one

**Prompt**: What weather front was warned to the user by youtube?

I needed a little help on this one, but I now know where to find logs of iOS notifications: `/private/var/mobile/Library/DuetExpertCenter/streams/userNotificationEvents/local`{: .filepath}. In here there's a notification log `690753503959675`{: .filepath}.

![Notification Log](/assets/img/2024-03-27/16_1.png){: width="600"}

I extract the file and pipe the strings command into a grep (`strings 690753503959675 | grep -B 5 youtube`) to look for YouTube notifications.

![Youtube Notifications](/assets/img/2024-03-27/16_2.png){: width="600"}

A little bit down the results I come across a notification in Spanish, with the word 'frente' (front).

![Spanish Notification](/assets/img/2024-03-27/16_3.png){: width="400"}

Googling for the title fragments and the channel (Univision Noticias) gives the video, warning of a **frente ártico**, or **arctic front**.

![YouTube Video](/assets/img/2024-03-27/16_4.png){: width="400"}

## Debrief

My first iOS image, and I did not enjoy it very much! I probably need to do a few more to get more comfortable, but I didn't find the OS very intuitive or navigable; if I recall correctly, even the Android CTF challenge seemed worlds easier. Nevertheless, this was a good introduction to the Magnet CTF; I felt the challenges were an appropriate difficulty with interesting solutions, though I am missing the investigative component prevalent in the Belkasoft CTFs. I was proud of myself for being able to tackle most of the questions, and for my intuition in knowing where to begin looking for the answers.


