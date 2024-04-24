---
title: "BelkaCTF #6: BogusBill Writeup"
description: Writeup for Belkasoft's BogusBill CTF.
author: sosborn
date: 2024-04-07 09:00:00 -0400
categories: [CTF]
tags: [belkasoft, windows, ios]
---

### Info

**Link**: <https://belkasoft.com/belkactf6/>

**Problem**: A counterfeit bill was found at a corner store, and we're tasked to investigate leads with only our wits, smarts (assuming we have them), and an iPhone.

### 1. Ident (Baby - 100 points)

**Prompt**: What is the Apple ID used on the imaged iPhone?

In `/private/var/mobile/Library/Accounts/Accounts3.sqlite`{: .filepath} there's information about the phone's accounts in the `ZACCOUNT`{: .filepath} table. Here there are records indicating an iCloud account **billthemegakill@icloud.com**.

![Accounts3.sqlite](/assets/img/2024-04-07/1_1.png){: width="600" }

### 2. Namedrop (Baby - 100 points)

**Prompt**: What is the iPhone owner's full name?

In `/private/var/mobile/Library/AddressBook/AddressBook.sqlitedb`{: .filepath}, I stumbled on the owner's name in the `ABPerson`{: .filepath} table: **William Phorger**.

![AddressBook.sqlitedb](/assets/img/2024-04-07/2_1.png){: width="400" }

### 3. Conspirators (Warmup - 100 points)

**Prompt**: Which Telegram accounts did the owner discuss shady stuff with?

I hadn't heard of Telegram before this competition, so I was hung up on this one for a bit. In `/private/var/mobile/Containers/Shared/AppGroup/A667456A-6F8F-48C7-A8CF-37EFCC6BD644/telegram-data/account-112545592466388074/postbox/db/db_sqlite`{: .filepath}, one can find most Telegram user data, including accounts and messages. Unfortunately, this information is dispersed over 40+ tables. Navigating the database is unintuitive, to say the least.

I finally found my way to a table named `t2`{: .filepath} that contained what looked to be some usernames. I was able to pull four usernames out of this table that worked: **@JesusStreeton1999**, **@diddyflowers**, **@locknload771**, and **@Sm00thOperat0r**.

![JesusStreeton1999](/assets/img/2024-04-07/3_1.png){: width="500" }

![diddyflowers](/assets/img/2024-04-07/3_2.png){: width="500" }

![locknload771](/assets/img/2024-04-07/3_3.png){: width="500" }

![Sm00thOperat0r](/assets/img/2024-04-07/3_4.png){: width="500" }

### 4. Visit (Tricky - 127 points)

**Prompt**: Where does William live?

In Autopsy, under `Data Artifacts -> Installed Programs`{: .filepath}, I found that William has Uber installed on his phone. I navigated to the Uber user data folder (`/private/var/mobile/Containers/Data/Application/279D4B50-A8FC-4F16-86DA-61F3800F1E91`{: .filepath}) and came across a file `database.db`{: .filepath} in the `Documents`{: .filepath} folder. In here is a table called `places`{: .filepath}, which contains a few addresses. One of note is labeled Home, and corresponds to address **7402 Nottingham Ave, Saint-Louis, MO, 63119**.

![Uber Address](/assets/img/2024-04-07/4_1.png){: width="600" }

### 5. Username (Baby - 100 points)

**Prompt**: What is the username of the laptop user?

At this point, I was provided with the password to extract the second image file provided, that of a Windows laptop. Navigating to `C:\Users`{: .filepath}, it's trivial to see the username is **phorger**.

![Users Folder](/assets/img/2024-04-07/5_1.png){: width="300" }

### 6. April Paycheck (Warmup - 208 points)

**Prompt**: What is the amount of William's first take in April?

On the iPhone image, I recall seeing some Web history entries (`Data Artifacts -> Web History`{: .filepath}) for Crooked River Bank at <https://crbk.org>.

