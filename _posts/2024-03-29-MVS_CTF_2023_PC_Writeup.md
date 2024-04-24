---
title: Magnet Virtual Summit CTF 2023 - PC Writeup
description: Writeup for the PC portion of the MVS 2023 CTF.
author: sosborn
date: 2024-03-30 03:00:00 -0400
categories: [CTF]
tags: [magnet, windows]

---

### Info

**Link**: <https://www.magnetforensics.com/blog/announcing-the-mvs-2023-ctf-winners-and-a-new-ctf-challenge/>

**Problem**: There's an Windows physical image to analyze and a series of questions to answer.

### 1. Gmail? Outlook? Yeah, right..

**Prompt**: What non-standard email service has the user used previously?

After running Autopsy's ingest modules on the E01, I noticed in `Data Artifacts -> Web Form Autofill`{: .filepath} a Chrome autofill entry with value MichaelKBorchardt@protonmail.com, indicating the user had a **Protonmail** account. 

![Protonmail Autofill](/assets/img/2024-03-29/1_1.png){: width="400" }

**Answer**: Protonmail

### 2. Two different versions, twice the emulation power! Makes sense to me!

**Prompt**: The user installed and ran a mobile device emulation program on their system. Which 2 versions of this software did the user install? (Format: SoftwareName V1/V2)

Navigating in Autopsy to `Data Artifacts -> Installed Programs`{: .filepath} gives an entry for BlueStacks 5 from the `SOFTWARE`{: .filepath} registry hive, which a quick Google verifies to be a mobile emulation program.

![BlueStacks 5](/assets/img/2024-03-29/2_1.png){: width="500" } 

But in `C:\Program Files (x86)`{: .filepath} there's a folder `BlueStacks X`{: .filepath}. So the answer is **BlueStacks 5/X**. 

**Answer**: BlueStacks 5/X

### 3. LITEning fast write speeds!

**Prompt**: The user’s system is equipped with a 256GB NVMe SSD. What is the make and model of this drive?

