---
author: Davide Quaranta
title: "NVIDIA drivers and Ubuntu: an horror story"
date: 2022-11-21T09:00:00+02:00
categories: [Linux]
tags: [desktop, nvidia, ubuntu]
description: "A short horror story (with a nice ending) featuring Ubuntu and NVIDIA drivers."
---

It is a widely accepted truth that NVIDIA and Linux are a bad combination. There are some stories about broken systems after drivers updates; I know a person who is afraid to update his system because of past horror experiences.

For the past 3 years my Ubuntu+NVIDIA experience has been quite smooth, until last week. I don't really know what caused it, but out of the blue my system stopped to be reliable after suspension. In this post I describe the symptoms and what I did to finally solve the problem.

## Symptoms

After the unknown culprit update, my system started to behave like that:
1. No lock screen video after resuming from suspension.
2. Broken <span style="font-variant:small-caps;">X11</span> after logging back in.
3. No video in virtual <span style="font-variant:small-caps;">TTY</span> consoles (`CTRL+F1`, ... `F6`)
4. Occasional fails to resume from suspension (basically the system appeared as shut down) or to shutdown (stuck with black screen).

Not pleasant.

## Short-term treatment

When my system became sick, I tried to get around the issues. The issues from 1 to 3 can be short-term solved without rebooting, by just using the system without video to log back in, and to use keyboard shortcuts to open a terminal and type something in it; for example:

1. Press a key to resume from suspension.
2. Type your password without video output.
3. Use a keyboard shortcut to open a terminal.
4. Restart your window manager, like `mutter --replace &` or `xfwm4 --replace &`.
5. If needed, restart your compositor.

In my case, these operations were sufficient to **resume** a normal system state **without losing the current session**. The operations can be also done in a <span style="font-variant:small-caps;">TTY</span> virtual console, still without any video feedback.

This strategy—however—is not sustainable.

## Medium-term treatment

A super easy fix consists of disabling the proprietary NVIDIA driver and sticking with the [noveau](https://nouveau.freedesktop.org/) driver.

That is possible to do for free by removing any trace of NVIDIA from the system: `sudo apt purge \*nvidia\*`.

It may also be needed to manually remove [NVIDIA power management services](https://forums.developer.nvidia.com/t/fixed-suspend-resume-issues-with-the-driver-version-470/187150/3):

```shell
sudo systemctl stop nvidia-suspend.service
sudo systemctl stop nvidia-hibernate.service
sudo systemctl stop nvidia-resume.service

sudo systemctl disable nvidia-suspend.service
sudo systemctl disable nvidia-hibernate.service
sudo systemctl disable nvidia-resume.service
```

After a reboot, you have a system that works and uses the open source nouveau video driver. But you also have a system **without hardware acceleration**, which uses 90% of your <span style="font-variant:small-caps;">CPU</span> just for a video call.

Maybe we need to reach a peaceful agreement with the NVIDIA proprietary driver.

## Long-term treatment

That is how I discovered the appropriate things to do and how I ended up with.

### Experimenting (the horror part)

This is the horror part of the story. One day I decided to install the `nvidia-driver-515-open`. The result was that **the video was completely absent at boot**.

Here is when [I discovered](https://www.google.com/search?q=nomodeset) that I could temporarily **defer the initialization** of the video driver, by adding a keyword in the Grub2 configuration. To do that at boot:

1. Move to your <span style="font-variant:small-caps;">OS</span> entry on the Grub2 menu.
2. Press the `e` key to enter in *edit mode*.
3. Add `nomodeset` at the end of the line that starts with `linux`.
4. Exit.

That fixed the black-screen problem and I could enter in my system.

### The right plan (light bulb part)

Now that I knew that I could experiment with drivers and just use the `nomodeset` trick to bypass any no-video issue, I decided to press the accelerator.

I found out that `ubuntu-drivers devices` tells you not only the available drivers, but also the **recommended** one for your system.

The plan:

1. Cleanup everything NVIDIA from my system.
2. Install the recommended driver.
3. Eventually use the `nomodeset` trick to revert in case of boot problems.

### Finding peace with NVIDIA

I followed the plan by installing `nvidia-driver-515` and I basically restored the initial starting configuration with the broken power management.

But booting with `nomodeset` fixed the power management issues.

Hence, the solution was easy: make the `nomodeset` trick permanent.

1. Do `sudo vim /etc/default/grub`.
2. Add `nomodeset` at the end of `GRUB_CMDLINE_LINUX_DEFAULT`.
3. Save and exit.
4. Refresh the configuration with `sudo update-grub2`.

That fixed everything.

## Conclusions

The take-home message from this experience is that, if you have problems with NVIDIA cards:

1. Cleanup everything related to NVIDIA.
2. Install the recommended proprietary driver for your system.
3. Add the `nomodeset` parameter to the boot options.

Step 3 may be different according to the specific video card: there are other kernel flags that can be played with if `nomodeset` doesn't work.