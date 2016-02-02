---
layout: page
title: Motor-Imagery EEG
description: "Genetic and Evolutionary algorithms on a multilayer neural-net for detecting Motor-Imagery signals"
modified:
intro_image: /projects/genetic2.png 
image:
  feature: abstract-6.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/  
line1: Genetic and Evolutionary algorithms on a multilayer
line2: neural-net for detecting Motor-Imagery signals
category : project
comments: false
---

##Where
[*Zine Research Lab*](http://www.zine.co.in), NIT Jaipur

##Objective
EEG or Electroencephalogram refers to recordings of electrical activity in the brain. [Motor-Imagery EEG](http://www.wikiwand.com/en/Motor_imagery) is generated when users simulate an action mentally, such as *imagining your left hand move*. Our objective was to recognize these signals from processed EEG using [stochastic search algorithms](http://www.wikiwand.com/en/Stochastic_optimization), where we focussed on Genetic and Evolutionary algorithms.

##Methodology
This was one of my first research initiatives during 2nd year of engineering. With the guidance of seniors, I learned and implemented various such algorithms. After applying several published methods and additional improvements to some classic algorithms in this area(*PSO*,*DEs*,etc) we were able to formulate two promising algorithms which showed impressive results.

1. **BSA-NN** : A randomized backtracking swarm on a three layer neural-net, with minimal parameter tuning and parallelized OVA(one vs. all multiclass implementation). 

2. **GSEA** : Iterative group based evolution and mutation scheme, dividing the swarm into multiple fitness groups.

<figure>
	<center><a href="{{ site.baseurl }}/images/projects/genetic2.png"><img src="{{ site.baseurl }}/images/projects/genetic2.png" alt="" height="200px" width="400px"></a></center>
	<center><figcaption><b>GSEA Architecture</b></figcaption></center>
</figure>

##Results

*Accuracy of 69% across 3 different subjects on a multiclass motor-imagery problem.* This performance is better than 21 previous approaches including the winning approach of BCI competition 3(dataset 2).

##Publication

####1. SK Agarwal, Saatvik Shah, and Rajesh Kumar. *"Classification of mental tasks from EEG data using backtracking search optimization based neural classifier."* Neurocomputing (2015). [URL](http://www.sciencedirect.com/science/article/pii/S0925231215003409)

####2. SK Agarwal , Saatvik Shah, and Rajesh Kumar. *"Group based Swarm evolution algorithm (GSEA) driven mental task classifier."* Memetic Computing 7.1 (2015): 19-27. [URL](http://link.springer.com/article/10.1007%2Fs12293-015-0155-0)


##[Code@Github](https://github.com/saatvikshah1994/bsa-nn)