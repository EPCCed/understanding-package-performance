---
title: "HPC performance and benchmarking"
teaching: 30
exercises: 25
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
- "Different timing and performance metrics are used for different applications."
- "Use the lowest node/core count that is feasible for your baseline."
- "Plan your benchmarking before you start, make sure you understand which parameters you want to vary and why."
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
> Remember, it is not just the software package you are benchmarking - it is the combination
> of the software package and the input data that constitute the benchmark case. Throughout
> this course we will refer to this combination of the software and the input as the
> *application*.
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
   measure performance improvement against. In HPC benchmarking, this
   will usually be the performance on the smallest number of nodes. (For the
   extremely simple application we are going to look at, we will actually use
   a single core but this is often not possible for most real HPC applications.)
 - **Scaling**: A measure of how the performance changes as the number of 
   nodes/cores are increased. Scaling is measured relative to the
   *baseline performance*. *Perfect scaling* is the performance you would expect
   if there was no parallel overheads in the calculation.
 - **Parallel efficiency**: The ratio of measured scaling to the perfect scaling.

> ## Your application use an benchmarking
> Think about your use of HPC. For an HPC application you use, try to identify
> the timings and performance metric you might use when benchmarking. Why do
> you think the metrics you have chosen are the correct ones for this case?
{: .challenge}

## Practical considerations

 - Plan the benchmark runs you want to perform - what are you planning to measure
   and why?
   + Remember, you can often vary both input parameters to the application
     and the parallel distribution on the HPC system itself.
   + If you vary multiple parameters at once it can become difficult to interpret
     the data so you often want to vary one at at time (e.g. number of MPI processes).
 - Benchmark performance should be measured multiple times to assess variability.
   Three individual runs are usually considered the minimum but more are better.
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
application.

To do this, we will run the image sharpening program on different numbers of MPI
processes for the same input to look at how well its performance scales.

### Initial setup

Log into ARCHER2, if you are not already logged in, and load the `training/sharpen`
module to gain access to the software and input data:

```
module load training/sharpen
```
{: .language-bash}

Once this is done, move to your /work directory, create a sub-directory to 
contain our benchmarking results and move into it (remember to replace 
`t001` with the correct project code for your course and `auser` with your
username on ARCHER2).

> ## Only work file system is visible on the compute nodes
> Remember, the work file system is the only one available on the ARCHER2
> compute nodes. All just should be launched from a directory on the work
> file system to ensure they run correctly.
{: .callout}

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

For this small example, we are going to use a run on a single core of a compute node as
our baseline.

> ## Baseline size
> Remember that for real parallel applications, it will
> often not be possible to use a single core or even a single node as 
> your baseline (due to memory requirements or fitting the run within
> a reasonable runtime). Nevertheless, you should try and use the smallest size
> that you feasibly can for your baseline.
{: .callout}

Run the single core calculation on an ARCHER2 compute node with:

```
srun {{ site.workshop_srun_options }} --nodes=1 --ntasks-per-node=1 --time=0:10:0  sharpen-mpi.x > sharpen_1core_001.out
```
{: .language-bash}
```
srun: job 62318 queued and waiting for resources
srun: job 62318 has been allocated resources
```
{: .output}

Using `srun` in this way launches the application on a compute nodes with the 
specified resources.

This line is quite long and is going to be tedious to type out each time we
want to run a calculation so we will setup a command alias with the options
that will not change each time we run to make things easier:

```
alias srunopt="srun {{ site.workshop_srun_options }}"
```
{: .language-bash}

> ## Making the alias permanent
> If you want this alias to persist and be available each time you log into ARCHER2
> then you can add the `alias` command above to the end of your `~/.bashrc` file on
> ARCHER2.
{: .callout}

Now we can run the baseline calculation again with:

