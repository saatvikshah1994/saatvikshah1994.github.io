---
layout: post
title: A Glance at Ensembling
image:
  feature: /ensembling_1/ensemble_introimg.jpg 
description: "Motivation, Bagging, Boosting and other Variants, Use Cases"
tags: [Machine Learning]
---

[//]: # (Image from http://cse-wiki.unl.edu/wiki/index.php/Bagging_and_Boosting)

## Getting Started
This is going to be the first of a series of posts about **Ensembling** - understanding all the *whats,hows,wheres and whys*? 

Without much ado lets get started!
The usual modus-operandi of searching for the term on [Google](http://www.google.com) would spit out the following [Wikipedia](en.wikipedia.org) definition:
<figure>
	<center><a href="{{ site.baseurl }}/images/ensembling_1/ensembling_defshot.png"><img src="{{ site.baseurl }}/images/ensembling_1/ensembling_defshot.png" alt="" height="600px" width="400px"></a></center>
</figure>

Not bad, in fact let's stick with the first half of the definition above

> An ensemble is a technique for combining many weak learners in an attempt to produce a strong learner.

The key term here is *combining* and *learners*. Here the *learners* refer to the algorithm which you are using for estimating values from testing data(be it classification or regression), once they have learnt from the given training data.

For a simple analogy, lets say you are a learner and, well you are here trying to learn how Ensembles work. You would have gone through a number of research papers before you stumbled upon this blog post. Now say your 2 besties are also studying the same topic, but instead one went through some [tutorials](http://scikit-learn.org/stable/modules/ensemble.html) online, while your second *lazy* friend decided to give the [Wikipedia page](http://en.wikipedia.org/wiki/Ensemble_learning) a cursory glance. Now when the three of you sit down to make a report, what is finally written will be derived from the overall knowledge from all 3 of you. Quite often your *lazy* friend would give some wrong information, but thanks to the others knowledge the mistake is rectified. Additionally with the combination of knowledge present in the room, you'll end up writing a report way better than you could alone. And thats how you end up getting an *A+* on your work :).

## Taking a Deep Dive
Come to think of the situation described above, the real reason why the three of you could produce better results than any single one, is because there were *variations in your understanding of the subject* i.e. while you understood very clearly how Ensembles work, your friends knew about the correct type of ensembles one should apply and where(which might have been you gray area). Thus the combined product was way better.

In a similar way 2 major conditions for a learner in an ensemble is

1. **Accuracy** : The learner should predict results, more accurate than random guessing
2. **Diversity** : The learner should give different errors on newly encountered data points during training

<figure>
	<center><a href="{{ site.baseurl }}/images/ensembling_1/EnsembleDepiction.png"><img src="{{ site.baseurl }}/images/ensembling_1/EnsembleDepiction.png" alt="" height="600px" width="600px"></a></center>
	<center><figcaption><b>Depiction of an Ensemble</b></figcaption></center>
</figure>

## Why do Ensembles work?
**3** Simple Reasons:

1. **Statistical Reason** : The learning algorithm is able to formulate multiple different representations from the input data, which may individually give similar accuracies with slight variations. By merging their results we can get an overall view of the problem space aka *a bigger picture*.

2. **Computational Reason** : Learners are more than often prone to getting stuck in local minimas of the problem space - these are good solutions but not the best ones. Thus by initializing multiple learners with different weights, we can ensure that majority of the learners are able to reach in close proximity to the global minima

3. **Representational Reason** : Since different learners learn different hypothesis or representations of the input, an ensemble is able to represent a solution as a combination of different hypothesis. This reason may seem a bit vague so it is ok to concentrate on the first 2 for now.

<figure>
	<center><a href="{{ site.baseurl }}/images/ensembling_1/3reasonsensemble.png"><img src="{{ site.baseurl }}/images/ensembling_1/3reasonsensemble.png" alt="" height="500px" width="500px"></a></center>
	<center><figcaption><b>The Big 3</b></figcaption></center>
</figure>

## How are they made?
Lets now come to the different methods by which Ensembles can be constructed. The most commonly used technique for this is of manipulating training examples or input features and injecting randomness. There are a few methods of manipulating output targets but I wont be covering this(though I'll give you a few resources towards the end if your really interested)

### Bagging
Simply picking random samples from the training examples, with replacement, for the learner to learn from is called Bagging. On an average each such subset(known as *Bootstrap Replicate*) contains *63.2%* of the original training set.
<figure>
	<center><a href="{{ site.baseurl }}/images/ensembling_1/bagging.png"><img src="{{ site.baseurl }}/images/ensembling_1/bagging.png" alt="" height="500px" width="500px"></a></center>
	<center><figcaption><b>Bagging Classifier</b></figcaption></center>
</figure>


### Boosting
When the first hypothesis or data representation is learnt, the erroneously learnt samples are trained with more weightage i.e. the importance is given to the misclassified examples. In this way Boosting algorithms are able to learn how to correctly represent harder representations. This method has been applied in **AdaBoost** and **Gradient Boosting** ensembling learners.
<figure>
	<center><a href="{{ site.baseurl }}/images/ensembling_1/boosting.png"><img src="{{ site.baseurl }}/images/ensembling_1/boosting.png" alt="" height="500px" width="500px"></a></center>
	<center><figcaption><b>Boosting Classifier</b></figcaption></center>
</figure>

### Feature Manipulation
The learners are trained on random subsets of the features instead of random subsets of samples. For example 3 neural networks can be trained on 5 features such that the first learns feat1-feat3 while the next 2 learn feat4 and feat5. However this technique works only when **the input features are highly redundant**.

### Injecting Randomness
Works best with highly stochastic and unstable algorithms - In this method we simply initialize our learners with different parameters randomly. For example a neural network can be initialized with differed *rng's*(random number generator) resulting in the weights being initialized differently each time. It helps reduce overfitting in such unstable algorithms.

## Where should we apply Ensembling?
Quite honestly, I have seen that some sort of ensemble is always useful in pushing results in your favour. The bigger question is that "**Which Ensemble should we apply where?**". Here are a few tips I can share on this topic:

1. Inject Randomness when using Neural Networks and Genetic Algorithms or their variations. Simply generate a large number of the same learner, only with a new *rng*. Then taking a majority vote should make a large difference.

2. For Linear Classifiers, use Bagging to ameliorate results. However the number of classifiers required in the ensemble maybe large.

3. For non linear classifiers such as Decision Trees, Bagging with randomly selected features at every level(as used in [RandomForests](http://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestClassifier.html)) 

Oh Wait! One last thing - Please leave your comments or any other feedback which you might have. If theres anything you would recommend on improving be sure to leave it in the comments. You could also shoot me a mail at *saatvikshah1994@gmail.com*. 

## References
1. Ensemble Methods in Machine Learning by Thomas G. Dietterich
2. Scikit Learn's Ensembling [Documentation](http://scikit-learn.org/stable/modules/ensemble.html)
