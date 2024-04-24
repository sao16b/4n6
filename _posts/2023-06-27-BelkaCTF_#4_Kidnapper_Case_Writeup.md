---
title: "BelkaCTF #4: Kidnapper Case Writeup"
description: Writeup for Belkasoft's Kidnapper Case CTF.
author: sosborn
date: 2023-06-27 20:28:00 -0400
categories: [CTF]
tags: [belkasoft, linux]
---

### Info

**Link**: <https://belkasoft.com/ctf_march_2022/>

**Problem**: A boy has gone missing. Let's find out what happened to him! A laptop loaded with Linux? Oh boy.

### 1. Users

**Prompt**: List all users of the laptop.

Easy peasy. On Linux, users are allocated a directory in `/home`{: .filepath}. I opened the E01 in Autopsy and navigated to that directory. There are two users: **ivan** and **stanley**.

![User Folders](/assets/img/2023-06-27_2/1_1.png){: width="700" }

**Answer**: ivan, stanley

### 2. Special Web Site

**Prompt**: What web application was used by this boy to earn his pocket money?

From the timestamps it looks like the ivan user was made after stanley, so that is likely the alternate account. Autopsy lays out the web history nice for me, but I want to learn, so I find out that Firefox web history is located in `~/.mozilla`{: .filepath} in a database called `places.sqlite`{: .filepath}. Also learned that Autopsy will display the database for me; no need for DBBrowser!

I found the `places.sqlite`{: .filepath} database in `/home/ivan/.mozilla/firefox/yyxsdegu.default-release`{: .filepath}. One entry catches my eye: **x-tux-0.web.app**. Navigating to it shows it's a "drug" store.

![Drug Store](/assets/img/2023-06-27_2/2_1.png){: width="400" }

**Answer**: x-tux-0.web.app

### 3. Wallet

**Prompt**: Which BTC wallet did the boy use to sell drugs?

Clicking on one of the 'View Product' buttons on the site gives us a bitcoin wallet to send money to: **1KFHE7w8BhaENAswwryaoccDb6qcT6DbYY**.

![Bitcoin Address](/assets/img/2023-06-27_2/3_1.png){: width="600" }

**Answer**: 1KFHE7w8BhaENAswwryaoccDb6qcT6DbYY

### 4. Passme

**Prompt**: On which date does the kid's database show the most sales for "Acapulco Gold"?

The task points us to email. Stanley has a `.thunderbird`{: .filepath} folder in his home directory. Autopsy formats E-mail messages nicely so I go to `Data Artifacts -> E-Mail Messages -> Default -> Default`{: .filepath} to see Stanley's emails. The latest e-mail was sent from ivstanely@yandex.com to tuxnetwork@yandex.com with the database attached.

![E-Mail Messages](/assets/img/2023-06-27_2/4_1.png)

I export the file to my local computer. Looking through the other e-mails, tuxnetwork@yandex.com instructs Stanley to encrypt the database using a password from a password list, which I export as well.

However, it looks like the password list, despite it's name (`10-million-password-list-top-1000000.txt`{: .filepath}) only contains 909540 lines. So I do a quick search for the file online and download the full file from [Github](https://raw.githubusercontent.com/danielmiessler/SecLists/master/Passwords/Common-Credentials/10-million-password-list-top-1000000.txt).

