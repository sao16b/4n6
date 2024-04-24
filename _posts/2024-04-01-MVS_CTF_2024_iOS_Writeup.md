---
title: Magnet Virtual Summit CTF 2024 - iOS Writeup
description: Writeup for the iOS portion of the MVS 2024 CTF.
author: sosborn
date: 2024-04-04 20:37:00 -0400
categories: [CTF]
tags: [magnet, ios]

---

### Info

**Link**: <https://www.magnetforensics.com/blog/2024-magnet-virtual-summit-ctf-winners-and-another-chance-to-play/>

**Problem**: There's an iOS logical image to analyze and a series of questions to answer.

### 1. Why are your messages green?

**Prompt**: On what date did Rocco and Chadwick first meet in person according to their conversations? YYYY-MM-DD format

iPhone chat messages are stored in `/private/var/mobile/Library/SMS/sms.db`{: .filepath}. It's a very relational database, so I need to draft a SQL query to get three pieces of information: who the messages were to/from, what the message was, and the time the message was sent.

```sql
SELECT 
	chat.chat_identifier AS "TO", 
	message.text as "TEXT", 
	datetime(substr(message.date,1,9) + 978307200, 'unixepoch') as "TIME"
FROM chat_message_join
INNER JOIN chat ON chat.ROWID = chat_message_join.chat_id
INNER JOIN message ON message.ROWID = chat_message_join.message_id
ORDER BY message.date ASC
```

From the results of that query I can gather that Chadwick and Rocco met on **2023-12-17**.

![SMS Messages](/assets/img/2024-04-01/1_1.png){: width="400" }

### 2. Where /r u going on safari?

**Prompt**: What subreddit was visited in a browser?

In Autopsy, under `Data Artifacts -> Web History`{: .filepath}, according to the Safari history at `/private/var/mobile/Library/Safari/History.db`{: .filepath}, the user visited the **Twitch** subreddit.

![Reddit History](/assets/img/2024-04-01/2_1.png){: width="700" }

### 3. Don’t ghost me

**Prompt**: At what time did Chadwick get annoyed at MYAI? YYYY-MM-DD HH:MM:SS UTC

I do a keyword search for "myai" and come across a file `arroyo.db-wal`{: .filepath} which seems to contain some chat logs. This file is a write-ahead log for the database located at `/private/var/mobile/Containers/Data/Application/9A0EF110-47F4-45D4-B96D-C3EF301F18FC/Documents/user_scoped/29312c28c183c406b035a7b3d40e2c6921a13c1a99a71dca20d0062085989beb/arroyo/arroyo.db`{: .filepath}, deep in the Snapchat user data folders.

![arroyo.db-wal](/assets/img/2024-04-01/3_1.png){: width="600"}

I extract `arroyo.db`{: .filepath} and browse the data in DBBrowser. In the table `conversation_message`{: .filepath}, I identify the chat between Chadwick and the MyAI chatbot, and quickly form an SQL query to isolate just the messages between them.

```sql
SELECT
	conversation_message.sender_id as "FROM",
	conversation_message.message_content as "MESSAGE_BLOB",
	datetime(substr(conversation_message.creation_timestamp,1,10), 'unixepoch') as "TIME"
FROM conversation_message
WHERE conversation_message.client_conversation_id='054078ed-2781-51e5-95db-4038c876bd59'
ORDER BY conversation_message.creation_timestamp ASC
```

Chadwick (sender id 22da...) seemingly gets annoyed at **2023-12-26 23:27:45**.

![MyAI Messages](/assets/img/2024-04-01/3_2.png)

### 4. IMAGEine living in pain

**Prompt**: Chad seemed to be searching for pain relief medicine in a store, how much did it cost?

In `/private/var/mobile/Media/DCIM/100APPLE`{: .filepath} there's an image `IMG_0017.HEIC`{: .filepath} of an ArnicareGel unit with the price displayed underneath: **$10.99**.

![Arnicare](/assets/img/2024-04-01/4_1.png){: width="400"}

### 5. Your keyboard is salt-y

**Prompt**: How many total words were typed on the device?

