---
layout: post
title: Sentiment Analysis Primer - Part 2
image:
  feature: /smr_intro/smileys-bg.jpg 
description: "Sentiment Analysis Primer"
tags: [sentiment analysis]
category: blog-post
---

<h2 id='intro'>Introduction</h2>
In Part 2, we'll look at an unsupervised approach proposed by *Peter Turney* in his seminal paper titled [Thumbs Up or Thumbs Down? Semantic Orientation Applied to Unsupervised Classification of Reviews](http://www.aclweb.org/anthology/P02-1053.pdf). The approach is unique compared to traditional dictionary and BOW based techniques, in that it is completely unsupervised and makes use of information retrieval from **Knowledge Bases**(via *Search Engines*).

**Note :** Information about datasets, metrics and preprocessing being used has already been outlined in [Sentiment Analysis Primer - Part 1]({{site.baseurl}}/blog-post/SMR_Intro/)

<h2 id="outline">Outline</h2>
<ol style="line-height:1.2em">
<li><a href="#caution">Search Engine Scraping : Be Wary!</a></li>
<li><a href="#overview">Technique Overview</a></li>
<li><a href="#step1">Phrase extraction</a></li>
<li><a href="#step2">The PMI-IR score</a></li>
<li><a href="#code">The Code</a></li>
<li><a href="#results">Results</a></li>
<li><a href="#whatsnext">What's next?</a></li>
<li><a href="#refs">References</a></li>
</ol>


<h2 id="caution">Search Engine Scraping : Be Wary!</h2>

<figure>
    <center><a href="{{ site.url }}/images/smr_p2/search-engines.jpg"><img src="{{ site.url }}/images/smr_p2/search-engines.jpg" alt="" height="400px" width="400px" align="middle" /></a></center>
</figure>

This approach involved scraping specific sections of search engine(such as [Google](https://www.google.co.in/?gws_rd=ssl),[Bing](https://www.bing.com/)) results. 

>  This line of research depends on the good will of major search engines.

Use of screen scrapers is disallowed by these engines. However they do provide APIs:

1. [Google Custom Search API](https://developers.google.com/custom-search/). Read more [here](http://stackoverflow.com/questions/4082966/what-are-the-alternatives-now-that-the-google-web-search-api-has-been-deprecated) and [here](http://stackoverflow.com/questions/22657548/is-it-ok-to-scrape-data-from-google-results) : **~40 requests/hour**
2. [Microsoft Bing Search API](http://www.bing.com/toolbox/bingsearchapi) and the [python-bing](https://github.com/tristantao/py-bing-search) API client : **~5K requests/month**

The author of the paper has himself used the [Alta Vista](https://en.wikipedia.org/wiki/AltaVista) search engine which was popular in the pre-Google days.

<h2 id="overview">Technique Overview</h2>

<figure>
    <center><a href="{{ site.url }}/images/smr_p2/technique_overview.png"><img src="{{ site.url }}/images/smr_p2/technique_overview.png" alt="" height="400px" width="600px" align="middle" /></a></center>
</figure>

The above diagram shows gives a rough overview of how the proposed technique works. The main steps involved in obtaining the semantic orientation score are **Phrase Extraction** and **PMI Scoring**.

<h2 id="step1">Step 1 : Phrase Extraction</h2>
The heavy dependence of this technique on querying commercial search engines coupled with the importance of only specific substrings in adding to sentence sentiment means that it would be most sensible to extract and query only such specific phrases. The author has proposed that only those phrases following specific **part of speech** patterns be queried to obtain the final score. The table of acceptable patterns is shown below:

<figure>
    <center><a href="{{ site.url }}/images/smr_p2/acceptable_patterns.png"><img src="{{ site.url }}/images/smr_p2/acceptable_patterns.png" alt="" height="300px" width="300px" align="middle" /></a></center>
</figure>

The steps for phrase extraction are:

1. Run a *Part of Speech tagger*(such as the one provided by NLTK) on the concerned sentence.
2. Check every triplet word combination pattern. If a valid pattern is found in correspondence with the above table then pass it for querying and scoring.

<figure>
    <center><a href="{{ site.url }}/images/smr_p2/phrase_extraction.png"><img src="{{ site.url }}/images/smr_p2/phrase_extraction.png" alt="" height="500px" width="800px" align="middle" /></a></center>
</figure>

<h2 id="step2">Step 2 : The PMI-IR score</h2>
**PMI-IR** stands for *Pointwise Mutual Information score* which can be denoted as: 

$$PMI(word1,word2)=log_{2}(\frac{hits(``word1"\ near:10\ ``word2")}{hits(word1)*hits(word2)})$$

*where hits(x) = Number of results returned when x is issued as a query*

This can be understood from the diagram below:

<figure>
    <center><a href="{{ site.url }}/images/smr_p2/semantic_orientation.png"><img src="{{ site.url }}/images/smr_p2/semantic_orientation.png" alt="" height="500px" width="800px" align="middle" /></a></center>
</figure>

Finally, the *Semantic Orientation score* can be calculated by

$$SO_{score}(phrase)=PMI(phrase,``excellent")-PMI(phrase,``poor")$$

This can also be expressed as(by applying log multiplication rule):

$$SO_{score}(phrase)=log_{2}(\frac{hits(``phrase"\ near:10\ ``excellent")*hits(``poor")}{hits(``phrase"\ near:10\ ``poor")*hits(``excellent")})$$

<h2 id="code">The Code</h2>
{% gist saatvikshah1994/d53bd73c095a80ad5d1c %}

**Note:** The **utilities.py** script contains some helper functions that have been used in the above code and can be found [here](https://gist.github.com/saatvikshah1994/0353fa06298f81b58cd5). Ensure that you modify the dataset path in the *load_data* method correctly.

<h2 id="results">Results</h2>
The proposed technique is reported to have an accuracy of approximately **66%** on movie review data from the Epinions(reference pending) dataset. Due to :

1. The restrictions on screen scraping my modern search engines
2. Strict restrictions on number of API calls by search engine APIs

I have not run a test on the complete dataset. Tests on a small subsample gave decent results. I have currently thought out a plan to overcome the screen scraping limitations, but this is currently in development.

<h2 id='whatsnext'>What's next?</h2>
Next up, I'll be going into the details of a completely unsupervised system for sentiment analysis, by utilising **search engine results**.

<h2 id='refs'>References</h2>
<h4>Papers</h4>
1. [Thumbs up or Thumbs Down? Semantic Orientation Applied to Unsupervised Classsification of Reviews](http://www.aclweb.org/anthology/P02-1053.pdf)