New tool! Zip2John can extract password hashes from a zip file, on which we can run a password cracker like JohnTheRipper. Which is what I did, following a [tutorial](https://infinitelogins.com/2020/04/29/how-to-crack-encrypted-7z-archives/). First I ran zip2john on `Monthly_DB.zip`{: .filepath} to get password hashes of all the files in the ZIP (`john/run/zip2john Monthly_DB.zip >> zip.hash`).

![Zip2John](/assets/img/2023-06-27_2/4_2.png)

Then, I ran john on the hashes using the wordlist from the Github (`john/run/john --wordlist=/path/to/wordlist zip.hash`).

![John](/assets/img/2023-06-27_2/4_3.png)

Looks like the password is vondutcemonaheem_gangsta78. I decrypted the `Monthly_DB.zip`{: .filepath} file with that password to get a bunch of CSV files organized into `2020`{: .filepath} and `2021`{: .filepath} folders.

I ran the following command to get only sales for Acapulco Gold from both folders: `cat 2020/* | grep 'Acapulco Gold'; cat 2021/* | grep 'Acapulco Gold'`.

![Acapulco Gold Sales](/assets/img/2023-06-27_2/4_4.png)

A quick look through the results showed Stanley/Ivan sold $16,044 in Acapulco Gold on **5/12/2021**.

**Answer**: 5/12/2021

### 5. Cryptlet

**Prompt**: What was the other BTC wallet of the victim, which he used to hide his "under the counter" sales from his superior?

Whew! This one was a doozy, but what a rush when I figured it out!

Snooping around in Ivan's `Documents`{: .filepath} folder gives a file `.custom.info`{: .filepath} in the `.custom`{: .filepath} folder with almost his private Bitcoin wallet: bc1q__2kgdygjrs__zq2n0yrf2493p__kkfjhx__lh.

![Partial Bitcoin Address](/assets/img/2023-06-27_2/5_1.png)

The other files in the folder look like PDF invoices, but no sign of a bitcoin wallet. There's one file called `101.bin`{: .filepath} that's a little off, though. I examined the hexadecimal, and heard Brain Troll once more, barking at me, "File signatures, file signatures." I recall from my digital forensics course some PDF files starting with %PDF-1.4.1, and ending with %%EOF or something similar. This bin file is close, but it's all...jangled. I see 1-4.1 in the header.

![101.bin Hexdump Header](/assets/img/2023-06-27_2/5_2.png){: width="600"}

I see %%OE.F in the footer.

![101.bin Hexdump Footer](/assets/img/2023-06-27_2/5_3.png){: width="600"}

I compare against another PDF in the folder. There are a lot of similar components, but in this `101.bin`{: .filepath} file, they all seem mixed up. I stared gloomily at hexadecimal for about a half hour before a pattern emerged...swapping every two bytes gave the standard PDF format. This was an endianness problem! I rushed to Google to find a simple solution to swap the endianness of a file (`dd conv=swab < 101.bin > 101_pdf`). Oh dd, you versatile, beautiful beast.

![dd Results](/assets/img/2023-06-27_2/5_4.png){: width="600"}

With the endianness swapped, everything looked cleaner. I had -1.4.1 in the header.

![Post-dd 101.bin Hexdump Header](/assets/img/2023-06-27_2/5_5.png){: width="600"}

I had %%EOF. in the footer.

![Post-dd 101.bin Hexdump Footer](/assets/img/2023-06-27_2/5_6.png){: width="600"}

All that's left is to fill in the header.

![101.bin Fixed Hexdump Header](/assets/img/2023-06-27_2/5_7.png){: width="600"}

Saving this new file and opening it gives us an invoice listing the private bitcoin address: **bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh**.

![Fixed PDF](/assets/img/2023-06-27_2/5_8.png){: width="500"}

**Answer**: bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh

### 6. Notipass

**Prompt**: What is the password to the boy's notes?

In `/home/stanley/Documents/.mynote`{: .filepath} I found a CDFV2 Encrypted file called `notes`{: .filepath}. I export this to my local machine; now to find the password.

The task points us in the direction of containers. I thought I saw a `vmware`{: .filepath} folder somewhere...yes! `/home/stanley/.cache/vmware`{: .filepath}. Dishearteningly, I find folders with PDFs that give the answer to [task 5](#5-cryptlet) with just a simple header substitution. But there's something else! A PDF called `NOTHING_IMPORTANT_INFO.pdf`{: .filepath} with something seemingly important embedded in it (thank you Autopsy!): a file called `passwd`{: .filepath}.

![Important PDF](/assets/img/2023-06-27_2/6_1.png)

Exporting it and examining shows a long string of text.

![passwd](/assets/img/2023-06-27_2/6_2.png){: width="500"}

Troll is at it again with, "Base...something." A moment of weakness with the official read me confirms it's Base32. I plug it into a Base32 decoder and out spits more gibberish.

![Base32 Decode](/assets/img/2023-06-27_2/6_3.png){: width="500"}

But this! This is something I recognize. It looks like there may be a substitution/shift cipher in play. Good old <https://www.dcode.fr> shows a rotation of 13 for Snprobbx to get Facebook. Sticking all of it into <https://www.dcode.fr> gives us a decrypted password list, including the password for `notes`{: .filepath}: **!mp0rt4nTNot3**.

![dcode.fr Results](/assets/img/2023-06-27_2/6_4.png){: width="500"}

**Answer**: !mp0rt4nTNot3

### 7. Specudio

**Prompt**: What is the "secret pin" mentioned in the notes?

I decrypted the `notes`{: .filepath} file with the command `msoffcrypto-tool notes notes_d -p '!mp0rt4nTNot3'` and opened the decrypted file as a Word document. At the very end of the file was a paragraph saying he had stashed the secret pin in a shark file. Shark...Wireshark! I remembered seeing a pcap file somewhere, in `/home/ivan/Music/.secs/.secret.pcapng`{: .filepath}.

I open the PCAP in Wireshark and scroll through the packets until I see interesting HTTP traffic: a GET request for a `vault_secret_code.wav`{: .filepath} file.

![Wireshark](/assets/img/2023-06-27_2/7_1.png)

I export the `vault_secret_code.wav`{: .filepath} file from Wireshark and give it a listen. Nothing discernible...sounds like robot noises.

I open the file in Audacity, but I'm not too familiar with audio analysis so I'm just pushing buttons for a few minutes. New information: you can change how you visualize the audio in Audacity! Changing from waveform to spectrogram gives a pretty picture with the secret pin: **1257**.

![Audacity Spectrogram](/assets/img/2023-06-27_2/7_2.png){: width="500"}

**Answer**: 1257

### 8. Ultimatum

**Prompt**: When did the boy receive a threat?

Looking back at the `notes`{: .filepath} document, there's a timestamp associated with the final paragraph where Ivan/Stanley describes being threatened 10 minutes earlier. Subtracting 10\*60=600 from 1637948867 gives **1637948267**.

**Answer**: 1637948267

### 9. Whois

**Prompt**: Who was the kidnapper?

From the `notes`{: .filepath} file we know it's his friend **0xTux**. His email can be found back in Autopsy in `Data Artifacts -> E-Mail Messages -> Default -> Default`{: .filepath} (tuxnetwork@yandex.com). Or, at least, so I thought.

But, turns out there's a different e-mail address for Tux. If I stopped to think that we hadn't used the secret code from the previous task for anything, maybe I would have come to this conclusion. But I'm tired and I haven't had dinner so I give myself a pass.

I recall one more password-protected zip file I had come across that I hadn't investigated yet: `mycon.zip`{: .filepath} found in `/home/ivan/.local`{: .filepath}. Extracting the zip and plugging in the pin from the previous task gives another zip file called `Connections.zip`{: .filepath}. Unzipping that gives a folder called `resources`{: .filepath} and an HTML file called `Sheet1.html`{: .filepath}. Opening this with Edge gives a table of names and addresses.

![Sheet1.html](/assets/img/2023-06-27_2/8_1.png){: width="400"}

Searching for Tux gives an email: **wixelig493@keagenan.com**.

![Tux Email](/assets/img/2023-06-27_2/8_2.png){: width="400"}

**Answer**: wixelig493@keagenan.com

### Debrief

And that's this CTF done! I really like these challenges so far, and there's still three more to go! I think they're really well-rounded, considering all the skills I had to employ in this challenge and the one before. I learned a little about Audacity, got to brush up on Wireshark and file signature knowledge, and this was my first time doing forensics on a Linux image! I also learned I cannot be trusted to not look at the writeup whenever I struggle for more than 5 minutes on something! But I'm still learning, and I think that's ultimately what's important.

