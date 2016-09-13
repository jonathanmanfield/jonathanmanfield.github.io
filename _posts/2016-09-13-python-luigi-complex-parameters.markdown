---
title:  "Complex Parameter types for Python Luigi"
date:   2016-09-13 16:09:00
description: Here, we demonstrate using lists and dictionaries as parameters for our Data Pipeline that detect anomalous numbers found in a directory of files.
---

Let me begin with a quick note. Python Luigi is a great library for building Data Pipelines, which are extremely useful for any data analysis process which is manipulating data step-by-step. You can find a great explanation of the motivations and how to get started on Marco Bonzanini's [blog][bonza-pipe].

# Problem

For the purpose of this post, consider a simple task. We have a collection of files, each contain a single number. The aim is to find the set of numbers which are considered to be anomalous, in terms of their value, with respect to the collection of numbers. 

This a simplified version of a challenge many organisations actually face. For example, these numbers could represent; load across a network of servers, or the inventory values of vehicles in a fleet of amored cars. In these cases, employing a basic measure of Anomaly Detection could be useful for re-distribution of resources or activation of an early warning system, respectively.

In my case, I've recently been working with news datasets, so these numbers characterise reports from journalist. In future blog posts, I plan to discuss this in great detail.

# Solution

My approach is to:

1. Read the number in each file and store it
2. Examine all stored data and identify outliers

The step-by-step layout makes Python Luigi a natural solution. We can setup up tasks for the two steps in such a way, that the first step is always completed prior to the second step. If you referred to the earlier blog post explaining Luigi these ideas will be basic, so I will skip straight to paremeterisation.

## Parameterisation 

The motivation for the organisation could be to examine a subset of their entire operation and adjust the sensitivity of their Anomaly Detection method accordingly.

So in our case, we want to pass a list of names of the files we want to examine, and parameters for our [statisical threshold model][in-quest] to detect outliers. We use an interquartile range based method. Where $$D$$, $$U$$ are specificed lower and upper percentiles, $$IQR$$ is the interquartile range and $$K$$ is a sensitivity parameter, the set of outliers is given as any values outside of the closed range:

$$[Q_D-K\cdot IQR, Q_U+K\cdot IQR]$$

So, ideally we could use a list for the file names and a dictionary for the method parameters and pass these to Python Luigi. As I was originally attempting to accomplish this in a separate piece of work, I followed a GitHub [issue][gh-issue] that led me to believe that this required the implementation of a subclass of the Luigi's Parameter class. The [documentation][luigi-param-doc] was not helpful either.

Eventually, I settled with a solution of passing string representations of these data structures to the pipeline as command-line arguments and evaluating them using a system library. Messy.

## Implementation

As it turns out, the [source code][param-source] for Luigi's Parameter class includes several extra Parameter types and some useful examples. List and Dictionary are actually built-in.

{% highlight python %}
fn = luigi.ListParameter(default=[1,2,3,4,5,6,8,9,10])
p = luigi.DictParameter(default={"u": 75, "d": 25, "k": 1.5})
{% endhighlight %}

When passing the command line arguments, remember to pass them as a string. 

{% highlight shell %}
$ python pipe.py ReadData --local-scheduler --fn '[1,3,5]'
{% endhighlight %}

Also, when using values from a dictionary parameter as part of a filename, it is better to use the actual values not the entire formatted data structure. This avoids saving just the type of the data structure in the filename.

{% highlight python %}
def output(self):
  return luigi.LocalTarget("detect_outliers_fn_{}_u_{}_d_{}_k_{}.txt".format(self.fn, self.p["u"], self.p["d"], self.p["k"]))
{% endhighlight %}

# Code

In case it might be useful, I've created a [gist][gh-gist] containing code to generate the data and the pipeline.

[bonza-pipe]: https://marcobonzanini.com/2015/10/24/building-data-pipelines-with-python-and-luigi/
[in-quest]: http://arxiv.org/abs/1508.03981
[gh-issue]: https://github.com/spotify/luigi/issues/211
[param-source]: https://github.com/spotify/luigi/blob/master/luigi/parameter.py
[luigi-param-doc]: http://luigi.readthedocs.io/en/stable/parameters.html#parameter-types
[gh-gist]: https://gist.github.com/jonathanmanfield/1767fa19570f7a0188de08b6362e8437
