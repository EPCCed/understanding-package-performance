---
title: "Automating performance data collection and analysis"
teaching: 30
exercises: 10
questions:
- "Why is automation useful?"
- "What considerations should I have when automating collecting performance data?"
- "Are there any tools that can help with automation?"
objectives:
- "Understand how automation can help with performance measurements."
- "See how different automation approaches work in practice."
keypoints:
- ""
---

We now know how to collect basic benchmark performance data and how to analyse it
so that we can understand the performance and make decisions based on it. We will
now look at techniques for making data collection and analysis more efficient.

## Why automate?

The manual collection of benchmark data can become a onerous, time-consuming task,
particularly if you have a large parameter space to explore. When you have more 
than a small number of benchmark runs to undertake it is often worthwhile automating
the collection of benchmarking data.

In addition to reducing the manual work, automation of data collection has a number
of other advantages:
 - It can make the collection of data more systematic across different parameters
 - It can make the recording of details more consistent
 - It can simplify the automation of analysis

## Automation approaches

There are a number of different potential approaches to automation: ranging from
using a existing framework to creating your own scripts. The approach you choose
generally depends on how many benchmark runs you need to perform and how much
data you plan to collect. The larger the dataset you are interested in, the more
time it will be worth investing in automation.

## Automating Sharpen benchmarking

In this lesson, we will look at writing our own simple scripts to automate the
collection of benchmark data for the Sharpen application to repeat the work we
have already done by hand to demonstrate one potential approach.

First, we need to decide what parameters are varying and if there are any data
we need to capture from the runs that will not be captured in the output from the
application itself.

Things we need to vary:
 - Number of cores
 - Run number at this data point

Things we need to capture that are not in the application output:
 - Date and time of the run
 - Run number at this data point

We have already seen how to use `srun` directly to run these benchmarks, but in
this case we will generate job submission scripts to run the benchmarks. This is
not strictly needed in this case but will demonstrate an approach that is more 
flexible for your potential use cases than submitting using `srun` directly.

### Step 1: Write the template job submission script

### Step 2: Identify parameters that will vary

### Step 3: Write the automation script

### Step 4: Run the benchmark process

## More complex automation: potential frameworks

- ReFrame
- JUBE

{% include links.md %}