```
srunopt --time=0:10:0 --nodes=1 --ntasks-per-node=1 sharpen-mpi.x > sharpen_1core_002.out
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
 - *Overall time (in s)* and *Overall performance (in Mpixels/s)*: computed as the
   overall time and divided by the image size in Mpixels.

So, for the output above:

 - Image size in Mpixels = (564 * 770) / 1,000,000 = 0.43428 Mpixels
 - Calculation time = 2.882s
 - Overall time = 3.182s
 - Calculation performance = 0.43428 / 2.882 = 0.151 Mpixels/s
 - Overall performance = 0.43428 / 3.182 = 0.136 Mpixels/s

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
   + Note that you should not generally combine rate metric results using the 
     arithmetic mean as this can lead to incorrect conclusions. It is better
     to combine results using the timings and the convert this result into the
     rate. (If you need to combine rate metrics, you can use the harmonic
     mean rather than the arithmetic mean.) See
     [Scientific Benchmarking of Parallel Computing Systems](https://spcl.inf.ethz.ch/Teaching/2019-dphpc/hoefler-scientific-benchmarking.pdf)
     for more information on how to report performance data.)
 - You want an idea of the best case scenario to allow you to compare the
   performance of different HPC systems or parameter choices. In this case,
   you will likely want to take the *best* performance at each of your 
   relevant values (core counts for our Sharpening example).
 - You want an idea of the performance variation. In this case, you will 
   likely look at the differences between the best and worst performance
   values, maybe as a percentage of the mean performance.

In this case, we are interested in the change in performance as we change the
number of MPI processes (or cores used) so we will use the minimum timing value
from the multiple runs (as this corresponds to the maximum measured performance).
You can look at this more conveniently using the `grep` command:

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

To make the process of extracting the timings and performance data 
from the sharpen output files easier for you we have written a small
Python program: `sharpen-data.py`. This program takes the extension
of the output files ("out" in our examples above), extracts the
data required to compute performance (image size and timings) and
saves them in a CSV (comma-separated values) file.

```
sharpen-data.py out
```
{: .language-bash}
```
Cores        Size      Calc   Overall
    1    0.434280     2.844     3.096
    1    0.434280     2.857     3.118
    1    0.434280     2.882     3.182
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
> `sharpen-data.py` program. If you prefer, you can write a job submission script
> to run these benchmark calculations rather than using `srun` directly.
>
> > ## Solution
> > People often use doubling of the number of MPI processes as a useful first
> > place for a set of benchmark runs. Going from 1 MPI process to 256 MPI processes
> > this gives the following runs: 2, 4, 8, 16, 32, 64, 128 and 256 MPI processes.
> >
> > ```
> > Cores        Size      Calc   Overall
> >    16    0.434280     0.181     0.450
> >     2    0.434280     1.432     1.684
> >     4    0.434280     0.727     0.982
> >   256    0.434280     0.016     0.310
> >   128    0.434280     0.029     0.314
> >    64    0.434280     0.053     0.326
> >     1    0.434280     2.844     3.096
> >   256    0.434280     0.016     0.314
> >    32    0.434280     0.093     0.362
> >     8    0.434280     0.360     0.615
> >     8    0.434280     0.360     0.618
> >   128    0.434280     0.029     0.316
> >    64    0.434280     0.053     0.327
> >    32    0.434280     0.093     0.364
> >     2    0.434280     1.446     1.770
> >     2    0.434280     1.432     1.685
> >     4    0.434280     0.727     0.984
> >    16    0.434280     0.187     0.457
> >     1    0.434280     2.857     3.118
> >    32    0.434280     0.094     0.363
> >   256    0.434280     0.016     0.309
> >     8    0.434280     0.359     0.617
> >     4    0.434280     0.717     0.970
> >    16    0.434280     0.181     0.450
> >    64    0.434280     0.053     0.322
> >     1    0.434280     2.882     3.182
> >   128    0.434280     0.029     0.380
> > ```
> > {: .output}
> {: .solution}
{: .challenge}

## Aggregating data