![CRBK](/assets/img/2024-04-07/6_1.png){: width="500" }

Navigating to the site, it seems I can recover the password if I have two pieces of information: William's username, and his card number.

![CRBK Forgot Password Page](/assets/img/2024-04-07/6_2.png){: width="400" }

Back in the Windows image, in `Data Artifacts -> Recent Documents`{: .filepath}, I see a couple LNK files for some PNG files in `C:\Users\phorger\Pictures`{: .filepath}. There, `Capture2.png`{: .filepath} shows some transactions.

![Capture2.png](/assets/img/2024-04-07/6_3.png){: width="600" }

In this folder we also have a Zone.Identifier data stream, which can indicate where files originated. The Zone.Identifier for that PNG indicates it came from Microsoft ScreenSketch.

![Zone.Identifier Stream](/assets/img/2024-04-07/6_4.png){: width="300" }

Maybe there's something to be found in the `AppData`{: .filepath} folder. In `C:\Users\phorger\AppData\Local\Packages\Microsoft.ScreenSketch_8wekyb3d8bbwe\TempState`{: .filepath}, I find what I'm looking for: cached screenshots, including the full version of `Capture2.png`{: .filepath}, which shows William's username, card number, and recent transactions.

![Full Capture2.png](/assets/img/2024-04-07/6_5.png){: width="500" }

Putting that information into <https://crbk.org/forgot-password> gives a new password in plaintext. This is a seriously insecure banking system!

![New Password](/assets/img/2024-04-07/6_6.png){: width="400" }

Oh well, not my problem. I log in with the new password, and see a new transaction for April, a credit for **7012.39**.

![April Transaction](/assets/img/2024-04-07/6_7.png){: width="500" }

### 7. Party (Tricky - 643 points)

**Prompt**: Where did the gang go to celebrate their success together in March?

Back in the Telegram database, there's a table `t7`{: .filepath} which seems to contain messages between William and his crew. One message suggests celebrating at a bar downtown.

![Celebration Telegram Message](/assets/img/2024-04-07/7_1.png){: width="500" }

Another message mentions an app Splitwise in relation to a great meal.

![Splitwise Telegram Message](/assets/img/2024-04-07/7_2.png){: width="500" }

Back on the iPhone, William has Splitwise installed. In the Splitwise user data folder (`/private/var/mobile/Containers/Data/Application/E951DC58-CF30-4518-9482-3E076D89A20B`{: .filepath}) I come across a `database.sqlite`{: .filepath} file in `Library/Application Support`{: .filepath} that has expense records in a table named `SWExpense`{: .filepath}. One of these records has a description "Get-together at Cunetto," and the Unix timestamp in the `date`{: .filepath} column corresponds to March 20. 

![Splitwise Records](/assets/img/2024-04-07/7_3.png){: width="600" }

Doing a quick search for "cunetto" shows a restaurant in St. Louis named **Cunetto House of Pasta**, this corresponds to lat/long **38.61034, -90.28030**.

![Cunetto](/assets/img/2024-04-07/7_4.png){: width="500" }

### 8. Crypto (Warmup - 286 points)

**Prompt**: Which file does the guy keep his encrypted container in?

Back in the Window's image, it doesn't take too long to stumble on a VHDX container hiding in an alternate data stream at **C:\Users\phorger\Documents\desktop.ini**.

![ADS VHDX](/assets/img/2024-04-07/8_1.png){: width="500" }

I extract the container and try to mount it, but it asks for a Bitlocker password or recovery key. I remember seeing a recovery key in Autopsy under `Data Artifacts -> Recent Documents`{: .filepath}, but a cursory keyword search doesn't seem to indicate it's resident on the Windows image. However, I have better luck on the iPhone. A keyword search reveals the recovery key in an obscure file `/private/var/mobile/Library/Spotlight/CoreSpotlight/NSFileProtectionCompleteUntilFirstUserAuthentication/index.spotlightV2/store.db`{: .filepath}.

