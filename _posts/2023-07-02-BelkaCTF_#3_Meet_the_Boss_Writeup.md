---
title: "BelkaCTF #3: Meet the Boss Writeup"
description: Writeup for Belkasoft's Meet the Boss CTF.
author: sosborn
date: 2023-07-02 11:27:00 -0400
categories: [CTF]
tags: [belkasoft, linux]
---

**Link**: <https://belkasoft.com/ctf_june/>

**Problem**: We have the computer of a drug lord to comb through. We need to find evidence to lock him up for good this time.

### 1. Username (Baby - 166 points) - What is the username on the imaged laptop?

Linux again? Debian this time. It's still the same process as the last writeup. I load the image into Autopsy and navigate to the `/home`{: .filepath} directory. Just one user this time: **vt**.

### 2. Forums (Warmup - 213 points) - Which specific forums did the Boss have accounts on?

Autopsy loads any web searches it can find in Data Artifacts -> Web History. Looking at the entries, it seems Boss might have moved to the Brave browser, with searches such as "privacy browser" and visits to <https://brave.com>. Maybe there'll be some more history to look through if I go looking for files associated with Brave.

I navigate to `/home/vt/.config`{: .filepath} and find a folder called `BraveSoftware`{: .filepath}. I poke around a bit until I end up in `/home/vt/.config/BraveSoftware/Brave-Browser/Default`{: .filepath}. Here there's a file called `Login Data`{: .filepath} which seems to be a database. In this file there's a table called logins. Here we see sites Boss has logged into. Two forums are listed, associated with domains **grasscity.com** and **thctalk.com**.

![Brave Logins](/assets/img/2023-07-02/2_1.png)

### 3. Delivery (Warmup - 242 points) - What was the location where the boss had ordered a delivery to on October 19, 2020?

I look around a little more in the `Default`{: .filepath} folder and come across a `Web Data`{: .filepath} file, which also seems to be a database. There's a table called autofill, and when I view the records, I see an entry for an address: **9111 W McKinley St, Tolleson, AZ 85353**. There's a timestamp for the last time it was used.

![Brave Web Data](/assets/img/2023-07-02/3_1.png)

I plug it into <https://www.epochconverter.com/> and confirm the address was last used on October 19, 2020.

### 4. Product (Hard - 457 points) - What equipment has the boss ordered?

This one sucked. I looked all over the `Brave-Browser/Default`{: .filepath} folder, checking every subdirectory and every file in every subdirectory. The only thing I found even a little related was a folder called `databases`{: .filepath} that contained three folders: `https_singin.ebay.com_0`{: .filepath}, `https_signup.ebay.com_0`{: .filepath}, and `https_www.ebay.com_0`{: .filepath}, with timestamps corresponding to October 19, 2020.

![Ebay Folders](/assets/img/2023-07-02/4_1.png)

No links pointing to any items or anything, though. New lesson: when all else fails, use grep! I ended up in `/home/vt/.cache/BraveSoftware/Brave-Browser/Default/Cache`{: .filepath}, which contained over 11,000 files that I did not have the patience or sanity to manually look through. So I exported the folder to my local machine, and ran `grep ebay.com/itm *`. And we get some matches!

![Grep Results](/assets/img/2023-07-02/4_2.png){: width="500"}

Catting each of these gives the following links:

- <https://www.ebay.com/itm/273997170926?hash=item3fcb8108ee:g:OaAAAOSwee5dcUNS>
- <https://www.ebay.com/itm/283602462421?hash=item42080626d5:g:oNkAAOSw5nZdZXlN>
- <https://www.ebay.com/itm/313460105738?hash=item48fbada20a:g:9o4AAOSwcMNgU6NA>

The first and third links point to the same page: a listing for **Pyrex Container/Tube w/ Flat Bottom Chemistry Lab Glassware**.

### 5. Thumb Drive (Tricky - 471 points) - What thumb drive was the Boss actively using?

I did some light google searching and found that one might be able to see USB devices in `/var/log`{: .filepath}. So I start looking through the files in Autopsy. I don't have to look for very long this time; in `kern.log4.gz`{: .filepath} there's a file called `kern.log.4`{: .filepath}. The very first few lines show a USB Mass Storage device was connected to the system.

