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

> ## Both program and input are important
> Remember, it is not just the application you are benchmarking - it is the combination
> of the application and the input data that constitute the benchmark case.
{: .callout}

## Key benchmarking terminology and concepts

We will use a number of different terms and concepts throughout our discussion on
benchmarking so we will define them first:

 - **Timing**: Measured timings for the application you are benchmarking. These
   timings may be the full runtime of the application or can be timings for part
   of the runtime, for example, time per iteration or time per SCF cycle.
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
> the timings and performance metric you might use when benchmarking. Why do
> you think the metrics you have chosen are the correct ones for this case?
{: .challenge}

## Practical considerations

 - Plan the benchmark runs you want to perform - what are you planning to measure
   and why?
   + Remember, you can often vary both input parameters to the program
     and the parallel distribution on the HPC system itself.
   + If you vary multiple parameters at once it can become difficult to interpret
     the data so you often want to vary one at at time (e.g. number of MPI processes).
 - Benchmark performance should be measured multiple times to assess variability.
   Three individual runs are usually considered the minimum but five is better.
   + We will discuss how to combine multiple runs properly to produce a single
     value later in the course.
 - You should try to capture all of the relevant information on the run as most
   HPC software do not record these. Details that may not be recorded in the 
   software output may include: environment variables, process/thread distribution,
   We will talk about how automation can help with this later in the course.
 - Organise your output data so that you know which output corresponds to which
   runs in your benchmark set.

## Benchmarking the image sharpening program

Now we will use a simple example HPC application to run some benchmarks and extract
*timings* and *performance* data. In the next part of this course we will look at 
how to analyse and present this data to help us interpret the performance of the 
program.

To do this, we will run the image sharpening program on different numbers of MPI
processes for the same input to look at how well its performance scales.

### Initial setup

Log into ARCHER2, if you are not already logged in, and load the `sharpen`
module to gain access to the software and input data:

```
module load training/sharpen
```
{: .language-bash}

Once this is done, move to your /work directory, create a sub-directory to 
contain our benchmarking results and move into it (remember to replace 
`t001` with the correct project code for your course and `auser` with your
username on ARCHER2).

```
cd /work/t001/t001/auser
mkdir sharpen-bench
cd sharpen-bench
```
{: .language-bash}

Copy the input data from the central location to your directory:

```
cp $SHARPEN_INPUT/fuzzy.pgm .
ls
```
{: .language-bash}
```
fuzzy.pgm
```
{: .output}

### Baseline performance

For this small example, we are going to use a run on a single core as
our baseline.

> ## Baseline size
> Remember that for real parallel applications and input cases, it will
> often not be possible to use a single core or even a single node as 
> your baseline (due to memory requirements or fitting the run within
> a reasonable runtime). Nevertheless, you should try and use the smallest size
> that you feasibly can for your baseline.
{: .callout}

Run the single core calculation with:

```
srun --cpu-bind=rank --nodes=1 --ntasks-per-node=1 --time=0:10:0 --account=t001 --partition=standard --qos=standard sharpen-mpi.x > sharpen_1core_001.out
```
{: .language-bash}
```
srun: job 62318 queued and waiting for resources
srun: job 62318 has been allocated resources
```
{: .output}

This line is quite long and is going to be tedious to type out each time we
want to run a calculation so we will setup a command alias with the options
that will not change each time we run to make things easier:

```
alias srunopt="srun --cpu-bind=rank --time=0:10:0 --account=t001 --partition=standard --qos=standard"
```
{: .language-bash}

> ## Making the alias permanent
> If you want this alias to persist and be available each time you log into ARCHER2
> then you can add the `alias` command above to the end of your `~/.bashrc` file on
> ARCHER2.
{: .callout}

Now we can run the baseline calculation again with:

```
srunopt --nodes=1 --ntasks-per-node=1 sharpen-mpi.x > sharpen_1core_002.out
```
{: .language-bash}
```
srun: job 62321 queued and waiting for resources
srun: job 62321 has been allocated resources
```
{: .output}

Run the baseline calculation one more time so that we have three separate results.

Lets take a look at the output from one of our baseline runs:

```
cat sharpen_1core_001.out
```
{: .language-bash}
```
 Image sharpening code running on  1 process(es)
 Input file is: fuzzy.pgm                       
 Image size is: 564 x 770 pixels
 
 Using a filter of size 17 x  17 pixels
 Reading image file: fuzzy.pgm                       
 ... done
 
 Starting calculation ...
Rank 0 on core 0 of node <nid001961>
 .. finished
 
 Writing output file: sharpened.pgm                   
 
 ... done
 
 Calculation time was  2.882 seconds
 Overall run time was  3.182 seconds
```
{: .output}