Next we want to aggregate the data from our multiple runs at particular core counts -
remember that, in this case, we want the minimum timing (maximum performance) from the
runs to use to compute the performance. To complete these steps we are going to make
use of the [VisiData tool](https://www.visidata.org/) which allows us to manipulate and
visualise tabular data in the terminal.

As well as printing the timing data to the screen, the `sharpen-data.py` program also
produces a file called `benchmark_runs.csv` with the data in CSV (comma-separated value)
format that we can use with VisiData. Let's load the timing data into VisiData:

```
module load cray-python
module load visidata
vd benchmark_runs.csv
```
{: .language-bash}
```
 Cores | Size     | Calc  | Overall ║
 16    | 0.434280 | 0.181 | 0.450   ║
 2     | 0.434280 | 1.432 | 1.684   ║
 4     | 0.434280 | 0.727 | 0.982   ║
 256   | 0.434280 | 0.016 | 0.310   ║
 128   | 0.434280 | 0.029 | 0.314   ║
 64    | 0.434280 | 0.053 | 0.326   ║
 1     | 0.434280 | 2.844 | 3.096   ║
 256   | 0.434280 | 0.016 | 0.314   ║
 32    | 0.434280 | 0.093 | 0.362   ║
 8     | 0.434280 | 0.360 | 0.615   ║
 8     | 0.434280 | 0.360 | 0.618   ║
 128   | 0.434280 | 0.029 | 0.316   ║
 64    | 0.434280 | 0.053 | 0.327   ║
 32    | 0.434280 | 0.093 | 0.364   ║
 2     | 0.434280 | 1.446 | 1.770   ║
 2     | 0.434280 | 1.432 | 1.685   ║
 4     | 0.434280 | 0.727 | 0.984   ║
 16    | 0.434280 | 0.187 | 0.457   ║
 1     | 0.434280 | 2.857 | 3.118   ║
 32    | 0.434280 | 0.094 | 0.363   ║
 256   | 0.434280 | 0.016 | 0.309   ║
1› benchmark_runs| user_macros | saul.pw/VisiData v2.1 | opening benchmark_runs.csv a           27 rows 
```
{: .output}

Your terminal will now show a spreadsheet interface with the timing data from
your benchmark runs. You can navigate between different cells using the arrow 
keys on your keyboard or by using your mouse. 

We are now going to use VisiData to aggregate the timing data from our runs. To
do this, we first need to let the tool know what type of numerical data is in
each of the columns.

Select the "Cores" column and hit `#` to set it as integer data, 
next select the "Size" column and hit `%` to set it as floating point
data; select the "Calc" column and hit `%` to set it as 
floating point data too; finally, select the "Overall" column and hit `%` to
set it as  floating point data. Your terminal should now look something like:

```
 Cores#| Size    %| Calc %| Overall%║
    16 |     0.43 |  0.18 |    0.45 ║
     2 |     0.43 |  1.43 |    1.68 ║
     4 |     0.43 |  0.73 |    0.98 ║
   256 |     0.43 |  0.02 |    0.31 ║
   128 |     0.43 |  0.03 |    0.31 ║
    64 |     0.43 |  0.05 |    0.33 ║
     1 |     0.43 |  2.84 |    3.10 ║
   256 |     0.43 |  0.02 |    0.31 ║
    32 |     0.43 |  0.09 |    0.36 ║
     8 |     0.43 |  0.36 |    0.61 ║
     8 |     0.43 |  0.36 |    0.62 ║
   128 |     0.43 |  0.03 |    0.32 ║
    64 |     0.43 |  0.05 |    0.33 ║
    32 |     0.43 |  0.09 |    0.36 ║
     2 |     0.43 |  1.45 |    1.77 ║
     2 |     0.43 |  1.43 |    1.69 ║
     4 |     0.43 |  0.73 |    0.98 ║
    16 |     0.43 |  0.19 |    0.46 ║
     1 |     0.43 |  2.86 |    3.12 ║
    32 |     0.43 |  0.09 |    0.36 ║
   256 |     0.43 |  0.02 |    0.31 ║
1› benchmark_runs|                                                         %  type-float        27 rows 
```
{: .output}

Next, we want to tell VisiData how to aggregate the data in each of the columns. Remember,
we want the minimum value of the timings from the "Calc" and "Overall" columns. Highlight
the "Calc" column and hit `+`, select `min` from the list of aggregators and press Return.
Do the same for the "Overall" column. We also want to keep the size value in our aggregated 
table so select an aggregator for that column too (min or max are fine here as every row
has the same value). Once you have set the aggregators, select the "Cores" column and 
hit "Shift+f" to perform the aggregation. You should see a new table that looks something
like:

```
 Cores#║ count♯| Size_min%| Calc_min%| Overall_min%║
    16 ║     3 |     0.43 |     0.18 |        0.45 ║
     2 ║     3 |     0.43 |     1.43 |        1.68 ║
     4 ║     3 |     0.43 |     0.72 |        0.97 ║
   256 ║     3 |     0.43 |     0.02 |        0.31 ║
   128 ║     3 |     0.43 |     0.03 |        0.31 ║
    64 ║     3 |     0.43 |     0.05 |        0.32 ║
     1 ║     3 |     0.43 |     2.84 |        3.10 ║
    32 ║     3 |     0.43 |     0.09 |        0.36 ║
     8 ║     3 |     0.43 |     0.36 |        0.61 ║

2› benchmark_runs_Cores_freq|                                                        F           9 bins 
```
{: .output}

The final detail to tidy the aggregated data up before we save it, is to sort by
increasing core count. Select the "Cores" column and hit `[` to sort ascending.

Finally, we will save this aggregated data as another CSV file. Hit "Ctrl+s" and
change the file name to `benchmark_agg.csv`). Now, you can exit VisiData by typing
`gq`. We will use this CSV file in the next section to compute the performance.

