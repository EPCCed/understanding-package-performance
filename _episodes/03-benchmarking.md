---
title: "HPC performance and benchmarking"
teaching: 30
exercises: 10
questions:
- "Why should I benchmark my use of HPC?"
- "What are the key benchmarking concepts that I should understand?"
- "What is the right performance metric for my HPC use?"
- "What parameters can affect the performance of my applications?"
objectives:
- "Understand how benchmarking can improve my use of HPC resources."
- "Understand key benchmarking concepts and why they are useful for me."
- "Be able to identify the correct performance metric for my HPC use."
keypoints:
- ""
---

Having looked at workflow components in general we will now move on to look
at the specifics of understanding the HPC software component of your workflow
to allow you to plan and use your HPC resources more efficiently.

The main tool we are going to use to understand the performance during this
course, is *benchmarking*.

## What is benchmarking?

Benchmarking is measuring how the performance of something varies as you change
parameters. In our case, we are benchmarking parallel software on HPC systems
and so the parameters we will measure performance variation against are usually:

 - The number of parallel (usually MPI) processes we use
 - The number of threads (usually OpenMP threads) we use
 - The distribution of processes/threads across compute nodes
 - Calculation input parameters that affect performance

We often want to explore multiple parameters in our benchmarking to get an idea
of how performance varies. Needless to say, the search space can become very 
large!

## Why use benchmarking?

Benchmarking your use of software on HPC resources is potentially useful for
a range of reasons. These could include:

 - Understanding how your current calculations scale to different node/core counts so
   you can choose the most appropriate setup for your work
 - Understanding how your potential future work scales to allow you to request
   the correct amount of resource in applications for resources

Benchmarking is also commonly used in purchasing new HPC systems to make sure
that the new system gives the right level of performance for users. However, you
are unlikely to be purchasing your own HPC system so we will not discuss this
scenario further here!

## Key benchmarking terminology and concepts

We will use a number of different terms and concepts throughout our discussion on
benchmarking so we will define them first:

 - **Performance**: The measure of how well an application is running. Performance
   is always measured as a rate. The actual unit of performance depends on the 
   application, some common examples are ns/day, iterations/s, simulated years
   per day, SCF cycles per second. The actual measure you use is the 
   *performance metric*.
 - **Baseline performance**: Most benchmarking uses a baseline performance to 
   measure performance improvement against. In the benchmarking we do here, this
   will usually be the performance on the smallest number of nodes (often 1 node).
 - **Scaling**: A measure of how the performance changes as the number of 
   nodes/cores are increased. Scaling is measured relative to the
   *baseline performance*. *Perfect scaling* is the performance you would expect
   if there was no parallel overheads in the calculation.
 - **Parallel efficiency**: The ratio of measured scaling to the perfect scaling.

> ## Your application use an benchmarking
> Think about your use of HPC, for an HPC application you use try to identify
> the performance metric you might use and 

Other key points:

 - Plan the benchmark runs you want to perform - what are you planning to measure
   and why?
 - Benchmark performance should be measured multiple times to assess variability.
   Three individual runs are usually considered the minimum but five is better.
   (TODO: mention how you do and do not combine multiple runs.)
 - You should try to capture all of the relevant information on the run as most
   HPC software do not always record these. Details that may not be recorded in the 
   software output may include: environment variables, process/thread distribution,
   We will talk about how automation can help with this later in the course.
 - Organise your output data so that you know which output corresponds to which
   runs in your benchmark set.

> ## Exploring the performance of GROMACS: part 1
> We are going to use the GROMACS molecular dynamics software with a standard input
> set to explore benchmarking in practice. If you want to explore a piece of software
> you use in your research with your own input case, then please feel free to 
> do this.
>
> The first step is to understand what performance metric you will use and
> plan the initial set of benchmark runs. For GROMACS, the performance metric
> we will use is ns/day as this is conveniently reported by the GROMACS software.
>
> Plan an initial set of benchmark runs to explore how the performance
> of GROMACS changes as you increase the number of nodes?
>
> Run the the benchmark runs and record the performance.
>
> > ## Solution
> > A good initial set of runs would be 1, 2, 4, 8 and 16 nodes with three runs
> > of each of them. Performance will be measured as the maximum performance from
> > the three individual runs.
> >
> > Example benchmarking data (TODO: add example results)
> > 
> > - 1 node: result 1, result 2, result 3
{: .challenge}





{% include links.md %}
