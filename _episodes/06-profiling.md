---
title: "Introduction to profiling"
teaching: 15
exercises: 10
questions:
- "How does profiling differ from performance benchmarking?"
- "WHat information can I get from profiling?"
- "How can I profile the application I am using?"
objectives:
- "Understand what profiling is and the type of information it can reveal."
- "Be able to profile an application without the need to build/compile."
keypoints:
- "Profiling allows to see what routines are consuming runtime in our application."
- "Profiling can also provide information on memory use and IO by our application."
- "The differences between profiles at different node/core counts can be more revealing than a single profile."
---

So far in this workshop we have mostly looked at measuring the performance variation
of applications on HPC systems. While this gives us useful information to make decisions
on how best to use (or plan to use) HPC resources it does not directly tell us *why* 
the performance varies in the way that it does. *Profiling* applications on HPC
systems attempts to provide data and information on why the application performs the
way it does and, by extension, why the performance changes as we change the parameters
of the application.

## What is profiling?

When we profile an application on an HPC system we gather data on the performance of
an application such as:

 - How much time is spent in different parts of the application
 - Memory and IO usage statistics
 - MPI (or other parallel approaches) usage statistics
 - Differences across parallel tasks (for example, to understand load imbalance)

We do not have time in this workshop to go into profiling in any sort of detail (that
would be a whole course itself!); instead, we simply want to give you an idea of
what information is available from profiling tools on HPC systems and how you can
start to use them understand performance.

## Example: understanding scaling of Sharpen calculation part

To illustrate the use of profiling we will, again, turn to our Sharpen example. Of
course, we already understand the scaling of the overall Sharpen application 
which is limited by the serial (IO) part. However, as we saw when we used large
core counts (256 cores), the calculation part of the application also starts to 
show issues with scaling. We will use the *CrayPAT-lite* performance analysis tool
on ARCHER2 to look at this and use the information it provides to try and 
understand what is happening.

> ## Other profiling tools
> Although we are using a Cray-specific profiling tool here, the information 
> provided, at least at the basic level we are going to discuss here, by different
> profiling tools is very similar. Although the output/interface look different,
> you will be looking for similar information even when you are using different
> tools.
{: .callout}

As the Sharpen application is a toy case that runs in a very short amount of 
time we have had to modify the application to use with our profiling. In particular,
we have had to created a modified application that reruns the calculation part
100's of times to increase its significance within the application so we can
see the effect on scaling in the profiles. We have created a binary in the
`sharpen` module that has increased the number of calculations and has had the
profiling functionality added for you to use. The binary is called `sharpen-mpi-profile.x`

Let's use `srun` to run this binary on 64 cores:

```
srunopt --time=0:10:0 --nodes=1 --ntasks-per-node=64 sharpen-mpi-profile.x > sharpen_64cores_profile.dat
```
{: .language-bash}
```
Submitted job
```
{: .output}

Once this has run, take a look in the output file (e.g. using `less`):

```
less sharpen_64cores_profile.dat
```
{: .language-bash}
```
#################################################################
#                                                               #
#            CrayPat-lite Performance Statistics                #
#                                                               #
#################################################################

CrayPat/X:  Version 20.10.0 Revision 7ec62de47  09/16/20 16:57:54
Experiment:                  lite  lite-samples
Number of PEs (MPI ranks):     64
Numbers of PEs per Node:       64
Numbers of Threads per PE:      1
Number of Cores per Socket:    64
Execution start time:  Wed Jan 13 17:01:14 2021
System name and speed:  nid001060  2.250 GHz (nominal)
AMD Rome       CPU  Family: 23  Model: 49  Stepping:  0
Core Performance Boost:  All 64 PEs have CPB capability



Avg Process Time:       5.72 secs             
High Memory:         5,159.8 MiBytes     80.6 MiBytes per PE
I/O Read Rate:     60.430848 MiBytes/sec      
I/O Write Rate:   241.178058 MiBytes/sec      

Notes for table 1:

  This table shows functions that have significant exclusive sample
    hits, averaged across ranks.
  For further explanation, see the "General table notes" below,
    or use:  pat_report -v -O samp_profile ...

Table 1:  Profile by Function

  Samp% |  Samp | Imb. |  Imb. | Group
        |       | Samp | Samp% |  Function=[MAX10]
        |       |      |       |   PE=HIDE
       
 100.0% | 563.0 |   -- |    -- | Total
|-------------------------------------------------
|  61.6% | 346.9 |   -- |    -- | USER
||------------------------------------------------
||  34.7% | 195.5 | 37.5 | 16.3% | filter_
||  26.9% | 151.4 | 20.6 | 12.2% | dosharpen_
||================================================
|  32.7% | 183.9 |   -- |    -- | ETC
||------------------------------------------------
||  32.1% | 180.6 | 26.4 | 12.9% | __cray_EXP_01_
||================================================
|   5.7% |  32.2 |   -- |    -- | MPI
||------------------------------------------------
||   3.1% |  17.3 |  0.7 |  4.1% | MPI_BCAST
||   2.4% |  13.6 |  4.4 | 24.8% | MPI_BARRIER
|=================================================

...snip...

```
{: .output}

CrayPAT-lite provides a relatively large amount of information, other
performance analysis tools may provide less information but all should
provide the basic data we are going to look at below.

For looking at basic performance, we are going to look in the output
for the following information:

 - Proportion of time spent in parallel libraries/functions (e.g. MPI, OpenMP)
 - Memory use
 - I/O usage

#### Parallel libraries/functions

Often, the key piece of information we are interested in is the parallel 
overheads as these often (but not always, as we have seen for sharpen) limit
the scaling of parallel applications to greater node counts. In the output
from this run, this can be found by looking at the initial table:

```
Table 1:  Profile by Function

  Samp% |  Samp | Imb. |  Imb. | Group
        |       | Samp | Samp% |  Function=[MAX10]
        |       |      |       |   PE=HIDE
       
 100.0% | 563.0 |   -- |    -- | Total
|-------------------------------------------------
|  61.6% | 346.9 |   -- |    -- | USER
||------------------------------------------------
||  34.7% | 195.5 | 37.5 | 16.3% | filter_
||  26.9% | 151.4 | 20.6 | 12.2% | dosharpen_
||================================================
|  32.7% | 183.9 |   -- |    -- | ETC
||------------------------------------------------
||  32.1% | 180.6 | 26.4 | 12.9% | __cray_EXP_01_
||================================================
|   5.7% |  32.2 |   -- |    -- | MPI
||------------------------------------------------
||   3.1% |  17.3 |  0.7 |  4.1% | MPI_BCAST
||   2.4% |  13.6 |  4.4 | 24.8% | MPI_BARRIER
|=================================================
```
{: .output}

From this, we can see that just 5.7% of the total run time was spent in 
parallel functions (MPI in this case). This small proportion in MPI 
tells us that the scaling of the application at 64 cores is very likely
not limited by parallel overheads. 

The remaining 94.3% of the total run time was spent in functions that 
run the actual calculation.

#### Memory use

The default output from CrayPAT-lite only provides basic information 
on memory use. This can be found in the section before Table 1:

```
Avg Process Time:       5.72 secs             
High Memory:         5,159.8 MiBytes     80.6 MiBytes per PE
I/O Read Rate:     60.430848 MiBytes/sec      
I/O Write Rate:   241.178058 MiBytes/sec  
```
{: .output}

Form this, we can see that the sharpen application has very modest memory
requirements: just 5 GiB in total.

#### I/O use

The I/O use is shown in Tables 5 and 6 in the output:

```
Table 5:  File Input Stats by Filename

 Avg Read | Avg Read |   Read Rate | Number |      Avg | Bytes/ | File Name=!x/^/(proc|sys)/
 Time per |  MiBytes | MiBytes/sec |     of |    Reads |   Call |  PE=HIDE
   Reader |      per |             | Reader |      per |        | 
     Rank |   Reader |             |  Ranks |   Reader |        | 
          |     Rank |             |        |     Rank |        | 
|-----------------------------------------------------------------------------
| 0.027819 | 1.681152 |   60.430848 |      1 | 25,553.0 |  68.99 | fuzzy.pgm
|=============================================================================

Notes for table 6:

  This table show the average time and number of bytes written to each
    output file, taking the average over the number of ranks that
    wrote to the file.  It also shows the number of write operations,
    and average rates.
  For further explanation, see the "General table notes" below,
    or use:  pat_report -v -O write_stats ...

Table 6:  File Output Stats by Filename

      Avg |      Avg |  Write Rate | Number |      Avg | Bytes/ | File Name=!x/^/(proc|sys)/
    Write |    Write | MiBytes/sec |     of |   Writes |   Call |  PE=HIDE
 Time per |  MiBytes |             | Writer |      per |        | 
   Writer |      per |             |  Ranks |   Writer |        | 
     Rank |   Writer |             |        |     Rank |        | 
          |     Rank |             |        |          |        | 
|-----------------------------------------------------------------------------
| 0.004157 | 1.576238 |  379.132317 |      1 | 25,829.0 |  63.99 | sharpened.pgm
| 0.000024 |       -- |    0.000000 |     64 |     35.4 |   0.00 | _UnknownFile_
| 0.000013 | 0.000043 |    3.270135 |     64 |      1.3 |  34.92 | stdout
|=============================================================================
```

This shows that the I/O is purely serial (there is only 1 rank reading or writing).

### Comparing the performance profile at larger core counts

Although the information provided by the profile at a single point can be 
useful for understanding the performance we can often get much more insight 
by rerunning the profiling at different parameter settings and analysing how
the profile of the application changes.

The parameter we have been varying for the sharpen application is the number
of cores we are using. So, lets vary this and see how the profile changes.

> ## Profile on 256 cores
> Rerun the profiling experiment on 256 cores. Look at the performance profile
> compared to the one you have for 64 cores. Are there differences in:
> 
> - proportion of time spent in MPI functions;
> - memory use; and
> - I/O use?
> 
> Can you offer any explanation for the changes you observe and propose how 
> they might affect the performance of the application?
> Can you guess what might happen if you double the core count again to 
> 512 cores? Test if your theory is correct by producing the profile.
> 
> > ## Solution
> > You should see the following
> >
> > - MPI proportion: around 20%
> > - Memory use: 21,792.3 MiBytes
> > - I/O use: Same as for the 64 node case
> >
> > The increase in time in MPI functions mean that more of the runtime of
> > application is spent communicating between cores using MPI rather than
> > actually doing computational work on our problem. This means that the
> > scaling will move further away from ideal scaling (which assumes zero
> > parallel overheads).
> >
> > When we increase the core count again we would expect to see the proportion
> > of time in MPI functions increase still further leading to the application
> > scaling moving even further away from ideal.
> {: .solution}
{: .challenge}

## Summary

As you have seen, using even basic profiling information can give you insights
into why the scaling of your application on HPC systems is varying in the
way that it does. It also potentially allows you to predict performance on 
different parameter values without actually running the benchmark (which can 
be useful if you do not have access to a system that can run up to the scale
that you are interested in exploring).

Profiling is a very large topic in itself and we have only scraped the surface
of the type of information that is available and how it can be used. If you 
are interested in learning more about this topic, there are other courses
out there that can potentially help you.


{% include links.md %}