![Recovery Key](/assets/img/2024-04-07/8_2.png){: width="400" }

### 9. Luxury (Warmup - 357 points)

**Prompt**: Which luxurious item did Phorger put his laundered money into?

With the recovery key found in the previous step, I'm able to mount the VHDX. In here there's an Excel file called `Spending.xlsx`{: .filepath} that contains an entry for a **Rolex Submariner Date 126619LB**.

![Rolex](/assets/img/2024-04-07/9_1.png){: width="400" }

### 10. Vacation (Tricky - 603 points)

**Prompt**: Which concert were Phorger and his girlfriend planning to attend in May?

I wasn't able to find the answer to this one in the time limit, nor were many people. I recall from the `Spending.xlsx`{: .filepath} file that William bought two round-trip tickets to Paris. So there's our first clue.

There are some records on the Windows image in Autopsy under `Data Artifacts -> Web History`{: .filepath} that indicate William was talking with his girlfriend (@diddyflowers) on the web version of Telegram, using Microsoft Edge.

![Telegram Web](/assets/img/2024-04-07/10_1.png){: width="600" }

There are a few places in Edge where cached data is stored. One I didn't know about was in `C:\Users\phorger\AppData\Local\Microsoft\Edge\User Data\Default\Service Worker\CacheStorage`{: .filepath}. In here is a folder `ba00623a413aef1be0c65618db85f0b8176e803d`{: .filepath} which seems to contain some folders with cached data from <https://web.telegram.org>. 

![Telegram Cached Data](/assets/img/2024-04-07/10_2.png){: width="600" }

I extract the folder and run my familiar grep to look at all the URLs.

```bash
grep -EIroah 'https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&//=]*)' \
| sort \
| uniq
```

From among those URLs, some stand out, mainly JPG files that seem to be associated with Telegram messages.

![Telegram Message Cached URLs](/assets/img/2024-04-07/10_3.png){: width="500" }

These URLs are not functional, but luckily we still have the cached files the URLs were pulled from. A simple grep (`grep -r msg6827079691`) gives the file names of interest.

![Telegram Message Cached Files](/assets/img/2024-04-07/10_4.png){: width="500" }

A quick look in Autopsy verifies that these files contain the JPEG data; peep the JFIF!

![Autopsy Cached File](/assets/img/2024-04-07/10_5.png){: width="400" }

I pull the first cached file, `c6b7544d-4000-44db-9f5d-41a962dfd3ce/341523c8cabb5a2c_0`{: .filepath}, into HxD. I recall from my experience with file signatures that a JPEG file starts with hex values 0xFFD8, followed closely by a JFIF string, and ends with hex 0xFFD9. The screenshot below shows the first hex data of the cached file. What's in blue is everything prior to the first 0xFFD8 hex value (highlighted), which is just before the JFIF string (also highlighted). I removed everything prior to FFD8, and followed the same process for everything after the last instance of 0xFFD9 at the end of the hex data. Then, I saved the modified file as a jpeg.

![Autopsy Cached File](/assets/img/2024-04-07/10_6.png){: width="600" }

Opening up the carved jpeg shows that they were planning on going to see **Eric Clapton** at the **Accor Arena** in **Paris**.

![Eric Clapton](/assets/img/2024-04-07/10_7.png){: width="500" }

### 11. Illustrator (Warmup - 438 points)

**Prompt**: What's the name of the person who designed the print template for the bills?

In the `t7`{: .filepath} table in the Telegram database, William seemingly talked to his girlfriend, first name Drew, about her designing a template for counterfeit bills. She agrees, and sends William a PSD file.

![t7 Message 1](/assets/img/2024-04-07/11_1.png){: width="500" }

![t7 Message 2](/assets/img/2024-04-07/11_2.png){: width="500" }

![t7 Message 3](/assets/img/2024-04-07/11_3.png){: width="500" }

