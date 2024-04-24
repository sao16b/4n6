---
title: Magnet Virtual Summit CTF 2023 - VMDK Writeup
description: Writeup for the VMDK portion of the MVS 2023 CTF.
author: sosborn
date: 2024-03-30 15:18:00 -0400
categories: [CTF]
tags: [magnet, windows, vm]

---

### Info

**Link**: <https://www.magnetforensics.com/blog/announcing-the-mvs-2023-ctf-winners-and-a-new-ctf-challenge/>

**Problem**: There's a VMDK physical image to analyze and a series of questions to answer.

I couldn't import the file into Autopsy, so I had to export it as a raw image from FTK Imager.

### 1. Maybe you can get the Team to help you Viewer this?

**Prompt**: What non-default remote desktop tool was installed on the system? Version number not required.

It seems most of these first questions can be answered through the registry files. In Autopsy, I navigate to `C:\Users\sgarza`{: .filepath}, and then to the `Software\Microsoft\Windows\CurrentVersion\Uninstall`{: .filepath} key in the `NTUSER.dat`{: .filepath} file. There's just one subkey, and under that subkey is the application name, which is **Chrome Remote Desktop**.

![Chrome Remote Desktop Registry](/assets/img/2024-03-30/1_1.png){: width="600" }

**Answer**: Chrome Remote Desktop

### 2. Remember to turn back the clocks in November!

**Prompt**: What is the system time zone?

System Time Zone information can be found in the `C:\Windows\System32\config\SYSTEM`{: .filepath} registry hive under key `ControlSet001\Control\TimeZoneInformation`{: .filepath}. The `TimeZoneKeyName`{: .filepath} subkey reveals the time zone as **Greenwich Standard Time**.

![Time Zone Registry](/assets/img/2024-03-30/2_1.png){: width="600" }

**Answer**: Greenwich Standard Time

### 3. I think I’m going to have a (National) Expresso toDay!

**Prompt**: What was the date and time on the system when the user account with a password hint was created? (YYYY-MM-DD HH:MM:SS)

This information can be found in the `C:\Windows\System32\config\SAM`{: .filepath} hive. Under `Domains\Account\Users`{: .filepath} there's a list of keys corresponding to the RIDs of the users of the system. One of them (`000003E8`{: .filepath}) has a `UserPasswordHint`{: .filepath} subkey. Selecting the `V`{: .filepath} subkey and scrolling through the value, one can see the user is sgarza.

![SAM Hive](/assets/img/2024-03-30/3_1.png){: width="600" }

The account creation time can be found by pulling the modification timestamp from the key `Domains\Account\Users\Names\sgarza`{: .filepath}: **2022-11-23 02:46:17**.

![sgarza Name Modified Timestamp](/assets/img/2024-03-30/3_2.png){: width="500" }

**Answer**: 2022-11-23 02:46:17

### 4. WHAT? I CANT HEAR YOU OVER THE FANS!

**Prompt**: What is the operating system edition?

OS information can be found in the `C:\Windows\System32\config\SOFTWARE`{: .filepath} hive under key `Microsoft\Windows NT\CurrentVersion`{: .filepath}. From the `ProductName`{: .filepath} and `ReleaseID`{: .filepath} key values, the OS edition is **Windows Server 2019 Datacenter (1809)**.

![OS Information](/assets/img/2024-03-30/4_1.png){: width="600" }

**Answer**: Windows Server 2019 Datacenter (1809)

### 5. Let’s go to Jackson and see the capital while we’re here.

**Prompt**: How many states did the user connect to the system from using the previously mentioned remote desktop tool?

This one took a while and I'm not even sure if I ended up in the correct area. I couldn't find anything of particular value in the Chrome application folders, but at some point I wondered if the remote client created any event logs, which would likely post to `C:\Windows\System32\winevt\Logs\Application.evtx`{: .filepath}. 

I exported the file and opened it in Event Viewer. I noticed some events with a source of "chromoting," which turns out to correspond to the Chrome Remote Desktop service. Interestingly, there are some IPs listed in some events. One of them looks familiar from the PC challenge: 34.162.97.100. 

