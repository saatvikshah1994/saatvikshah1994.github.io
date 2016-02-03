---
layout: page
title: Brain Controlled SmartChair
description: "Just about everything you'll need to style in the theme: headings, paragraphs, blockquotes, tables, code blocks, and more."
modified:
intro_image: /projects/emotiv1.gif
image:
  feature: abstract-6.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
line1: Fully brain wave controlled wheelchair
line2: capable of locomotion and monitoring health
category : project
comments: false
---

## Where
[Zine Research Lab](http://zine.co.in/), NIT Jaipur

## Objective
A Brain Computer Interface to control a wheelchair, with:

1. Using signals originating completely from the brain, and **not** sources of noise such as eye-blinks or facial gestures
2. EEG headset to be used should be **economical** so as to make the system affordable
3. Monitoring patient mental health, to detect emergency situations

<br>
<figure>
	<center><a href="{{ site.baseurl }}/images/projects/emotiv1.gif"><img src="{{ site.baseurl }}/images/projects/emotiv1.gif" alt="" height="350px" width="350px"></a></center>
	<center><figcaption><b>Emotiv EEG headset</b></figcaption></center>
</figure>

## Methodology
The previous research on Motor-Imagery and P300 EEG were pivotal starting points for this project. Utilizing their software implementations in a GUI constructed using wxPython and MPI, to synchronize the recorded EEG with events on the GUI, a proof-of-concept was constructed. The [*emokit*](https://github.com/openyou/emokit) library was used for recording EEG.

<figure>
	<center><a href="{{ site.baseurl }}/images/projects/p300speller.png"><img src="{{ site.baseurl }}/images/projects/p300speller.png" alt="" height="350px" width="350px"></a></center>
	<center><figcaption><b>P300 Speller GUI</b></figcaption></center>
</figure>

## Results
1. Accuracies were not promising, ranging from 65-70%. This resulted in a poorly functioning system.
2. Upon running a number of tests, we realized that the electrode positions were problematic, in addition to the large amount of noise given out by the Emotiv.