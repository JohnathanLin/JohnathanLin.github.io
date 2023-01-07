---
title: "Windows XP Virtual Machine Chinese Version Without Activation Image Download"
date: 2022-10-02T18:51:17+08:00
draft: false
categories: ["Tech", "Nostalgic"]
tags: ["WinXp"]
---
# Click here to download my virtual machine image
Download virtual machine image:

Link：https://pan.baidu.com/s/1yfY0SjDrtOeuTiEWf7YizA?pwd=374l 

Fetch Code：374l

My VirtualBox version is 6.1

# Let me talk about nostalgic
For most people, Win95 is their youth, so they create UI component library [React95](https://github.com/arturbien/React95), or create an OS [serenity](https://github.com/SerenityOS/serenity) together. But for most Chinese people, when they got PC in 2000-2010, they used Windows XP in their PC.

[sh1zuku](https://sh1zuku.csie.io/#/) is from China, Taiwan, he created a Windows XP simulator on website: [Project Link](https://github.com/ShizukuIchi/winXP), [Online Demo](https://winxp.vercel.app/), and another programmer from Vietnam wrote a Windows 7 Simulator on website [Online Demo](https://win7simu.visnalize.com/).

Honestly, I don't have enough ability to develop simulator program like them, but I have an idea that I can install a Windows XP virtual machine on my PC.

# The steps you might choose
1. Install VirtualBox.
2. Download Image: Find Windows XP Image in [MSDN Itellyou](https://msdn.itellyou.cn/) and then download it.
3. Install the Image in VirtualBox.
4. Run the installed image, and install the OS.

Then, you will discover that the Windows XP only has 30-day trail. In the early 2000s, piracy was rampant, and Microsoft took strong measures to limit it. Even if you enter the correct serial number when you install, Microsoft will ask you to activate using a phone, etc.

But I just want to experienct the old OS in my PC! (It's too hard for me to activate a copyrighted WinXP!)

# The Right Solution
So I don't use this copyrighted WinXP image anymore, I should search some WinXP image without activation.

Here, I found [github WinXPImage](https://github.com/lucianoferrari/winxpimage) from Github, but his Image file saved on Google Drive. I have downloaded that and uploaded to Baidu Cloud:

Link：https://pan.baidu.com/s/1ypKeaZixJXnbqAo4ZT0YLQ?pwd=30dd 
Fetch Code：30dd

When you install this image on VirtualBox, you will get a WinXP(**English Edition**) that you can use indefinitely.

But you may not be able to start this Image successfully, you should execute this after importing the OVA File:
``` cmd
vbox-img geometry --filename Windows_XP_Professional-disk1.vdi --format VDI --cylinders 5874 --heads 255 --sectors 56
```
From [github issue](https://github.com/lucianoferrari/winxpimage/issues/1) streeg's solution.

# How to Chinesize Windows XP?
I want to use Chinese Windows XP to reminisce about the past, but this image doesn't have Chinese language support.

After searching many source, I finally found a language package mui_win_xp_pro_n_cd1 in [here](https://msdn.alicesworld.tech/Windows%20XP).

I downloaded it and uploaded to Baidu Cloud:

Link：https://pan.baidu.com/s/18mW9OCRejMDoEpUcih-zlA?pwd=peu4 

Fetch Code：peu4

Then,
1. Configure share folder in Windows XP virtual machine. [Source](https://www.jianshu.com/p/060103d48244)
2. Transfer mui_win_xp_pro_n_cd1.iso to Windows XP virtual machine by share folder.
3. Install it, [Source](https://www.docin.com/p-2998336055.html)

Finally, reboot Windows XP virtual machine.

Screenshot:
![](/images/nostalgic_xp.png)