I notice also that the various events seem to group under an identifier in the event description, e.g. chromoting_ftl_&lt;identifer&gt;. Looking at the log patterns, it seems that maybe Event ID 1 indicates a session start, Event ID 4 indicates a connection, and Event ID 2 indicates a session end. Event ID 4 contains IP information, as well as the connection type, "stun" or "relay," which seems to indicate a direct or third-party connection, respectively.

![Chromoting Event](/assets/img/2024-03-30/5_1.png){: width="600" }

Looking through the event logs, there are **two** destination IPs that fall under the "stun" connection type, 10.202.0.2 and 34.162.97.100. I'm not at all confident about the 10.202.0.2 address, and would have liked to have seen the other IP, 34.162.141.21. Maybe operate on the assumption that if the user connected to one, they connected to the other? 

**Answer**: two

### 6. I lost Control of my Services and broke all the Tcpipconnections. This is the last time I’m going to use random Parameters I don’t understand!

**Prompt**: What was the second domain in the search list provided by the DHCP server?

In the `C:\Windows\System32\config\SYSTEM`{: .filepath} hive, the key `CurrentControlSet\Services\Tcpip\Parameters\Interfaces`{: .filepath} gives information about the network interfaces on the system, including DHCP information. The third subkey has the most information, including a key `DhcpDomainSearchList`{: .filepath}. The second domain in the list is **c.boxwood-scope-369502.internal**.

![DhcpDomainSearchList](/assets/img/2024-03-30/6_1.png)

Interestingly, the `DhcpIPAddress`{: .filepath} is 10.202.0.2, the IP address found in the chromoting events in the previous question.

**Answer**: c.boxwood-scope-369502.internal

### 7. Which email did I use for this again?

**Prompt**: Not including their work account or gmail, what other email address did the primary user of the system have?

This one was easy. In Autopsy, under `Data Artifacts -> Web Form Autofill`{: .filepath} there are a few email addresses for the Default profile in Chrome: sgarza@kurvalis.com, sgarza1284@gmail.com, and **sgarza1284@proton.me**.

![Autofill](/assets/img/2024-03-30/7_1.png){: width="700"}

**Answer**: sgarza1284@proton.me

### 8. Can I sync these on my mobile device?

**Prompt**: What is the name for the bookmark item added on December 11, 2022 at 2:04:54 AM (local system time)?

The user's Chrome bookmarks are in `C:\Users\sgarza\AppData\Local\Google\Chrome\User Data\Default\Bookmarks`{: .filepath}. It's a JSON file, and each bookmark has a `date_added`{: .filepath} Chromium timestamp. Plugging these into Dcode, it seems that the bookmarks Bookmarks bar, Other bookmarks, and Mobile bookmarks were added at 2:04:54 AM, though the answer was **Mobile bookmarks**.

![Bookmarks](/assets/img/2024-03-30/8_1.png){: width="400"}

![Dcode](/assets/img/2024-03-30/8_2.png){: width="700"}

**Answer**: Mobile bookmarks

### 9. I’m tired of Googling about the Cloud. Time to learn about PowerShell modules instead.

**Prompt**: What is the GUID of the PowerShell module found on the system?

In Autopsy, `Data Artifacts -> Installed Programs`{: .filepath} says Google Cloud SDK is installed on the system. I find the folder for it in `C:\Program Files (x86)\Google\Cloud SDK`{: .filepath}. I look around for a while and stumble upon a file `powershell-windows.manifest`{: .filepath} in `.install`{: .filepath} pointing to various PowerShell files. 

![PowerShell Manifest](/assets/img/2024-03-30/9_1.png){: width="700"}

I navigate to the first path (`platform\GoogleCloudPowerShell\GoogleCloudPowerShell.psd1`{: .filepath}) and there I find the GUID: **e74637e6-7a4e-422d-bb9c-ca50809d78bb**.

![GoogleCloudPowerShell](/assets/img/2024-03-30/9_2.png){: width="500"}

**Answer**: e74637e6-7a4e-422d-bb9c-ca50809d78bb

### Debrief

I'm feeling a little forensically fatigued after staying up until 3am last night to finish the PC writeup, and I think that's partially the reason I felt kind of frustrated working through this one at times. I'm hoping to have a better time of it working through the 2024 CTF. I am noticing that I'm not needing as much help, which is great, and I'm proud of myself for thinking to look through the Application log for finding Chrome Remote Desktop events, though I do wish there was an official writeup to walk through that one. Until next time!


