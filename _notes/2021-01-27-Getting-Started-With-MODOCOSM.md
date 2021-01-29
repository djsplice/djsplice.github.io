---
title: Hello MODOCOSM!
tags: Ableton MaxForLive Music_Production
article_header:
  type: cover
  image:
    src: /assets/images/sample-image-6.jpg
---

# Getting started with MODOCOSM

Thanks for your interest in this Max For Live plugin for Ableton. Hopefully you find it inspires your creative use of the otherworldly Microcosm effects unit made by Hologram Electronics.

This plugin is in no way affiliated with Hologram Electronics, rather, it's a personal project built by me used as a creative interface to the Microcosm!

By now you've probably already read a bit about what MODOCOSM does, and are interested in getting it going yourself. You've come to the right page!

## Prerequisites
* This plugin was written using Max For Live, I believe you need to have Ableton Suite in order to use Max For Live plugins! I'm sorry if it doesn't work with Ableton Standard, I was a Standard user for a while until they had a good discount on updating to Ableton 11...
* You need to have a Microcosm effects box for this to do much of anything as intended!
* You also need a way of patching your Microcosm effects unit into your computer / Ableton. I use a Scarlett 18i8 external audio interface for this purpose and have patched the Microcosm inputs/outputs into it.
* Your Microcosm device needs to be connected to Ableton via MIDI configuration - I'll cover how I have mine setup below.
* Have an inquisitive and open mind.

## Configure your Microcosm
* Not much to do here, enter setup mode and specify the MIDI channel it should be listening on. Check out the Global Configuration section of the [Microcosm User Manual](https://b82316c2-7eca-4286-9569-a4da8097c930.filesusr.com/ugd/74428b_c4e6e20555914198bdb59c12f9a9e4d4.pdf) for more information.
* While you're in setup mode, ensure the Input Mode is configured as desired (stereo sounds great)

## Download and Install MODOCOSM
* Download MONOCOSM from the Max For Live site
* Locate the download file and double click to open it. This should install it under MIDI Effects Folder of Max For Live plugins in Ableton.

## Configure Microcosm in Ableton
There are a few ways to integrate external effects processors with Ableton. Ableton has a guide on [Using External Audio Effects](https://help.ableton.com/hc/en-us/articles/360005113200-Using-external-audio-effects) you should check out and decide which option best fits your studio setup.

I've tried setting it up as both an 'insert effect', and as a 'return channel'. I prefer using it as part of a return channel so that I can pass any / many instrument through it just as I would with other Ableton effects like the standard reverb and delay sends. In my demos I'll be showing the 'return channel' configuration option.

### Create new MIDI track for the Microcosm
* Create a new MIDI track, rename it to something meaningful to you, I use `MICROCOSM (M)`. I add the (M) to the name to make it easy to see at a glance if it's an Audio or MIDI track.
* Configure the MIDI track with the destination device and MIDI channel of your Microcosm device. My Microcosm is connected to Ableton via a MIDI Through port on a synth connected to the Scarlett audio interface, I've configured the Microcosm to listen on MIDI Channel 9.  
