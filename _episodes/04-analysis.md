---
title: "Analysing performance"
teaching: 30
exercises: 10
questions:
- "How do I analyse the results of my benchmarking?"
- "Which metrics can I use to make decisions about the best way to run my calculations?"
- "What are the aspects I should consider when making decisions about parallel performance?"
- "What stops my HPC use from scaling to higher node counts?"
objectives:
- "Understand how to analyse and understand the performance of your HPC use."
- "Understand speedup and parallel efficiency metrics and how they can support decisions."
- "Have an awareness of the limits of parallel scaling: Amdahl's Law and Gustavson's Law."
keypoints:
- "You can use benchmarking data to make decisions on how to best use your HPC resources."
- "Parallel efficiency is often the key decision metric for limits of parallel scaling."
- "Parallel performance is ultimately limited by the serial parts of the calculation."
---

In the last section we talked about performance metrics and how to gather benchmark
data and used an simple example program as an example. In this section we will look
at how to analyse the data we have gathered, start to understand performance and 
how to use the analysis to make decisions.

## Plotting performance

(TODO: change in light of change to previous section to use VisiData to aggregate
values)

To get a quick view of how the performance varies as we increase the number of 
MPI processes (equivalent to cores in our case) we are going to use the
[VisiData tool](https://www.visidata.org/) which allows us to manipulate and
visualise tabular data in the terminal.

As well as printing the timing and performance data to the screen, the 
`sharpen-perf.py` program also produces two files with the data in 
CSV (comma-separated value) format that we can use with VisiData. Let's load
the performance data into VisiData:

```
module load cray-python
module load visidata
vd perf.csv
```
{: .language-bas}
```

 Cores | CalcMin | CalcMean | CalcMax | OverallMin | OverallMean | OverallMax ║
 1     | 0.151   | 0.152    | 0.153   | 0.136      | 0.139       | 0.140      ║
 2     | 0.300   | 0.302    | 0.303   | 0.245      | 0.254       | 0.258      ║
 4     | 0.597   | 0.600    | 0.606   | 0.441      | 0.444       | 0.448      ║
 8     | 1.206   | 1.207    | 1.210   | 0.703      | 0.704       | 0.706      ║
 16    | 2.322   | 2.373    | 2.399   | 0.950      | 0.960       | 0.965      ║
 32    | 4.620   | 4.653    | 4.670   | 1.193      | 1.196       | 1.200      ║
 64    | 8.194   | 8.194    | 8.194   | 1.328      | 1.336       | 1.349      ║
 128   | 14.975  | 14.975   | 14.975  | 1.143      | 1.290       | 1.383      ║
 256   | 27.142  | 27.142   | 27.142  | 1.383      | 1.396       | 1.405      ║

1› perf| user_macros | saul.pw/VisiData v2.1 | opening perf.            9 rows 
```
{: .output}

Your terminal will now show a spreadsheet interface with the performance data from
your benchmark runs. You can navigate between different cells using the arrow 
keys on your keyboard or by using your mouse. 

VisiData also allows us to show scatter plots in the 
terminal to get a quick visual representation of the relationship between the
data in different columns. To do this, we need to make sure the columns are 
set to the correct numerical data types and say which one is going to be the 
x-axis.

Select the "Cores" column and hit `#` to set it as integer data, 
next select the "CalcMax" column and hit `%` to set it as floating point
data, finally select the "OverallMax" column and hit `%` to set it as 
floating point data too. Your terminal should now look something like:

```
 Cores#| CalcMin | CalcMean | CalcMax%| OverallMin | OverallMean | OverallMax%║
     1 | 0.151   | 0.152    |    0.15 | 0.136      | 0.139       |       0.14 ║
     2 | 0.300   | 0.302    |    0.30 | 0.245      | 0.254       |       0.26 ║
     4 | 0.597   | 0.600    |    0.61 | 0.441      | 0.444       |       0.45 ║
     8 | 1.206   | 1.207    |    1.21 | 0.703      | 0.704       |       0.71 ║
    16 | 2.322   | 2.373    |    2.40 | 0.950      | 0.960       |       0.96 ║
    32 | 4.620   | 4.653    |    4.67 | 1.193      | 1.196       |       1.20 ║
    64 | 8.194   | 8.194    |    8.19 | 1.328      | 1.336       |       1.35 ║
   128 | 14.975  | 14.975   |   14.97 | 1.143      | 1.290       |       1.38 ║
   256 | 27.142  | 27.142   |   27.14 | 1.383      | 1.396       |       1.41 ║

1› perf|                                          %  type-float         9 rows 

```
{: .output}

Now we will set the "Cores" column as the x-axis variable. Select the "Cores" 
column and hit `!` - it should change colour (it is now blue in my terminal)
to show it has been set as the catagorical data.

Finally, we can plot the graph by hitting `g.` (`g` followed by a period). You
should see something like the plot shown below which shows the variation of 
maximum performance for the calculation part of Sharpen and the overall 
performance of Sharpen (calculation and IO).

```
27.14                                                             1:CalcMax ⠁
                                                                  2:OverallMax



20.39



                                           ⡀
13.64




                          ⠐
6.89

                  ⠁

              ⠁           ⢀                ⡀                                ⡀
0.14      ⢠⠆⠃ ⠁   ⠁
Cores»    1               64               128             192              256
2› perf_graph| loading data points | loaded 18 points (0  g.          18 plots 

```
{: .output}

> ## What does this show?
> You can see that the performance of the calculation part and the overall 
> performance look different. Can you come up with any explanation for the
> difference in performance?
{: .challenge}

## Analysis

We are going to analyse our benchmark data for the Sharpen example by looking at
two metrics that we introduced earlier:

 - *Speedup*: The ratio of performance at a particular core count relative to our
   baseline performance.
 - *Parallel efficiency*: The percentage of perfect speedup that we observe at a
 - particular core count.

> ## Compute the speedup for the maximum calculation and overall performance
> Using the data you have, compute the speedup for the maximum calculation and
> maximum overall performance. You can do this by hand, by using VisiData,
> by downloading the CSV and using a spreadsheet software on your laptop 
> (e.g. Excel) or by writing a script. Remember that the speedup is the ratio
> of performance to the baseline performance.
> 
> You should also add a column with the values for perfect speedup. As our 
> baseline performance is measured on a single core, the perfect speedup is
> actually just the same as the number of cores in this case.
> 
> Finally, can you produce a plot showing the calculation speedup, the overall
> speedup and the perfect speedup?
{: .challenge}

> ## Speedup and performance
> If plotted, the speedup should show the same shape as the performance plot we
> produced earlier with the difference that it will start at 1  for a single
> core (where the baseline and the observed performance are identical). Given this,
> Why do we use the two different metrics in our analysis? In short, we do not
> have to, but it can be more convenient to work with speedup rather than performance
> when comparing to ideal parallel performance and it is also a useful step
> towards computing the parallel efficiency which is often a key dimensionless
> decision-making metric.
> 
> Plotting performance directly is generally better for showing the differences
> between different setups (e.g. different HPC systems, different software versions)
> as it avoids the choice of baseline that must be made when plotting speedup and
> gives a better indication of the absolute performance differences between the
> different setups.
{: .callout}

Now we have columns with the calculated speedups and perfect speedup we can use these
to compute the parallel efficiency.

> ## Calculating the parallel efficiency
> Add columns to your spreadsheet (or output if you wrote a script) that show the
> calculation parallel efficiency and the overall parallel efficiency.
> Remember, the parallel efficiency is the ratio of the observed speedup to the 
> perfect speedup.
> 
> Given the parallel efficiencies you have calculated can you propose a maximum
> core count that can be usefully used for this Sharpen example - both in terms
> of overall and in terms of just the calculation part? Why did you choose these
> values?
> > ## Solution
> > (TODO: Add example results.)
> >
> > 8 cores for the overall speedup (63% parallel efficiency) is the limit of 
> > useful scaling and 256 cores for the calculation speedup (69% parallel
> > efficiency) is the limit of useful scaling from the values measured.
> >
> > Anything less than 70% parallel efficiency is generally considered poor
> > scaling and less than 60% parallel efficiency is generally considered not
> > a reasonable use of compute time in normal workflow production. Of course,
> > if the time to solution is critical (for example, responding to an urgent
> > request for data) then it may be acceptable to run with very poor parallel
> > efficiency to achieve a faster time to solution but it should be carefully
> > considered if the compute cycles could be better used by running multiple
> > calculations in parallel rather than the inefficient use of resources by 
> > scaling the single calculation beyond its limits. In this case, the overall
> > speedup is not improving at all beyond 64 cores so using more than this is
> > futile anyway.
> {: .solution}
{: .challenge}


{% include links.md %}