In the mounted VHDX, there's a file `baker/presets/1.psd`{: .filepath} which shows a template for counterfeit bills.

![PSD Template](/assets/img/2024-04-07/11_4.png){: width="300" }

Running a strings/grep combo against this file for Drew (`strings 1.psd | grep Drew`) reveals her full name to be **Drew Linesworth**.

![Drew Linesworth](/assets/img/2024-04-07/11_5.png){: width="600" }

### 12. Homebrew Lab (Tricky - 438 points)

**Prompt**: Where is the makeshift lab where they printed the cash located?

In the Telegram database, William talks with his crew about a "secure location," and seemingly sends them an address via "secret chat," though I'm unable to find any traces of the address in the Telegram folders. 

![t7 Message 1](/assets/img/2024-04-07/12_1.png){: width="500" }

![t7 Message 2](/assets/img/2024-04-07/12_2.png){: width="500" }

However, there are a few places to find location data on the iPhone. One of them is in `/private/var/root/Library/Caches/locationd/consolidated.db`{: .filepath}. In the `GeoFences`{: .filepath} table, there's one entry for 38.5924436, -90.057325.

![consolidated.db](/assets/img/2024-04-07/12_3.png){: width="400" }

This location corresponds to address **900 N 88th St, East St Louis, IL 62203**.

![Address](/assets/img/2024-04-07/12_4.png){: width="500" }

### 13. Largest Batch (Warmup - 315 points)

**Prompt**: What is the precise moment their largest printing batch was completed?

There's a python script on the VHDX at `baker/bot.py`{: .filepath} that reveals how the bot works. It seems William provides the bot with a "recipe number," a "cake size," and finally the "number of cakes." The prompts from the bot are highlighted in the screenshot below.

![Bot.py](/assets/img/2024-04-07/13_1.png){: width="500" }

The bot seems to have been hosted in Telegram, as these same messages are present throughout the `t7`{: .filepath} table. This also means that William's specifications for each batch are present as well.

![Bot Message 1](/assets/img/2024-04-07/13_2.png){: width="500" }

![Bot Message 2](/assets/img/2024-04-07/13_3.png){: width="500" }

![Bot Message 3](/assets/img/2024-04-07/13_4.png){: width="500" }

![Bot Message 4](/assets/img/2024-04-07/13_5.png){: width="500" }

![Bot Message 5](/assets/img/2024-04-07/13_6.png){: width="500" }

Of the numbers William seemingly sent over Telegram, 160 was the largest.

![Largest Batch Size](/assets/img/2024-04-07/13_7.png){: width="500" }

After spending some time in Telegram, I figured out how to pull timestamps from each message in the `t7`{: .filepath} table from the `key`{: .filepath} column. I take 4 bytes starting at 12 bytes in, and plug it into <https://epochconverter.com>. The 160 message was sent at 2024-04-01 11:22:38 UTC.

![Timestamp in Hex](/assets/img/2024-04-07/13_8.png){: width="500" }

![Epoch Converter](/assets/img/2024-04-07/13_9.png){: width="500" }

Using this same method, I look through the `t7`{: .filepath} table and decode timestamps for the batch completed messages; the closest one I could find to that timestamp was **2024-04-02 22:44:30 UTC**.

### 14. Device (Tricky - 357 points)

**Prompt**: What's the printer model they used to print money?

On the Windows image in Autopsy, there are entries under `Data Artifacts -> USB Device Attached`{: .filepath} for an **HP LaserJet M1132 MFP**; this information was pulled from the `SYSTEM`{: .filepath} registry hive.

![USB Device Attached](/assets/img/2024-04-07/14_1.png){: width="500" }

### 15. Night Shift (Tricky - 579 points)

**Prompt**: Which ATM did Phorger test his bills on recently?

In the Telegram database `t7`{: .filepath} table, messages seem to indicate that there was a problem with one of the ATMs, and William went to check it out. 

![t7 Message 1](/assets/img/2024-04-07/15_1.png){: width="500" }