![USB Log](/assets/img/2023-07-02/5_1.png){: width="600"}

Looking up the Vendor ID and Product ID gives a make of **Kingston** and a model of **HyperX Savage**.

![Device Hunt Results](/assets/img/2023-07-02/5_2.png){: width="400"}

Googling the device gives the color: **Red** and **Black**.

![Drive Color](/assets/img/2023-07-02/5_3.png){: width="500"}

### 6. CRM (Baby - 397 points) - What is the URL of the Syndicate's CRM system?

Back to Brave Browser's `Default`{: .filepath} folder! I come across a file called `Bookmarks.bak`{: .filepath} that has some interesting, drug-related bookmarks in it. I recall a file named `dough.dat`{: .filepath} in discovery, so of the bookmarks, I figure the most likely for a CRM would be Dope Dough at **http://dopedoughyignlbq.onion**.

![Dope Dough Bookmark](/assets/img/2023-07-02/6_1.png){: width="500"}

### 7. Password (Tricky - 800 points) - What is the password to the syndicate's CRM system?

In the autofill table from an earlier task I remember seeing a field for passwords, but they're not available in plaintext. So began a new learning objective: to mount an image as a VM to see if I can access those passwords.

I loaded the image into FTK Imager and exported it as a raw (dd) image. From there, I used VirtualBox to convert the raw image to VDI - a format VirtualBox would understand (`VBoxManage convertdd image.001 image.vdi --format VDI`).

![BBoxManage](/assets/img/2023-07-02/7_1.png){: width="700"}

Autopsy reveals the distro is Ubuntu. I create a new VM with the image as VDI. I enter into boot mode and change the password for vt by accessing root from recovery mode.

![Recovery Mode](/assets/img/2023-07-02/7_2.png){: width="600"}

I allow the machine to finish booting up. From here, I have an option to login as vt, which I do with the password I just set.

![Login](/assets/img/2023-07-02/7_3.png){: width="400"}

After logging in successfully, I open the Brave Web Browser.

![Brave](/assets/img/2023-07-02/7_4.png){: width="500"}

I click the hamburger icon in the top right, and click Settings. I scroll down until I hit the Autofill section.

![Brave Autofill](/assets/img/2023-07-02/7_5.png){: width="500"}

I click on Passwords, which expands to show associated passwords for a few sites. Clicking the eye next to each of them reveals the plaintext.

![Brave Site Passwords](/assets/img/2023-07-02/7_6.png){: width="500"}

Hm, maybe one of these will open that KeePass file on the Desktop I saw (`words.kdbx`{: .filepath}). Sure enough, the very first one I try opens it up to reveal more passwords, and one for dopedough.

![dopedought Password](/assets/img/2023-07-02/7_7.png){: width="500"}

Examining the entry gives me the password: **SdDlBtUE6fk6zAxb**.

![Plaintext Password](/assets/img/2023-07-02/7_8.png){: width="500"}

### 8. Big Client (Hard - 800 points) - What is the name of the night club the Syndicate was providing drugs to in bulk?

I can't log in to the CRM, because the link is no longer functional. But I can start poking around in some of his accounts. His email seems like a good place to start. I log in to Proton Mail with his credentials (which I also pull from `words.kdbx`{: .filepath}) and run a search for crm.

![Proton Mail crm Search](/assets/img/2023-07-02/8_1.png){: width="500"}

Lucky me! I open the most recent and pull out the attachment: a file called `backup.zip`{: .filepath}. From it, I extract a file called `backup.csv`{: .filepath}. Opening this file in LibreOffice Calc (yuck), I see it contains sales records with coordinates.

![backup.csv](/assets/img/2023-07-02/8_2.png){: width="600"}

The task is looking for a night club in a neighbor state. We know the boss is operating out of Arizona, which is bordered by Nevada, Utah, and New Mexico. For a nightclub, my guess would be Las Vegas, Nevada, so we're looking for latitude 36 and longitude -115.