## Computing performance

Load up VisiData again with the aggregate timing data:

```
vd benchmark_agg.csv
```
{: .language-bash}
```
 Cores | count | Size_min | Calc_min | Overall_min ║
 1     | 3     | 0.43     | 2.84     | 3.10        ║
 2     | 3     | 0.43     | 1.43     | 1.68        ║
 4     | 3     | 0.43     | 0.72     | 0.97        ║
 8     | 3     | 0.43     | 0.36     | 0.61        ║
 16    | 3     | 0.43     | 0.18     | 0.45        ║
 32    | 3     | 0.43     | 0.09     | 0.36        ║
 64    | 3     | 0.43     | 0.05     | 0.32        ║
 128   | 3     | 0.43     | 0.03     | 0.31        ║
 256   | 3     | 0.43     | 0.02     | 0.31        ║

1› benchmark_agg| user_macros | saul.pw/VisiData v2.1 | opening benchmark_agg.csv as             9 rows 
```
{: .output}

Now we are going to create new columns with the performance (in Mpixels/s) which
we will use in the next section when we analyse the data.

The maximum calculation performance is computed as the size divided by the minimum 
calculation timing. We can use VisiData to compute this for us but first we need to
tell the tool what numerical data is in each column (as we did for aggregating the
data, if we had not quit VisiData, we could skip this step). So, set the Cores
column as integer data (using `#`) and the other columns as floating point data 
(using `%`).

Now, select the "Calc_min" column and hit `=`, VisiData now asks us for a formula
to use to compute a new column. Enter `Size_min / Calc_min`. This should create
a new column with the performance:

