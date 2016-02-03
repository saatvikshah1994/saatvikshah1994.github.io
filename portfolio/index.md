---
title: Portfolio
layout: portfolio-template
description: 
tags: [about]
comments: false
image:
  feature: websitebg1.jpg
---


<h1 align="center" style="font-family:'Lato',Calibri,serif;font-size:4em">Projects</h1>
<div class="container text-center" style="margin-left:-190px">
<div class="jumbotron">	
	<!-- Example row of columns -->
	{% assign project_counter = 0 %}
	{% assign global_counter = 0 %}
	{% assign array_complete = 0 %}
	{% assign colsize = 3 %}
	{% for project in site.posts %}
		{% assign global_counter = global_counter | plus: 1 %}
		{% if project.category != "project" %}
		<!-- Non-Project here -->
		{% else %}
		<!-- Project here -->
			{% assign project_counter = project_counter | plus: 1 %}
			{% assign mod = project_counter | modulo: colsize %}
			{% if mod == 1 %}
				<div class="row center-block">
			{% endif %}
			<a href="{{ site.url }}{{ project.url }}">
			<div class="col-md-4" align="center" style="padding:15px 30px 30px 15px">
					<div class="btn btn-default btn-lg" role="button" style="height:360px;width:300px;">
					<img src="{{site.url}}/images/{{project.intro_image}}" alt="..." class="img-thumbnail" style="height:280px;width:295px;">
					<h5><strong>{{project.title}}</strong></h5>
					<h6 align="center">{{project.line1}}<br>{{project.line2}}</h6>
					</div>
			</div>				
			</a>
			{% if mod == 0  %}
				</div>
				{% assign array_complete = 1  %}
			{% endif %}
		{% endif %}
		{% if array_complete == 0 and global_counter == site.posts.size  %}
			</div>
		{% endif %}		 
	{% endfor %}
</div>
</div>
<hr>
<h1 align="center"  style="font-family:'Lato',Calibri,serif;font-size:3em">Publications</h1>


####1. SK Agarwal, Saatvik Shah, and Rajesh Kumar. *"Classification of mental tasks from EEG data using backtracking search optimization based neural classifier."* Neurocomputing (2015). [URL](http://www.sciencedirect.com/science/article/pii/S0925231215003409)

####2. SK Agarwal , Saatvik Shah, and Rajesh Kumar. *"Group based Swarm evolution algorithm (GSEA) driven mental task classifier."* Memetic Computing 7.1 (2015): 19-27. [URL](http://link.springer.com/article/10.1007%2Fs12293-015-0155-0)

####3. Saatvik Shah,Vaibhav Jain,Sarthak Jain,Anshul Mittal,Jatin Verma,Shubham Tripathi and Rajesh Kumar. *"Hierarchical classification for multilingual language identification and named entity recognition."* Proceedings of the Forum for Information Retrieval Evaluation, 2015(In Press). <br> [URL](http://irlab.daiict.ac.in/FIRE2015/WorkingNotes/T2-35-38-Shah.pdf) [Presentation]({{site.url}}/assets/extradocs/msir_fire2015.pdf)

####4. Saatvik Shah,Anirudha Kumar and Rajesh Kumar. *"A Robust framework for optimum feature extraction and recognition of P300 from raw EEG."* Neural Computing and Applications, 2015(Under Review) [URL](https://drive.google.com/file/d/0B3qdXauTZfQCMWppNVAzaHk1S1U/view?usp=sharing)

<hr>
<h1 align="center" style="font-family:'Lato',Calibri,serif;font-size:3em">Patents</h1>

#### *"Railway Maintenance Monitoring System."*, Application Number : 1098/DEL/2014. [URL](http://ipindiaonline.gov.in/patentsearch/PublishedSearch/publishApplicationNumber.aspx?application_number=0YzxNwRWF3YEjniqkANcLw==)

<hr>

<h1 align="center" style="font-family:'Lato',Calibri,serif;font-size:3em">Additional</h1>

####1. [One Page Resume]({{site.url}}/assets/extradocs/Resume.pdf)

<!-- ####2. [A Longer CV](#) -->

####2. [A More Visual CV]({{site.url}}/assets/extradocs/Visual_CV.pdf)


<hr>

<h1 align="center" style="font-family:'Lato',Calibri,serif;font-size:3em">Recess</h1>

####Here's a list of things I like to do when I'm free:

####**TV Series/Anime** : I'm currently catching up on [Fargo](http://www.imdb.com/title/tt2802850/). Recently finished [Stein's Gate](http://www.imdb.com/title/tt1910272/), [Fullmetal Alchemist Brotherhood](http://www.imdb.com/title/tt1355642/) and [Tokyo Ghoul](http://www.imdb.com/title/tt3741634/).

####**Munch!Munch!** : I **love** eating: be it *Chinese*,*Italian*,*Mexican* or *Indian*! 

####**Gaming** : Big fan of challenging First Person Shooters and RPGs. Currently binging on [Bloodborne](http://www.ign.com/games/bloodborne/ps4-20017456). When younger I had planned on being a Game Developer; didn't work out though :(

####**Ideating** : By ideating, I mean reading, listening or practicing  things I can learn from. Most of this ideating comes from **[Reddit](https://www.reddit.com/r/machinelearning)**,**[TheTalkingMachines](http://www.thetalkingmachines.com/)**, Kaggle's [competitions](http://kaggle.com/) and [No Free Hunch](http://blog.kaggle.com/) 

<hr>