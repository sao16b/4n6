---
title: "BelkaCTF #1: Insider Threat Writeup"
description: Writeup for Belkasoft's Insider Threat CTF.
author: sosborn
date: 2024-03-06 19:13:00 -0400
categories: [CTF]
tags: [belkasoft, windows]
---

### Info

**Link**: <https://belkasoft.com/ctf_march/>

**Problem**: A company's source code for an AI-based recommendation system has fallen into their competitor's hands, right before the launch. The prime suspect is a newly hired developer...and we have his hard drive.

### 1. Name (Baby - 100 points) - What is the full name of the laptop owner?

It's a Windows image, so expanding the file system out to `C:\Users`{: .filepath} gives the full name: **Anit Ghosh**.

![User Folders](/assets/img/2024-03-06/1_1.png){: width="300" }

### 2. Address (Baby - 208 points) - What is the full address of the company's office?

In Autopsy, I looked at Anit's inbox (Data Artifacts -> E-Mail Messages -> Default -> Default) and noticed many addresses ending in @praivacymatrix.com.

![Anit's Inbox](/assets/img/2024-03-06/2_1.png){: width="700" }

Navigating to praivacymatrix.com (note: as of 2024-03-29 this link is no longer functional) and surfing their site shows their address is **Ifangstrasse 6, 8952 Schlieren, Zurich, Switzerland**.

![praivacymatrix.com](/assets/img/2024-03-06/2_2.png){: width="500" }

### 3. First Shot (Warmup - 195 points) - When did the suspect first show interest in the company's trade secrets?

Looking through Anit's Sent folder, there's an email from Anit to John Finney (john.finney@praivacymatrix.com) asking for access to technical documentation. This email was sent at **2020-11-05 14:21:56 EST (19:21:56 UTC)**.

![Suspicious Email](/assets/img/2024-03-06/3_1.png){: width="300" }

### 4. Witness (Warmup - 150 points) - What 3 employees should be asked questions about unauthorized requests from the suspect?

We already have an email to **John Finney** asking for documentation. In Anit's inbox, there are also replies from **Noelle Johnson** (noelle.johnson@praivacymatrix.com) and **Rachel Corbin** (rachel.corbin@praivacymatrix.com) to Anit's requests for access.

![Rachael Corbin Email](/assets/img/2024-03-06/4_1.png){: width="500" }

![Noelle Johnson Email](/assets/img/2024-03-06/4_2.png){: width="500" }

### 5. Docs (Baby - 198 points) - What is the SHA256 hash of the product documentation obtained by the suspect?

Looking through Anit's `Documents`{: .filepath} folder, there's a file called `Doc_-_13_Feb_2021_-_13-40.pdf`{: .filepath} that contains technical details about a Project X, and has a big fat CONFIDENTIAL PM stamp on the front.

![Company Secrets](/assets/img/2024-03-06/5_1.png)

Autopsy also stores the hashes for each file, so I was able to grab the SHA256 hash from the File Metadata tab: **add33ea905399c5063bcc3437cb5c0436a2fd6deb086bb0ec5bf886f72767242**.

![File Metadata](/assets/img/2024-03-06/5_2.png){: width="500"}

### 6. Leaker (Tricky - 438 points) - What employee has actually provided the suspect with the product documentation?

Looking at the pdf, there are a few black boxes covering some content that could be of interest.

![Black Boxes](/assets/img/2024-03-06/6_1.png){: width="400"}

After doing some research, it looked as though Inkscape would be a good option for uncovering that content. Opening up the PDF in Inkscape and going to Objects -> Layers and Objects..., I'm able to move the boxes out of the way, giving the name of the employee who printed the document: **Mark Zukko**, employee ID **381**.

![Zukko Revealed](/assets/img/2024-03-06/6_2.png){: width="500"}

### 7. Source Code (Warmup - 160 points) - What URL did the suspect manage to obtain the product source code from?

In Anit's `Downloads`{: .filepath} folder, there's an interesting archive `xraicommend-761263a55b8cfed4bcb8f87cbbb68beaf2ec2423.tar.gz`{: .filepath} that seems to contain the product source code we're looking for. Luckily, on Windows computers, we can extract the URL from the corresponding Zone.Identifier alternate data stream. Examining this stream gives the URL as **http://git.pm.internal/GBringley/xraicommend/archive/761263a55b8cfed4bcb8f87cbbb68beaf2ec2423.tar.gz**.

![Alternate Data Stream](/assets/img/2024-03-06/7_1.png){: width="500"}

### 8. E-Mail (Hard - 750 points) - What e-mail address did the suspect's backdoor code send reports to?

In Anit's base directory, there's a folder called `adstresser`{: .filepath} that contains a git repository. Extracting this repository and running the `git branch -a` command shows a remote repository `remotes/origin/wip`{: .filepath} with no corresponding local repository, indicating this might have been the backdoor repo that has since been deleted locally.

![Git Branch Results](/assets/img/2024-03-06/8_1.png){: width="600"}

Running the command `git log remotes/orign/wip -3` shows the last three commits to this repository, the latest being from Anit.

![Recent Commits](/assets/img/2024-03-06/8_2.png){: width="600"}

Running the command `git show 08bca1dbc17adfc214f8d40c57673e0571914ac1` shows the changes of that latest commit, which includes an interesting base64 string.

![Git Show Results](/assets/img/2024-03-06/8_3.png){: width="600"}

Decoding this base64 string shows a bash command garbled by hex.

![Base64 Decoded](/assets/img/2024-03-06/8_4.png)

Passing the string through the echo command gives an intelligible script, which sends an email to address **alert872802737@protonmail.com**.

![Decoded Script](/assets/img/2024-03-06/8_5.png)

### 9. Recipient (Hard - 945 points) - What are the 2 phone numbers used in the leaking of the data: one of the suspect and one of their counterparty?

A cursory glance at the standard application folders doesn't yield anything for a chat-based app such as Whatsapp or Signal. But, doing a keyword search for Whatsapp gives a lead in the `hiberfil.sys`{: .filepath} file, indicating Anit might have used the Whatsapp web client for leaking data.

![Hiberfil.sys](/assets/img/2024-03-06/9_1.png){: width="700"}

In memory dumps, sometimes you can find phone numbers followed by @chat app domain. A quick regex later (`grep -Eoa '[0-9]{10,}+@\S+' /path/to/hiberfil.sys`), and we have a shortlist.

![Grep Results](/assets/img/2024-03-06/9_2.png)

Our first result indicates a Whatsapp account associated with the number **8562097771657**. We see that number a lot going down the list, but there's another one not too far down that's different: **8562099907377**@c.us.

### 10. Package (Tricky - 750 points) - What is the SHA256 hash of the file exfiltrated?

Doing a keyword search in Autopsy for 8562097771657@s.whatsapp.net shows a message with an interesting link in the `hiberfil.sys`{: .filepath} file: <https://anonfiles.com/z3jek3J2p3>.

![Keyword Search Results](/assets/img/2024-03-06/10_1.png){: width="500"}

Looking at the official writeup, we would have been able to access that link to download the file when the CTF was live, but the link is no longer functional. Wa wa.

### 11. Wallet (Hard - 674 points) - What is the suspect's cryptocurrency address they intended to get reward paid to?

Unfortunately it seems since the link in the previous task didn't work, and we don't have the `PHOTOS.7z`{: .filepath} file, that this task is unsolvable. Wa wa again.

### Debrief

The last of the Belkasoft CTF challenges ended unsatisfactorily, but I still had a lot of fun digging through the image and trying out tools, new and old. In this challenge, I learned a bit more about git and regular expressions, as well as how to manipulate PDFs. Completing all five Belkasoft CTFs has been frustrating and discouraging at times, but also extremely gratifying at others. I appreciate how in-depth these challenges are, how many tools and real-world scenarios they've exposed me to, and how much I've learned and improved by working through them.

