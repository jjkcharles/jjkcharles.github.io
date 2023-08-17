---
layout: post
title: Mobile Hotspot - a minor inconvenience
subtitle: A minor inconvenience and the tiny little hack!
author: jjk_charles
categories: tips
tags: tips mobile internet
sidebar: []
---

Just a couple days after [I thought I had it all figured out with my mobile internet situation]({% post_url 2023-03-17-internet-hotspot %}), I started running into a problem.

Calling it a problem might be a little exaggeration though - as it is more of a minor inconvenience than anything else.

So, after I started using Cox Hotspots on my laptop, I eventually started using it on my mobile as well. While this worked well overall, with the way Wi-Fi connections work in Windows & Android, I ended up being on the public Hotspot network even when I am at home.

Sometimes, I consciously  switch back to my home network only to find out later that somehow my mobile/laptop has decided to hop back on to the public network.

This proved especially annoying when I am on Teams calls and encounter some network issues only to realize that the culprit was me being connected to the public network.

Neither Windows nor Android allows an easy way to manage the order in which known Wifi networks are chosen by the system, when more than one becomes available in range.

Windows is in a much better spot, as it allows configuring it when we have elevated permissions on the machine. Refer to [this SuperUser answer on how to do it](https://superuser.com/a/994983).

The key here is to run the below command to modify the "priority" of known networks as per our needs. Priority 1 being the most preferred connection.

```powershell
netsh wlan set profileorder name="connectionname" interface="Wi-Fi" priority=1
```

Unfortunately, my work laptop doesn't allow elevated permissions for user accounts. So, I had to look for alternatives.

### Solution
I was fiddling around with the network settings on both Windows & Android and felt that "Metered Connection" settings might come in handy here. I knew that Android prioritizes non-metered connection over metered connections. This got me thinking - may be I can use it to my advantage to make Windows/Android prefer my home Wi-Fi over Cox Wi-Fi if both are within the range. 

And, it did work out the way I anticipated!

Below is how I did it on Windows and Android. 

#### Windows
Go to `Settings->Network & internet->Wi-Fi->Manage Known Networks`

And, choose the target "public" wifi network. Flip the "Metered Connection" toggle to ON.

![Windows Wifi Settings](https://i.postimg.cc/FKhhvWSK/image.png "Windows Wifi Settings - Windows 11")

#### Android
Depending on your Android flavor, go to `Internet->Saved Networks` and choose your target network.

Once there, change `Network Usage` from "Detect automatically" to "Treat as metered".

![Android Wifi settings](https://i.postimg.cc/FzHh5DDF/Untitled.jpg "Android Wifi settings - Pixel 7")

Again, steps could be different depending on Android version as well as the vendor customization. Above is based on Google Pixel 7 (Stock OS).