I write a simple script isolating out records with latitude 36 and longitude -115, rounding the coordinates to 3 decimal places, and writing to a new file.

```python
with open("backup.csv", "r") as file:
	with open("backup_2.csv", "w") as file2:
		for line in file:
			if("Location" in line):
				continue
			if("36." in line and "-115." in line):
				arr = line.split(',')
				arr[3] = "\""+str(round(float(arr[3][1:]),3))
				arr[4] = str(round(float(arr[4][:len(arr[4])-1]),3))+"\""
				file2.write(','.join(arr))
```

I open the new file in LibreOffice Calc and use the formula `INDEX(range,MODE(MATCH(range,range,0)))` to find the most frequent string of coordinates. This returns 36.129, -115.188.

![Script Results](/assets/img/2023-07-02/8_3.png){: width="600"}

Plugging these into Google Maps shows the location as **Embassy Nightclub**.

![Embassy Nightclub](/assets/img/2023-07-02/8_4.png){: width="500"}

### 9. Hosting (Tricky - 709 points) - What is the IP address their CRM is hosted at?

Since the site isn't functional anymore, maybe I could look at the email and see if I can pull the IP out of it.

Looking at the email in Proton Mail doesn't help me out much, so I export it to the Desktop for some Linux magic.

Running the head command gives the static IP for the CRM server: **49.12.108.254**.

![Email Header](/assets/img/2023-07-02/9_1.png){: width="700"}

### 10. Tranche (Tricky - 643 points) - What are the cryptocurrency wallet addresses the Boss had transferred salary to in the last batch? (4 of them)

There's a file of interest in `/home/vt/Documents/copy/dough_apr`{: .filepath} called `wallet.dat`{: .filepath}. I export the file to my local computer. Loading the file into the Bitcoin Core app shows the transactions; the ones with labels give the addresses we're looking for.

![Bitcoin Addresses](/assets/img/2023-07-02/10_1.png){: width="600"}

The addresses associated with those transactionsa re:

- **1B4uj2cMcVncGSsH7myYQF571JaojxCy4T**
- **1HcZrD8gqDEtYhgmKJ2ECJAwA2stioknme**
- **1FrxcR5Fj593ZbgYiiApVVEUV6PjvYfnqn**
- **1NJXwLN5uC1PwWQ7yfvJiQTG76UUXTkrxr**

### 11. Retrospective (Hard - 1000 points) - What cryptocurrency wallets did the Boss transfer salary to in February? (4 addresses)

If we cat the `wallet.dat`{: .filepath} file, we can see that there's repeated strings "fromaccount" and "timesmart" followed by what looks like a timestamp.

![Wallet.dat](/assets/img/2023-07-02/11_1.png){: width="600"}

There are no transactions from February in `wallet.dat`{: .filepath}, but maybe there's a trace of another wallet on the system. Autopsy lets us do a keyword search. Searching for "fromaccount" gives us a few results, a couple of which are from unallocated space.

![Keyword Search Results](/assets/img/2023-07-02/11_2.png){: width="700"}

Now we're just looking for a timestamp for February 2021. There's a few February timestamps in the second unallocated file.

![Second File](/assets/img/2023-07-02/11_3.png){: width="700"}

Going back in the unallocated space a bit, we're able to find some wallets. Pulling out the ones with labels related to February gives the four wallets we seek:

- **1GChpNsQ5x5yynwkuyXWq3Ea5JKYgdPKir**
- **1CwWyNGGCjzmkuQnQRRA7H6zsUjt2Y9d8g**
- **1PGLY1kFGi1RcZ4JAKY4v1VdcHJJwhY1Hs**
- **19yJnHa5smr63fdDBpwrJUBCkud8J6EbSn**

![Wallets](/assets/img/2023-07-02/11_4.png){: width="700"}

### Debrief

I hated this one! It was extremely frustrating; I had to consult the writeup on pretty much every single question. What a blow to the self-confidence I'd built up over the past two CTFs. Positives to come out of this awful, awful CTF are 1) I now know that there's so much more I need to learn, and 2) I was able to learn a lot from these tasks. I'm hoping for a bit of an easier time on the next one.

