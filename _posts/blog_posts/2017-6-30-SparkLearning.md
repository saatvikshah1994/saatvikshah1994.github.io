---
layout: post
title: ! "Apache Spark: A Learning Path"
image:
  feature: /spark/header.jpg 
description: "Learning Path for Apache Spark"
tags: Spark
category: blog-post
---

<h2 id='intro'>Background</h2>
Someone at the office requested me for good material to learn about Spark in-depth. In the process, I setup most of the documentation included in this post. Most resources are based on **PySpark**.
 
<h2 id="outline">Outline</h2>
<li><a href="#hadoop">The Prerequisites: Hadoop and MapReduce</a></li>
<li><a href="#sparkbasics">The Basics of Apache Spark</a></li>
<li><a href="#sparkadvanced">Advanced Spark: Tuning and Internals</a></li>

<h2 id="hadoop">The Prerequisites: Hadoop and MapReduce</h2>

<figure>
    <center><a href="{{ site.url }}/images/spark/hadoop_mapred.jpg"><img src="{{ site.url }}/images/spark/hadoop_mapred.jpg" alt="" height="600px" width="800px" align="middle" /></a></center>
</figure>


1. **Why?** :  Spark is the **successor** to MapReduce and essentially uses a combination of multiple map-reduce operations internally. Its thus important to learn about MapReduce to get a good grasp over its internals. Hadoop and HDFS is still very much in existence and is very popular with Spark users. 
2. **Understanding Hadoop and HDFS**([article](http://www.thegeekstuff.com/2012/01/hadoop-hdfs-mapreduce-intro/),[video](https://www.youtube.com/watch?v=8bQBnBTGv0A))
3. **Understanding MapReduce**
	1. What is Map and Reduce in MapReduce?([SO link](https://stackoverflow.com/questions/28982/simple-explanation-of-mapreduce))
	2. MapReduce Explained in detail([article](http://ksat.me/map-reduce-a-really-simple-introduction-kloudo/),[video](https://www.youtube.com/watch?v=bcjSe0xCHbE))
4. **Exercises**: Its best to write up a few simple to complex programs to get a good idea of how MapReduce works. These can work outside of a cluster environment, but it'd be great to practice in a multinode cluster environment because that is where you face a number of issues.<br> 
*Environment*: Python 2.7 with [MRJob](https://github.com/Yelp/mrjob)(Use pip) <br>
*Datasets*: I used [UCI ML Repository](http://archive.ics.uci.edu/ml/) datasets(such as Iris when learning)
	1. Word-Count
	2. K Nearest Neighbours classifier
	3. Random Forest classifier(Slightly advanced - feel free to use scikit-learn for Decision Trees)
5. **Related Advanced Topics**
	1. The Small File Problem([article](http://inquidia.com/news-and-info/working-small-files-hadoop-part-1))
	2. YARN in depth([article](https://blog.cloudera.com/blog/2015/09/untangling-apache-hadoop-yarn-part-1/))

<h2 id="sparkbasics">The Basics of Apache Spark</h2>

<figure>
    <center><a href="{{ site.url }}/images/spark/spark_logo.png"><img src="{{ site.url }}/images/spark/spark_logo.png" alt="" height="500px" width="500px" align="middle" /></a></center>
</figure>

1. **Overview**: Spark is essentially a beast of a system hidden under a lot of abstraction. Before approaching the internals it would be a good idea to experiment with and understand the following: **RDDs, DataFrames, Transformations, Actions, Spark I/O, UDFs, Partitioning, Caching**
2. **Books and Literature**
	1. [Learning Spark](https://www.amazon.com/Learning-Spark-Lightning-Fast-Data-Analysis-ebook/dp/B00SW0TY8O/ref=sr_1_2?s=digital-text&ie=UTF8&qid=1498696916&sr=1-2&keywords=mastering+apache+spark): I don't think it covers Spark 2.0 components well.
	2. [Mastering Apache Spark 2](https://jaceklaskowski.gitbooks.io/mastering-apache-spark/content/): I use this book often for reference.
	3. *Blogs/Documentation/Stack Overflow*: These are generally most updated with the latest.
	4. [Original RDD Research paper](https://www2.eecs.berkeley.edu/Pubs/TechRpts/2011/EECS-2011-82.pdf): Must Read!
3. **Environment**: Miniconda Virtualenv with PySpark and Jupyter notebooks. Its a good idea to learn in an interactive notebook as there will be a lot of trial and error.
4. **Learning**: 
	1. Read the original RDD research paper
	2. Week 1 of this Coursera course([link](https://www.coursera.org/learn/scala-spark-big-data))
	3. Basics of RDDs, Dataframes and Datasets([article](https://databricks.com/blog/2016/07/14/a-tale-of-three-apache-spark-apis-rdds-dataframes-and-datasets.html)) 
	4. Transformations vs. Actions([blog post](https://www.analyticsvidhya.com/blog/2016/10/using-pyspark-to-perform-transformations-and-actions-on-rdd/))
	5. IPython notebooks on Spark basics([link](https://github.com/bradenrc/Spark_POT/tree/master/Modules))
	6. Partitions([article1](https://jaceklaskowski.gitbooks.io/mastering-apache-spark/spark-rdd-partitions.html),[article2](https://databricks.gitbooks.io/databricks-spark-knowledge-base/content/performance_optimization/how_many_partitions_does_an_rdd_have.html))
	7. Caching([article](https://jaceklaskowski.gitbooks.io/mastering-apache-spark/spark-rdd-caching.html) and videos from Week 1 of above Coursera course)
	8. Checkpointing([article1](https://jaceklaskowski.gitbooks.io/mastering-apache-spark/spark-rdd-checkpointing.html),[article2](https://github.com/JerryLead/SparkInternals/blob/master/markdown/english/6-CacheAndCheckpoint.md)): Article 2 is slightly advanced
	9. Pyspark ML Pipelines([official docs](https://spark.apache.org/docs/latest/ml-pipeline.html)): ML not MLLib.
5. **Exercises** 
The best exercise to do would be to pick up 1 or 2 problems on [Kaggle](https://www.kaggle.com) such as the Titanic dataset and experiment with loading the data onto a DataFrame and run a ML pipeline on it. Be sure to apply Cross Validation to evaluate your results. Pipelines in Spark are slightly different than scikit-learn - Be sure to use them. This can be done with Spark standalone - local mode and is enough to learn all of the above basics.

6. **Extras**
The thing with Spark is that its evolving in leaps and bounds. Most hardcover books are generally outdated as a result. There's two additional courses on Spark, I've heard excellent things about:
	1. *Big Data Analysis with Apache Spark* from UCBerkeley([course link](https://www.edx.org/course/big-data-analysis-apache-spark-uc-berkeleyx-cs110x)): Amongst top 50 online courses according to Class Central. Uses Pyspark.
	2. *Big Data Analysis with Scala and Spark* from EPFL(Scala powerhouse)([course link](https://www.coursera.org/learn/scala-spark-big-data)): Needs a good understanding of Scala. The concepts are covered very well.

<h2 id="sparkadvanced">Advanced Spark: Tuning and Internals</h2>

<figure>
    <center><a href="{{ site.url }}/images/spark/spark_advanced.jpg"><img src="{{ site.url }}/images/spark/spark_advanced.jpg" alt="" height="500px" width="500px" align="middle" /></a></center>
</figure>

This section will cover more advanced concepts of Spark: mainly **Tuning Guides** and **Spark internals**. **Strongly recommended** to be working with a cluster at this stage - either inhouse or provided by Amazon/Google cloud services. I'm currently in this stage myself, so will update the guide as I learn more.

1. **Why learn this?** Tuning spark has only got easier with improvements such as dynamic allocation and Project Tungsten. It is still important to understand as, as peculiar use cases always show up, and clusters dont always behave optimally in dynamic allocation.
2. **Spark Official Tuning Guide**([documentation](https://spark.apache.org/docs/latest/tuning.html)): Concise with great content. Must Read.
3. **Cloudera Tuning Guide**([article](https://blog.cloudera.com/blog/2015/03/how-to-tune-your-apache-spark-jobs-part-1/)): The concepts from this article might be slightly dated, mainly because a lot is handled automatically by dynamic allocation. It is still an excellent post to understand about individual components of the cluster, and how they can be tuned for optimal results. 
4. **Spark Internals in depth**([github docs](https://github.com/JerryLead/SparkInternals)): Hover to the contents tab - This material is advanced and should be approached only after finishing everything else.
5. **Big Data Cluster computing in Production**([book](https://www.amazon.com/Spark-Data-Cluster-Computing-Production/dp/1119254019)): As the name suggests, its a book meant to understand spark tuning and configuration to go into production level applications. I'm currently reading this and its pretty good at the start at least.

*Feel free to leave your comments or doubts below, or you can email me at saatvikshah1994@gmail.com.*