```
 Cores#| count | Size_min%| Calc_min%| Size_min/Calc_min  | Overall_min%║
     1 | 3     |     0.43 |     2.84 | 0.151408450704225…%|        3.10 ║
     2 | 3     |     0.43 |     1.43 | 0.300699300699300…%|        1.68 ║
     4 | 3     |     0.43 |     0.72 | 0.5972222222222222%|        0.97 ║
     8 | 3     |     0.43 |     0.36 | 1.1944444444444444%|        0.61 ║
    16 | 3     |     0.43 |     0.18 | 2.388888888888889 %|        0.45 ║
    32 | 3     |     0.43 |     0.09 | 4.777777777777778 %|        0.36 ║
    64 | 3     |     0.43 |     0.05 | 8.6               %|        0.32 ║
   128 | 3     |     0.43 |     0.03 | 14.333333333333334%|        0.31 ║
   256 | 3     |     0.43 |     0.02 | 21.5              %|        0.31 ║

1› benchmark_agg|                                        BUTTON1_RELEASED  release-mouse         9 rows 

```
{: .output}

Move to the new column and set it to floating point data. We can also rename
the column to be more descriptive: hit `^` and give it the name `Calc_perf_max`.

```
 Cores#| count | Size_min%| Calc_min%| Calc_perf_max     %| Overall_min%║
     1 | 3     |     0.43 |     2.84 |               0.15 |        3.10 ║
     2 | 3     |     0.43 |     1.43 |               0.30 |        1.68 ║
     4 | 3     |     0.43 |     0.72 |               0.60 |        0.97 ║
     8 | 3     |     0.43 |     0.36 |               1.19 |        0.61 ║
    16 | 3     |     0.43 |     0.18 |               2.39 |        0.45 ║
    32 | 3     |     0.43 |     0.09 |               4.78 |        0.36 ║
    64 | 3     |     0.43 |     0.05 |               8.60 |        0.32 ║
   128 | 3     |     0.43 |     0.03 |              14.33 |        0.31 ║
   256 | 3     |     0.43 |     0.02 |              21.50 |        0.31 ║

1› benchmark_agg|                                        BUTTON1_RELEASED  release-mouse         9 rows 

```
{: .output}

> ## Compute the Overall maximum performance at each core count
> Add a column called `Overall_perf_max` that contains the Overall maximum performance
> at each core count and that is formatted as floating point values.
> > ## Solution
> > Select the "Overall_min" column and hit `=`, VisiData now asks us for a formula
> > to use to compute a new column. Enter `Size_min / Overall_min`. Move to the
> > new column and hit `%` to format it as floating point values and then hit `^`
> > to rename it `Overall_perf_max`. The final table should look something like:
> > ```
> >  Cores#| count | Size_min%| Calc_min%| Calc_perf_max     %| Overall_min%| Overall_perf_max    %║
> >      1 | 3     |     0.43 |     2.84 |               0.15 |        3.10 |                 0.14 ║
> >      2 | 3     |     0.43 |     1.43 |               0.30 |        1.68 |                 0.26 ║
> >      4 | 3     |     0.43 |     0.72 |               0.60 |        0.97 |                 0.44 ║
> >      8 | 3     |     0.43 |     0.36 |               1.19 |        0.61 |                 0.70 ║
> >     16 | 3     |     0.43 |     0.18 |               2.39 |        0.45 |                 0.96 ║
> >     32 | 3     |     0.43 |     0.09 |               4.78 |        0.36 |                 1.19 ║
> >     64 | 3     |     0.43 |     0.05 |               8.60 |        0.32 |                 1.34 ║
> >    128 | 3     |     0.43 |     0.03 |              14.33 |        0.31 |                 1.39 ║
> >    256 | 3     |     0.43 |     0.02 |              21.50 |        0.31 |                 1.39 ║
> > 
> > 1› benchmark_agg| "Overall_perf_max"                                       ^  rename-col         9 rows 
> > ```
> > {: .output}
> {: .solution}
{: .challenge}

Finally, save this data in a CSV file called `benchmark_perf.csv` and exit VisiData.

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

We also used a simple example application to allow us to collect some benchmark
data on ARCHER2.

Now we have collected our benchmarking data we will turn to how to analyse the data,
understand the performance and make decisions based on it.

{% include links.md %}
