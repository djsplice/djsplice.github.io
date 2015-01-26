---
layout: post
title: "Automation First"
modified:
categories:
excerpt: "A lesson learned, Automation First - It's a good thing! "
tags: [pi, xig, chef]
image:
  feature: sample-image-5.jpg
  credit: WeGraphics
  creditlink: http://wegraphics.net/downloads/free-ultimate-blurred-background-pack/
date: 2015-01-25T09:58:34-08:00
comments: true
---

## Automated Infrastructure Development

It's a bit embarrassing to admit, given what I do for a living, but I've got a good practical example of why you might consider adopting an automated infrastructure development model, and Automate First!

I've completely reversed my position on the topic of systems automation. If you were to have asked me several years ago about how I tackled systems automation, I would've probably said, well first, I manually install all the things to get the system working, then if I think I'm going to have to build the system again, I'd put the time and effort in to automate the setup.

For the most part, the tools I was working with then were a bit cumbersome to use, so I always considered automation 'extra work' that I needed to do only to meet the requirements of standardization, consistency, and repeatability in a production environment. It was definitely not my standard approach to building systems.

All that changed when I started using Chef and TestKitchen (KitchenCI). This toolset has helped reduce many of the more 'cumbersome' barriers I'd experienced in the past.
* By leveraging the vast community support Chef cookbooks, I'm able to get systems deployed relatively quickly, in a relatively sane way.
* Using TestKitchen allows me to quickly experiment with infrastructure code on VM's on my laptop, and to quickly iterate on configuration details.
* Saving these configurations (cookbooks) in Github makes it super easy to track changes, file defects/enhancements, and continue to enhance the systems I am building over time.

Now, I can get started very quickly composing my system based on the shared contributions of the great developers in the Chef community, adding a thin layer of customization to suit my particular applications needs.

## Taking an 'Automation Later' Approach
I've been happily running a piece of software called Xig as the core controller for a home automation platform I'm developing. It basically acts as an internet gateway for Xbee radios that are connected to some Arduino microcontrollers. I'd set this system up over 2 years ago on a Raspberry Pi, did a fair amount of manual configuration to get everything working, and moved on with other development tasks.

#### Everything included:
* Raspberry Pi tweaks (hardware watchdog, swap, other??)
* The Xig Software itself
* AutoSSH
* Monit
* CollectD
* Various init scripts

Well, the other day, things just stopped working, so I logged into the Pi and found that the SD card had become corrupted... Try as I may (read: desperately), I couldn't recover the filesystem.

Cue the sad trombone.

### Now, start over...
I'm a bit older now, have picked up some new tricks, and am determined not to repeat my mistakes... Let's build this thing up so that if (when) my SD card implodes again, I won't be so sad, and moreover, can get back on the road quickly without having to embark on a week long Evernote archeology expedition!

## Automation to the rescue
I've learned a good amount about Chef over the past few years, and decided to see what it would take to build and manage my application on the Raspberry Pi using Chef Cookbooks.

Turns out it's not such a crazy idea after all! There are a few guides showing how to install the Chef client on Raspbian, and how to use knife-solo to run cookbooks. The knife-solo development flow is slightly different than the TestKitchen method I'm used to, but once I got the differences worked out, I was off to the races.

This time, instead of starting out manually installing all of the components that made up my system, I started building using a combination of Chef Community cookbooks and some standard Chef constructs to template various configuration files.

#### Chef Components
* The APT community cookbook allowed me to manage the different supporting packages I needed (like hardware watchdog for the Pi), as well as update the OS.

* I was able to install the Xig software directly from my Github account using Chefs Git integration

* I installed and configured Monit and CollectD services using Community developed cookbooks.

* Parameterized my various configuration files, and mapped default entries using Chef Attributes.

## Automation FTW!
Now, when my SD card goes away, I'm able to transfer my Chef Bootstrapped Raspbian image to a new card, pull down my cookbooks from github, and use knife-solo to configure the system. All told it takes about 30 minutes to run through the cookbook, and at the end I have a fully functional system.