![t7 Message 2](/assets/img/2024-04-07/15_2.png){: width="500" }

![t7 Message 3](/assets/img/2024-04-07/15_3.png){: width="500" }

Using the same process of decoding the timestamps as in [task 13](#13-largest-batch-warmup---315-points), the timestamps of these messages seem to indicate William visited the ATM between the night of April 1st and the morning of April 3rd; this is also verified in the task description.

On the iPhone, in `/private/var/preferences/com.apple.wifi.known-networks.plist`{: .filepath}, there's an entry for a Wi-Fi network added on the morning of April 3rd, with SSID UCPLPublicWireless.

![Known Networks Plist](/assets/img/2024-04-07/15_4.png){: width="300" }

There's a site <https://wigle.net> that keeps a record of SSIDs discovered around the world. Using the SSID and BSSID from the plist, I can see if there are any hits for those values over St. Louis. Luckily, there's one.

![Wigle.net](/assets/img/2024-04-07/15_5.png){: width="500" }

This location turns out to be the University City Public Library at 6701 Delmar Blvd, University City, MO 63130. The closest ATM is the **Regions Bank ATM on Delmar Blvd**.

![ATM](/assets/img/2024-04-07/15_6.png){: width="500" }

### 16. Mole (Hard - 691 points)

**Prompt**: Who leaked the technical data on the bill validator to the gang?

This one was the last one that I was able to solve, and also the most satisfying! On the Windows image, in Autopsy under `Data Artifacts -> Recent Documents`{: .filepath}, I noticed a LNK file for the technical documentation in question. This documentation seemed to have been resident on the VHDX at one point; the `776AR-04U.PDF`{: .filepath} file referenced in the LNK record just above is on the VHDX. The technical documentation is not there anymore, though.

![ATM Documentation LNK File](/assets/img/2024-04-07/16_1.png){: width="500" }

At some point I realized that maybe the file data hadn't been overwritten yet. So I loaded the mounted VHDX as a logical drive into FTK Imager, which can detect deleted files. Sure enough, the ATM documentation pops up.

![Deleted ATM Documentation](/assets/img/2024-04-07/16_2.png){: width="500" }

I export the PDF from FTK Imager and open it up with Adobe Acrobat. The moment I open it, a warning pops up saying a signature is invalid. Interesting. I open up Signature Panel and we have our mole: **Kenneth Leek**.

![Signature Invalid](/assets/img/2024-04-07/16_3.png){: width="600" }

### 17. Financial Institution (Hard - 829 points)

**Prompt**: Which offshore financial institution did the gang bank with?

In the `t7`{: .filepath} table in the Telegram databse, William seems to receive a Shortcut from Chase that encrypts messages with a secret key. There seems to be some of these messages in the table as well.

![t7 Message 1](/assets/img/2024-04-07/17_1.png){: width="500" }

![t7 Message 2](/assets/img/2024-04-07/17_2.png){: width="500" }

![t7 Message 3](/assets/img/2024-04-07/17_3.png){: width="500" }

![t7 Message 4](/assets/img/2024-04-07/17_4.png){: width="500" }

On the iPhone, there's a `Shortcuts.sqlite`{: .filepath} database in `/private/var/mobile/Library/Shortcuts`{: .filepath}. In the `ZSHORTCUTACTIONS`{: .filepath} table there are some interesting code snippets in the `ZDATA`{: .filepath} column. The first entry contains logic for encoding data, and the second for decoding data.

![ZDATA Decode Logic](/assets/img/2024-04-07/17_5.png){: width="600" }

I pull out the decode logic into Notepad. It seems we would need a key, but really, we only need two integers a and b. The modulo operators indicate that a would initially be less than 137, and b would be less than or equal to 89.

![Decoding Logic](/assets/img/2024-04-07/17_6.png){: width="500" }

Those numbers are easy to brute force. I nest the decoding logic into two for loops, and add a regex check to see if the decoded text contains printable ASCII characters. The full script is below.

```javascript
var hexEncodedText = "addaf04dc94ddaf04ddfc980661fc91e7caac9f0dadac94df1664d0c";

for(let a_mod = 0; a_mod < 137; a_mod++) {
	for(let b = 1; b <= 89; b++) {
		a = 2*a_mod + 1;

		let decodedText = '';

		let modInverseA = 0;

		for (let i = 0; i < 256; i++) {
			if ((a*i)%256 === 1) {
				modInverseA = i;
				break;
			}
		}

		let encodedText = '';

		for (let i = 0; i < hexEncodedText.length; i+= 2) {
			let charCode = parseInt(hexEncodedText.substr(i, 2), 16);
			encodedText += String.fromCharCode(charCode);
		}

		for (let i = 0; i < encodedText.length; i++) {
			var tt = encodedText.charCodeAt(i) - b;
			if(tt < 0) {
				tt += 256;
			}
			let charCode = (modInverseA * tt) %256;
			decodedText += String.fromCharCode(charCode);
		}
		if(/^[\x20-\x7F]*$/.test(decodedText) == true) {
			console.log(decodedText);
		}
	}
}
```

I compile and run it, and I get my first printable ASCII strings, one of which looks like something a human might come up with.

![Decoded Hex](/assets/img/2024-04-07/17_7.png){: width="400" }

Yes I can see that, William. I start decoding the rest of the hex messages from the `t7`{: .filepath} table. One of them contains the SWIFT code for the offshore bank: **CRVBPA2P**.

![Decoded SWIFT Code](/assets/img/2024-04-07/17_8.png){: width="400" }

### 18. Statement (Hard - 590 points)

**Prompt**: Paste Phorger's entire bank statement here, containing all his offshore transactions.

Back at <https://crbk.org>, there seems to be an option to download all of William's transactions. Unfortunately, this requires a one-time password.

![CRBK 2FA](/assets/img/2024-04-07/18_1.png){: width="600" }

On the iPhone, I came across a screenshot of the Google Authenticator app (`/private/var/mobile/Media/DCIM/100APPLE/IMG_0032.png`{: .filepath}), which William seemingly used to authenticate to CRBK.

![IMG_0032](/assets/img/2024-04-07/18_2.png){: width="500" }

To do the same, one would need a QR code or setup key for CRBK. On the Windows image, there's an iPhone backup in `C:\Users\phorger\AppData\Roaming\Apple Computer\MobileSync\Backup\46803e574938af07acaf4ff465b25360e9680067`{: .filepath}. I'm learning that Apple is never going to make it easy for me, because the backup is spread over 250+ folders. I didn't have the time or patience to look through them all during the CTF, but the QR code is at `15\15eb410a07be50b098db1d43758d7d71a91d757d`{: .filepath}.

![QR Code](/assets/img/2024-04-07/18_3.png){: width="500" }

I download Google Authenticator on my phone and scan the QR Code. I then use the one-time password to download all the transactions from CRBK into a CSV file.

![Google Authenticator](/assets/img/2024-04-07/18_4.PNG){: width="300" }

![CRBK 2FA](/assets/img/2024-04-07/18_5.png){: width="600" }

I open the CSV in Notepad and paste the statement into the CTF site.


### Debrief

My first live CTF!!! What a thrill! I worked 30 of the 48 hours the CTF was live, and by the end my eyes were basically bleeding. I placed 15th, and am a little miffed at myself because I had two answers correct, but I didn't format them correctly. I think I would've needed at least a couple more days to solve [task 17](#17-financial-institution-hard---829-points), and if I had spent more time going through the iPhone backup, I definitely could've gotten [task 18](#18-statement-hard---590-points). So all in all, I don't feel too bad about where I'm at skill-wise. I really appreciated this timely opportunity to evaluate my skill level and see how much I've grown over the past couple of years, and I'm so excited to start my new job and put all this knowledge to use!