You can see that the output reports various parameters. In terms of timing
and performance metrics, the ones of interest to us are:

 - Image size: 564 x 770 pixels - this is the size of the image that has been 
   processed.
 - Calculation time: 2.882s - this is the time to perform the actual computation.
 - Overall run time: 3.182s - this is the total time including the calculation
   time and the setup/finalisation time (which is dominated by reading the input
   image and writing the output image).

Based on these parameters, we can propose two timing and corresponding 
performance metrics:

 - *Calculation time (in s)* and *calculation performance (in Mpixels/s)*: computed
   as calculation time divided by image size in Mpixels.
 - *IO time (in s)* and *IO performance (in Mpixels/s)*: computed by subtracting
   the calculation time from the overall time and dividing the result by the image
   size in Mpixels.

So, for the output above:

 - Image size in Mpixels = (564 * 770) / 1,000,000 = 0.43428 Mpixels
 - Calculation time = 2.882s
 - IO time = 3.182 - 2.882 = 0.3s
 - Calculation performance = 2.882 / 0.43428 = 6.636 Mpixels/s
 - IO performance = 0.3 / 0.43428 = 0.691 Mpixels/s

### Combining multiple runs

Of course, we have three sets of data for our baseline rather than just the 
single result. What is the best way to combine these to produce our final 
performance metric?

The answer depends on what you are measuring and why. Some examples:

 - You want an idea of the worst case to allow you to be conservative when
   requesting resources for future applications to make sure you do not run
   out. In this case, you likely want to use the *worst* performance at
   each of your relevant values (core counts for our Sharpening example).
 - You want an idea of what the likely amount of work you are going to
   get through with the current resources you have. In this case, you will
   likely want to take the arithmetic mean of the timings you have and convert this 
   into the performance metric.
   + Note that you should not generally calculate
     metrics and then use an arithmetic mean to combine them (TODO: add reasoning)
 - You want an idea of the best case scenario to allow you to compare the
   performance of different HPC systems or parameter choices. In this case,
   you will likely want to take the *best* performance at each of your 
   relevant values (core counts for our Sharpening example).
 - You want an idea of the performance variation. In this case, you will 
   likely look at the differences between the best and worst performance
   values, maybe as a percentage of the mean performance.

In this case, we are interested in the change in performance as we change the
number of MPI processes (or cores used) so we will use the maximum performance
(minimum timing values) from the multiple runs. You can look at this more 
conveniently using the `grep` command:

```
grep time *.out
```
{: .language-bash}
```
sharpen_1core_001.out: Calculation time was  2.882 seconds
sharpen_1core_001.out: Overall run time was  3.182 seconds
sharpen_1core_002.out: Calculation time was  2.857 seconds
sharpen_1core_002.out: Overall run time was  3.118 seconds
sharpen_1core_003.out: Calculation time was  2.844 seconds
sharpen_1core_003.out: Overall run time was  3.096 seconds
```
{: .output}

In my case, the best performance (lowest timing) was from run number 3.

(TODO: alter to create a CSV of all runs and then use VisiData to produce
the aggregates)

To make the process of extracting the timings and performance data 
from the sharpen output files easier for you we have written a small
Python program: `sharpen-perf.py`. This program takes the extension
of the output files ("out" in our examples above) and extracts and 
combines the timings and performance data. For example:

```
sharpen-perf.py out
```
{: .language-bash}
```

Timings (s):
      Calculation          IO                  
Cores    Min   Mean    Max    Min   Mean    Max
    1  2.844  2.861  2.882  0.252  0.271  0.300

Performance (Mpixel/s):
      Calculation          IO                  
Cores    Min   Mean    Max    Min   Mean    Max
    1  0.151  0.152  0.153  1.448  1.603  1.723

```
{: .output}

Now we have our baseline data. Next, we need to collect data on how the timings
and performance vary as the number of MPI processes we use for the calculation
increases. 

## Collecting benchmarking data

Building on your experience so far, the next exercise is to collect the benchmark
data we will analyse in the next section of the course.

