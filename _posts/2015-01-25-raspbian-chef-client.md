---
layout: post
title: "Raspbian and Chef-Client"
modified:
categories:
excerpt: "Bootstrapping Raspbian disk image with Chef client"
tags: [pi, xig, chef]
image:
  feature: sample-image-3.jpg
  credit: WeGraphics
  creditlink: http://wegraphics.net/downloads/free-ultimate-blurred-background-pack/
date: 2015-01-25T07:58:34-08:00
comments: true
---

Here are the high level steps I followed to install the chef client on the Pi, and also create a disk image that contained the base os with the chef client pre-installed.

These instructions are largely OSX specific.

#### Transfer the Raspbian disk image to the SD card
1. Run `diskutil` to determine where your SD Card device (I ran it before inserting the card, and after inserting, so I was confident I knew which device I was about to write over).
1. Run `diskutil unmountDisk /dev/disk1` to unmount your SD Card (if that is indeed your device)
1. Transfer the Raspbian image you've downloaded to the SD Card:
```
sudo dd bs=1m if=~/Downloads/2014-12-24-wheezy-raspbian.img of=/dev/disk1
```

This can take some time, so be patient, you should see activity on the SD Card reader... Blink Blink.

Once complete, boot the Pi, and don't do anything else, you want to keep the image factory fresh, and add the Chef client. You are now ready to install the Chef client.


#### Installing the Chef Client using 'Knife'
1. Install knife-solo gem:
```
gem install knife-solo
```
1. Ensure your Pi is booted with the base image you want to update
1. Install the Chef Client on your Pi:
```
knife solo prepare pi@your_pi_ip
```

This will take some time as it's got do download a fair bit from the internet. Once complete though, you will have a chef client installed on the Pi you can use to run your Cookbooks. More on that later.


* Power off the Pi.

#### Create a new disk image
1. Take the SD card from the PI and plug it into your desktop computer
1. Identify the device again using the diskutil steps from above
1. Unmount the device
1. Use `dd` to create your new image:

~~~
sudo dd bs=1m if=/dev/disk1 of=~/Downloads/2014-12-24-wheezy-chef-11-raspbian.img
~~~