I start inspecting the `SYSTEM`{: .filepath} hive (`C:\Windows\System32\config\`{: .filepath}) in Autopsy and look at the usual suspects for device information. I come to the `ControlSet001\Enum\SCSI`{: .filepath} key and see a subkey `Disk&Ven_NVMe&Prod_LITEON_CA1-8D256`{: .filepath}. Hmm...NVMe, LITEON, this could be the drive we're looking for. Clicking on the subkey displays Autopsy's aggregated information on the drive, including the `FriendlyName`{: .filepath}: **LITEON CA1-8D256-HP**.

![Enum/SCSI key](/assets/img/2024-03-29/3_1.png){: width="500" }

**Answer**: LITEON CA1-8D256-HP

### 4. Really...? Plaintext...?

**Prompt**: The user frequently accesses a Chrome Remote Desktop virtual machine. What password is used to log into this VM?

There's a file on the user's Desktop called `Employee Logins.txt`{: .filepath} which contains the password to the Google VM: **,a]JEU0yG^+]2O]**.

![VM Password](/assets/img/2024-03-29/4_1.png){: width="300"}

**Answer**: ,a]JEU0yG^+]2O]

### 5. Why was 6 afraid of 7? Because 7 can unarchive virtual drives!

**Prompt**: Within the past 2 years, a popular unarchiving program gained the ability to unarchive VHDX virtual disk images. What version of the program was this upgrade implemented?

In `C:\Program Files\`{: .filepath} there's a folder `7-Zip`{: .filepath} corresponding to the 7-Zip archiving application. In this folder there's a file `History.txt`{: .filepath} that contains version history for 7-Zip. Extracting and running a grep (`grep -B 3 VHDX History.txt`) reveals this feature was implemented in version **21.07**.

![7z History](/assets/img/2024-03-29/5_1.png){: width="500"}

**Answer**: 21.07

### 6. We’re not in Kansas anymore...

**Prompt**: The user has established an RDP connection to one destination more than any other. What is the Geolocation of this destination? (Format: City, ST)

I extract `C:\Windows\System32\winevt\Logs\Microsoft-Windows-TerminalServices-RDPClient%4Operational.evtx`{: .filepath} which contains Windows RDP logs, and open it in Event Viewer. The events indicate the user tried to connect to two different IPs: 34.162.141.21, and more prevalently, 34.162.97.100.

![Event Viewer](/assets/img/2024-03-29/6_1.png){: width="500"}

Looking up the IP on <https://www.maxmind.com/en/geoip-demo> shows the IP originates in **Columbus, OH**.

![Maxmind](/assets/img/2024-03-29/6_2.png){: width="600"}

**Answer**: Columbus, OH

### 7. Make sure to keep some tabs on that SysAdmin from Southern California

**Prompt**: The user visited the Mastodon page of one user more than any others on the platform. What is the full legal name of the user Michael visited?

In Autopsy, I look through `Data Artifacts -> Web History`{: .filepath} and notice the only profile visits are to <https://mastodon.social/@scriptingosx@mastodon.social>.

![Mastodon History](/assets/img/2024-03-29/7_1.png)

Following the URL to the profile, there's a link to the user's LinkedIn, and there we get the full name: **Armin Briegel**.

![Mastodon Profile](/assets/img/2024-03-29/7_2.png){: width="500"}

![LinkedIn](/assets/img/2024-03-29/7_3.png){: width="500"}

**Answer**: Armin Briegel

### 8. We have a History of attracting some sizeable donors with our projects

**Prompt**: Michael used PowerShell to clone a particular GitHub utility. What is the account name of one of this repo’s most prominent sponsors?

PowerShell history can be found in `C:\Users\borch\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`{: .filepath}. The first commands are cloning the repository at <https://github.com/LSPosed/MagiskOnWSALocal.git>.

![PowerShell History](/assets/img/2024-03-29/8_1.png){: width="500"}

Navigating to the repo, there's one entry for Sponsors: **yujincheng08**.

![Github Repo](/assets/img/2024-03-29/8_2.png){: width="500"}

**Answer**: yujincheng08

### 9. Scratch that Itch.io

**Prompt**: The user viewed a YouTube video by the creator BenBonk surrounding video game developers. Within this video, how many developers were involved with the project?

Looking back at Autopsy's `Data Artifacts -> Web History`{: .filepath}, there are three entries for YouTube videos with titles about game developers.

![YouTube URLs](/assets/img/2024-03-29/9_1.png)

Of those videos, only the third one was created by BenBonk. Per the title, **20** developers were involved.

**Answer**: 20

### 10. The breakfast bell is ringing

**Prompt**: The user has been doing some research lately on fast food items. What is, according to some experts, the unhealthiest food item of the bunch?

Still in Autopsy's `Data Artifacts -> Web History`{: .filepath}, there's a YouTube video entitled 'Ranking the "Healthiest" Taco Bell Items.'

![Taco Bell Video](/assets/img/2024-03-29/10_1.png)

The unhealthiest item they ranked in the video was the **Breakfast Crunchwrap Sausage Supreme**.

**Answer**: Breakfast Crunchwrap Sausage Supreme

### 11. Gotta Git going fast with some Accelrated emulation!

**Prompt**: In order to emulate an Android device, the user required some specialized management tools. What Android port is used by default with these services?

In Autopsy, under `Data Artifacts -> Installed Programs`{: .filepath} there's an entry for WSA PacMan, which is a package manager for Windows Subsystem for Android.

![WSA PacMan](/assets/img/2024-03-29/11_1.png){: width="600"}

I followed the first link from my preliminary Google search to the Github repository (<https://github.com/alesimula/wsa_pacman>), and the README listed the default port as **58526**.

![WSA PacMan README](/assets/img/2024-03-29/11_2.png){: width="400"}

**Answer**: 58526

### 12. Oh Deer...I think we’re lost

**Prompt**: Michael lives just a mile south of a beautiful body of water. What is the name of this body of water

One of the first autofill entries in `Data Artifacts -> Web Form Autofill`{: .filepath} gives the first line of an address, from the Chrome Default profile.

![Address Autofill](/assets/img/2024-03-29/12_1.png){: width="700"}

I plug this address into Google Maps and see it's just below a river called **Deer Creek**.

![Google Maps Address](/assets/img/2024-03-29/12_2.png)

**Answer**: Deer Creek

### 13. PCA – Program Clang Assistant?

**Prompt**: The user has installed Android Studio with a specialized plugin dedicating to diagnosing and fixing some programming errors. When this plugin runs, what exit code is used upon completion?

I struggled a lot with this one, and ultimately had to consult [Kevin Pagano's writeup](https://www.stark4n6.com/2023/03/magnet-virtual-summit-2023-ctf-windows.html), but on the upside, I learned about a new forensic artifact I'm positive I wouldn't have found on my own: the Program Compatibility Assistant described in [this blog](https://aboutdfir.com/new-windows-11-pro-22h2-evidence-of-execution-artifact/) by Andrew Rathbun and Lucas Gonzalez. Within `C:\Windows\appcompat\pca`{: .filepath} are three files that provide evidence for application execution: `PcaAppLaunchDic.txt`{: .filepath}, `PcaGeneralDb0.txt`{: .filepath}, and `PcaGeneralDb1.txt`{: .filepath}. The first provides file paths and timestamps, and the second and third provide more detailed information such as runtime, run status, execution path, software vendor... and the exit code value we're looking for.

There are many executions listed in `PcaGeneralDb0.txt`{: .filepath}, but significantly, there's evidence of an Android Studio plugin named `clang-tidy.exe`{: .filepath}, which turns out to be a C++ linter.

![PcaGeneralDb0.txt](/assets/img/2024-03-29/13_1.png)

The program continually exits with code **0xc0000135**.

**Answer**: 0xc0000135
### Debrief

This CTF was interesting, to say the least. I pretty much breezed through the first 12 questions, and my confidence soared to new heights. And then 13 hit me like a truck. I think this was the best outcome, though: I feel assured in that I'm most comfortable with Windows, but I also am glad to have learned something new along the way.