> ## Benchmarking the performance of Sharpen
> Run a set of calculations to benchmark the performance of the `sharpen-mpi.x` 
> program with the same input up to 2 full nodes (256 cores). Make sure you keep 
> the program output in a suitable set of output files that you can use with the
> `sharpen-perf.py` program. If you prefer, you can write a job submission script
> to run these benchmark calculations rather than using `srun` directly.
>
> > ## Solution
> > People often use doubling of the number of MPI processes as a useful first
> > place for a set of benchmark runs. Going from 1 MPI process to 256 MPI processes
> > this gives the following runs: 2, 4, 8, 16, 32, 64, 128 and 256 MPI processes.
> >
> > ```
> > 
> > Timings (s):
> >       Calculation          IO                  
> > Cores    Min   Mean    Max    Min   Mean    Max
> >     1  2.844  2.861  2.882  0.252  0.271  0.300
> >     2  1.432  1.437  1.446  0.252  0.276  0.324
> >     4  0.717  0.724  0.727  0.253  0.255  0.257
> >     8  0.359  0.360  0.360  0.255  0.257  0.258
> >    16  0.181  0.183  0.187  0.269  0.269  0.270
> >    32  0.093  0.093  0.094  0.269  0.270  0.271
> >    64  0.053  0.053  0.053  0.269  0.272  0.274
> >   128  0.029  0.029  0.029  0.285  0.308  0.351
> >   256  0.016  0.016  0.016  0.293  0.295  0.298
> >
> > Performance (Mpixel/s):
> >       Calculation          IO                  
> > Cores    Min   Mean    Max    Min   Mean    Max
> >     1  0.151  0.152  0.153  1.448  1.603  1.723
> >     2  0.300  0.302  0.303  1.340  1.572  1.723
> >     4  0.597  0.600  0.606  1.690  1.703  1.717
> >     8  1.206  1.207  1.210  1.683  1.690  1.703
> >    16  2.322  2.373  2.399  1.608  1.612  1.614
> >    32  4.620  4.653  4.670  1.603  1.610  1.614
> >    64  8.194  8.194  8.194  1.585  1.597  1.614
> >   128 14.975 14.975 14.975  1.237  1.412  1.524
> >   256 27.142 27.142 27.142  1.457  1.472  1.482
> > ```
> > {: .language-bash}
> {: .solution}
{: .challenge}


> ## Why do you want to use benchmarking
> Think about your use of HPC. What would you want to get out of benchmarking? Would
> measuring the minimum, maximum, mean or performance variation be appropriate for
> what you want to do and why?
> 
> What parameters would you be interested in varying for the application you want to
> benchmark?
{: .challenge}

## Capturing run details

As for determining what the correct performance measure is (min, max, mean), the 
choice of what details of the benchmark run to capture depend on what you are 
measuring performance against and why. If the output does not include the values
you require automatically then you should ensure it is captured in some way.

Here are some concrete examples of scenarios and the information you should 
ensure is captured:

 - **Comparing performance as a function of number of cores/nodes on a particular HPC system** - In this 
   case you need to ensure that the output captures the number of cores, nodes,
   MPI processes, OpenMP threads (depending on what varies). Most applications 
   will include this information somewhere in their output.
 - **Comparing performance as input parameters change** - In this case you
   must ensure that the output captures the value of the input parameters that
   are changing. These are usually captured in the output by the application
   itself.
 - **Comparing performance of applications compiled in different ways** - You 
   may be comparing the performance of different versions of an application or
   those compiled using different compilers or with different compile-time 
   options. These differences will **not** usually be captured by the application
   itself and so should be captured by yourself in the output in some way (this
   could be by the name of the directory or files the output is in or by adding
   details to the output as part of the benchmark run.)
 - **Comparing performance across different HPC systems**- This is the most 
   difficult as there is often a lot of differences to capture across different
   HPC systems. You usually need to capture all of the details that have been
   mentioned for the previous cases along with other details such as the run time
   environment and the hardware details of the different systems.

As with all the aspects we have discussed so far, you should plan in advance the
details that need to be captured as part of your benchmarking activity and how
they will be captured so you do not need to re-run calculations because you do
not have a good record of the differences between runs.

## Summary

In this section we have discussed:

 - Basic benchmarking terminology
 - The difference between timings and performance measures (rates)
 - Selecting performance metrics
 - Running benchmark calculations and gathering data
 - Capturing information on differences between benchmark runs

We also used a simple example program to allow us to collect some benchmark
data on ARCHER2.

Now we have collected our benchmarking data we will turn to how to analyse the data,
understand the performance and make decisions based on it.

> ## Benchmarking the performance of Sharpen: part 1
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