It's so interesting that this is actually something that's logged on an iOS device. In the `user_model_database.sqlite`{: .filepath} database in `/private/var/mobile/Library/Keyboard`{: .filepath}, there's a table `usermodeldurablerecords`{: .filepath} with key/value pairs. For key `tium.totalWordsTyped`{: .filepath}, the value is **1814**.

![Total Words Typed](/assets/img/2024-04-01/5_1.png){: width="500"}

### 6. Build me up, buttercup

**Prompt**: What is the current build version?

In `/private/var/root/Library/MobileContainerManager/mcm_migration_status.plist`{: .filepath} there's a key `LastBuildInfo -> ProductBuildVersion`{: .filepath} with value **20F75**.

![LastBuildInfo](/assets/img/2024-04-01/6_1.png){: width="700"}

### 7. Answer the call

**Prompt**: What is the guild ID of the discord server Chad was in?

In the Discord user data folder (`/private/var/mobile/Containers/Data/Application/FE27BB5E-D91E-4417-8669-C68FD6C67A97`{: .filepath}) I find a cache database at `Library/Caches/com.hammerandchisel.discord/Cache.db`{: .filepath}. In the `cfurl_cache_response`{: .filepath} table, there are some entries for Discord API calls to a guild with ID **136986169563938816**.

![Discord Cache](/assets/img/2024-04-01/7_1.png)

### 8. Warning Signs

**Prompt**: How many days did it take Chad to be warned about his Data Usage?

Going back to `sms.db`{: .filepath}, Chad was sent a welcome message from Boost Mobile on 2023-11-29, and received a data warning from them on 2023-12-17, **18** days later. I had to perform an outer instead of an inner join on this one, so that all messages were displayed, even ones that didn't document a sender.

```sql
SELECT 
	chat.chat_identifier AS "TO", 
	message.text as "TEXT", 
	datetime(substr(message.date,1,9) + 978307200, 'unixepoch') as "TIME"
FROM chat_message_join
INNER JOIN chat ON chat.ROWID = chat_message_join.chat_id
RIGHT OUTER JOIN message ON message.ROWID = chat_message_join.message_id
ORDER BY message.date ASC
```

![Boost Messages](/assets/img/2024-04-01/8_1.png){: width="500"}

### 9. Watching streams to stay current

**Prompt**: What is the name of Chad’s streaming channel?

This one took a while, and I kind of happened on the answer accidentally. I was looking through the iOS notifications (`/private/var/mobile/Library/DuetExpertCenter/streams/userNotificationEvents/local/722077640915524`{: .filepath}), when I noticed a Twitter notification indicating (one of his) usernames to be @GardenGamer95. 

![Twitter Notification](/assets/img/2024-04-01/9_1.png){: width="400"}

I looked up '"GardenGamer95" twitter' and one of the first results is actually a YouTube video from his streaming channel, **ChadwickGames**.

![ChadwickGames](/assets/img/2024-04-01/9_2.png){: width="400"}

### 10. One is The Loneliest Number

**Prompt**: What question did Chadwick ask to AI?

Autopsy lists ChatGPT among his installed programs (`Data Artifacts -> Installed Programs`{: .filepath}). Nosing around the user data folder (`/private/var/mobile/Containers/Data/Application/6BFA5EA3-61CB-4652-A60A-2A955B651E05`{: .filepath}) leads me to a folder (`Library/Application Support/conversations-b5c12911-e3c0-4961-bbe7-aec0a3ec3dd6`{: .filepath}) with three JSON files in it. These files seem to be records of Chadwick's interactions with ChatGPT.

He asks the AI many questions, including "What is doxing," "How to be a good gamer," "How to subtly dox without getting caught," and "**How to make online friends**".

![ChatGPT](/assets/img/2024-04-01/10_1.png)

### 11. Watch me sUAVely win this game

**Prompt**: How many kills did Chad have on his CoD Mobile winning game?

On Chadwick's YouTube channel, there's a video titled "Final Kills Lead to CoD Mobile Win!" Watching the video shows Chadwick had **7** kills at the end of the game.

![CoD Game](/assets/img/2024-04-01/11_1.png){: width="700"}

### 12. For when I can’t Find My gear

**Prompt**: What outdoor activity store did Chadwick Visit?

In `/private/var/mobile/Library/Caches/com.apple.findmy.fmipcore`{: .filepath} there are a few data files that look interesting, including one named `Devices.data`{: .filepath}. In here there's `lastConnected`{: .filepath} location information for the iPhone's owner.

![Cached Find My Data](/assets/img/2024-04-01/12_1.png){: width="500"}

Plugging this address in to Google Maps shows it's a store called **Neptune Mountaineering**.

![Find My Maps](/assets/img/2024-04-01/12_2.png){: width="600"}

### 13. Just a couple steps away

**Prompt**: How many steps did Chad take on 12/3/2023?

Magnet seems to like the Health database. I recall from the [previous iOS CTF](https://sao16b.github.io/4n6/posts/MVS_CTF_2023_iOS_Writeup/#15-a-river-runs-through-it) that the `healthdb_secure.sqlite`{: .filepath} database in `/private/var/mobile/Library/Health`{: .filepath} could be of use. I extract the database and form the following SQL query, after finding that the number of steps correlates to `data_type`{: .filepath} 7.

```sql
SELECT
	datetime(samples.start_date+978307200, 'unixepoch') as "START DATE",
	datetime(samples.end_date+978307200, 'unixepoch') as "END DATE",
	quantity as "STEPS"
FROM
	samples
LEFT OUTER JOIN
	quantity_samples
ON
	samples.data_id = quantity_samples.data_id
WHERE
	samples.data_type = 7
ORDER BY "START DATE"
```

After executing the query, I find there are four records for 12/03. Adding up the number of steps gives **968**.

![healthdb_secure.sqlite Steps](/assets/img/2024-04-01/13_1.png){: width="400"}

### 14. Another regularly scheduled program

**Prompt**: What Tattoo shop was visited on 12/27/2023?

I export the notable location database I came across in the [previous iOS CTF](https://sao16b.github.io/4n6/posts/MVS_CTF_2023_iOS_Writeup/#15-a-river-runs-through-it) at `/private/var/mobile/Library/Caches/com.apple.routined/Cache.sqlite`{: .filepath} and form a SQL query to extract some generalized latitudes/longitudes for 12/27/2023. 

```sql
SELECT DISTINCT
	ROUND(ZRTCLLOCATIONMO.ZLATITUDE,3) AS "LATITUDE",
	ROUND(ZRTCLLOCATIONMO.ZLONGITUDE,3) AS "LONGITUDE",
	substr(DATETIME(ZRTCLLOCATIONMO.ZTIMESTAMP+978307200, 'unixepoch'),11,3) AS "HOUR"
FROM ZRTCLLOCATIONMO
WHERE DATETIME(ZRTCLLOCATIONMO.ZTIMESTAMP+978307200, 'unixepoch') >= '2023-12-27'
ORDER BY "TIME" ASC
```

![Cache.sqlite](/assets/img/2024-04-01/14_1.png){: width="500"}

It appears that for the location data we have, Chadwick didn't stray too far from the S Broadway address. Looking at the shopping center a little closer, there's a tattoo shop called **Auspicious Tattoo** it's likely he visited.

![Auspicious Tattoo](/assets/img/2024-04-01/14_2.png){: width="500"}

### 15. I hear Stanley cups are all the rage

**Prompt**: What was the final score of the hockey game Chad went to? (home – away)

Chad took some pictures of a hockey game at the Ball Arena in Denver, CO (`IMG_0030`{: .filepath}, `IMG_0031`{: .filepath}, `IMG_0032`{: .filepath}).

![Hockey Game](/assets/img/2024-04-01/15_1.png){: width="400"}

Running exiftool on these shows that they were taken on 12/21/2023. Looking up "ball arena hockey 12/21/2023" gives the teams (Colorado Avalanche vs. Ottawa Senators) and final score of **6-4**.

![Score](/assets/img/2024-04-01/15_2.png){: width="500"}

### 16. Devil is in the details

**Prompt**: Whose bitmoji is dressed like a devil?

I needed a little help with this one, but now I know that there's plenty more information on iOS notifications in `/private/var/mobile/Library/UserNotifications`{: .filepath}. I extracted the folder and ran a grep for bitmoji (`grep -r bitmoji UserNotifications/`). Only one file matched: `UserNotifications/BC777906-DA27-4839-A269-37D489012B7A/DeliveredNotifications.plist`{: .filepath}. 

I opened the file in plist Editor and did a search for bitmoji, coming across a couple of bitmoji URLs, including <https://images.bitmoji.com/render/panel/10226594-482842799_5-s5-v1.png?transparent=1>. The bitmoji at this URL is dressed like a devil. Surrounding plist values indicate the bitmoji belongs to **Sofiakhan**.

![Plist Bitmoji](/assets/img/2024-04-01/16_1.png){: width="600"}

### 17. Excuse Moi?  What did you say?

**Prompt**: What is the content of the 2nd message that Chad deleted on Dec 18, 2023

I go back to `sms.db`{: .filepath} and with a modified SQL query, see the deleted messages and their GUIDs.

```sql
SELECT 
	chat.chat_identifier AS "TO", 
	message.guid as "MESSAGE GUID",
	message.text as "TEXT", 
	message.is_from_me as "FROM CHAD?",
	datetime(substr(message.date,1,9) + 978307200, 'unixepoch') as "TIME"
FROM chat_message_join
INNER JOIN chat ON chat.ROWID = chat_message_join.chat_id
INNER JOIN message ON message.ROWID = chat_message_join.message_id
ORDER BY message.date ASC
```

![Deleted Messages](/assets/img/2024-04-01/17_1.png){: width="600"}

I take note of the GUID of the second message and open `sms.db`{: .filepath} in HxD. I do a search for the GUID and uncover the deleted message: **Excuse me?! That's quite a bold statement considering I'm the one who walked away with a black eye and spent $30 last night on products to avoid one!**

![Deleted Message Content](/assets/img/2024-04-01/17_2.png){: width="500"}

### 18. Boost this server

**Prompt**: What is the 16 character carrier code?

I go to the iLEAPP Report generated by the iLEAPP Analyzer, and just for kicks click on the Cellular Wireless tab. There's an entry for `com.apple.carrier_1`{: .filepath} with a 16 character code: **310240_GID1-6432**, which iLEAPP pulled from `/private/var/wireless/Library/Preferences/com.apple.commcenter.plist`{: .filepath}.

![Carrier Code](/assets/img/2024-04-01/18_1.png){: width="500"}

### 19. The easy way or the hard way

**Prompt**: What is the timestamp of the message Chad sent to Rocco but was never recieved? YYYY-MM-DD HH:MM:SS UTC

On the Android image, Rocco's texts can be found in `/data/data/com.android.providers.telephony/databases/mmssms.db`{: .filepath}. Using the SQL query below, I find that Rocco's last text with Chad was sent on 2023-12-20 23:44:01.

```sql
SELECT 
	sms.address AS "TO", 
	sms.body as "TEXT",
	datetime(substr(sms.date, 1, 10), 'unixepoch') as "TIME"
FROM sms
ORDER BY sms.date ASC
```

![Rocco's Texts](/assets/img/2024-04-01/19_1.png){: width="400"}

Chad sent a message to Rocco on **2023-12-21 06:29:36** seeking to make amends, but Rocco did not receive it.

![Chad's Texts](/assets/img/2024-04-01/19_2.png){: width="400"}

### 20. Its been a long time

**Prompt**: When did Chad last login to Facebook? YYYY-MM-DD HH:MM:SS UTC

In the Facebook user data folder (`/private/var/mobile/Containers/Data/Application/BF2FEA88-C397-405D-90EE-A56B2720896C`{: .filepath}), there are two databases that look interesting in the `Documents`{: .filepath} folder, `time_in_app_61554675133740.db`{: .filepath} and `time_in_app_61555027042760.db`{: .filepath}. In the latter, assumedly the most recent, there's a `metadata`{: .filepath} table containing key/value paris. The `last_logging_timestamp`{: .filepath} key has a value of 1703712895. 

![Time in App DB](/assets/img/2024-04-01/20_1.png){: width="500"}

Plugging this into <https://www.epochconverter.com> gives the date **2023-12-27 21:34:55**.

### 21. Can anyone Kelp?

**Prompt**: What game was Chad asking to know the strategy to?

I did a keyword search in Autopsy for "strategy" and came across a file in the Facebook user data caches that indicates Chadwick was looking for assistance with the game **Terrarium**.

![FB Terrarium Post](/assets/img/2024-04-01/21_1.png){: width="500"}

### 22. Chat GPT is my PREFERENCE for AI

**Prompt**: What is the ChatGPT userID associated with chawickmr95@gmail.com

In the ChatGPT user data preferences folder (`/private/var/mobile/Containers/Data/Application/6BFA5EA3-61CB-4652-A60A-2A955B651E05/Library/Preferences`{: .filepath}), I searched through the plists and found the `userID`{: .filepath} in `com.openai.chat.StatsigService.plist`{: .filepath}: **user-xurgQ0xumvrujH5ESG17Yhcw**.

![ChatGPT Preferences](/assets/img/2024-04-01/22_1.png){: width="600"}

### 23. Read my mind

**Prompt**: What message was sent to Rocco in a video game

In the Call of Duty user data folder (`/private/var/mobile/Containers/Data/Application/3690AAA8-713A-482B-92F1-3F7D3BCC73E6`{: .filepath}), I comb through the `Documents`{: .filepath} and come across another folder named `ChatCache`{: .filepath} with a file `2023-12-20`{: .filepath} that indicates Chad sent the message **I know youre reading my messages**.

![CoD Message](/assets/img/2024-04-01/23_1.png){: width="500"}

### 24. Season’s Greetings

**Prompt**: What was the first emoji that was sent to Susan?

Using the same SQL query from [question 17](#17-excuse-moi--what-did-you-say), I see some messages in `sms.db`{: .filepath} between Chad and Susan. Two messages from Chad don't have any content. 

![Susan Messages](/assets/img/2024-04-01/24_1.png){: width="500"}

Just as in [question 17](#17-excuse-moi--what-did-you-say), I take note of the first message's GUID, open `sms.db`{: .filepath} in HxD, and do a search. I find the message, and the emoji bytes: F09F8E84. 

![Susan HxD](/assets/img/2024-04-01/24_2.png){: width="500"}

I look these up and the first result indicates they correspond to the **Christmas Tree** emoji. Although, according to other writeups, the answer is apparently **Potted Plant** when using Magnet? Doesn't make sense to me. 

![Bytes Search](/assets/img/2024-04-01/24_3.png){: width="500"}

### 25. Follow the Breadcrumbs

**Prompt**: How many times did Chad’s keyboard become visible within the Amazon app on 12/24/2023?

I learned a lot more than I bargained for about SEGB files on this one. Biome is an iOS service that records all kinds of iPhone usage data and data streams. This data can be found in `/private/var/mobile/Library/Biome/streams`{: .filepath}. One of the folders under the `public`{: .filepath} folder caught my eye: `TextInputSession`{: .filepath}. In there, under `local`{: .filepath}, there's a file `722655735826773`{: .filepath} that is reminiscent of the iOS notification streams I've found previously. Interestingly, there are some references to Amazon in this file.

![TextInputSession](/assets/img/2024-04-01/25_1.png){: width="500"}

I do a little research on Biome's SEGB format, coming across a repository <https://github.com/cclgroupltd/ccl-segb.git> that parses these files into a more useful format. I pull up `722655735826773`{: .filepath} in HxD, and go line by line through their script, trying to identify the components of a singular record in this file. Once I felt I had a good handle on the record structure, I copy and slightly modify the script from their README to run the `ccl_segb1.py`{: .filepath} module against the file.

```python
import sys
import ccl_segb1

input_path = "722655735826773"

# An open stream can be read using: read_segb1_stream
for record in ccl_segb1.read_segb1_file(input_path):
    offset = record.data_start_offset
    data = record.data
    ts1 = record.timestamp1
    ts2 = record.timestamp2

    print(offset, ts1, ts2)
    print(data)
```

I pipe the script output into a grep for Amazon (`python3 /path/to/script.py | grep -B 1 amazon`).

![SEGB Parser into Greps](/assets/img/2024-04-01/25_2.png){: width="600"}

From the output, it seems there were **2** instances of a text input stream (i.e. keyboard input) for the Amazon application on 12/24.


### Debrief
This one was a doozy! 25 questions, and a lot of them were head scratchers. I was excited to flex some old SQL knowledge this go-around. Overall I felt this challenge was very well-rounded, though I'll still complain about the sheer image size, and that some of the challenge wasn't self-contained, even though I realize that in the real world, both are prevalent issues.
