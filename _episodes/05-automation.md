---
title: "Automating performance data collection and analysis"
teaching: 20
exercises: 20
questions:
- "Why is automation useful?"
- "What considerations should I have when automating collecting performance data?"
- "Are there any tools that can help with automation?"
objectives:
- "Understand how automation can help with performance measurements."
- "See how different automation approaches work in practice."
keypoints:
- "Automation can help make benchmarking more rigourous"
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
 - It can make it easier to rerun benchmarking in the future (on the same or different systems)

## Automation approaches

There are a number of different potential approaches to automation: ranging from
using a existing framework to creating your own scripts. The approach you choose
generally depends on how many benchmark runs you need to perform and how much
data you plan to collect. The larger the dataset you are interested in, the more
time it will be worth investing in automation.

> ## Many ways to skin a cat!
> Like many programmatic problems, there are an extremely large number of ways
> to address this problem. We will cover one approach here using bash scripting,
> but that does not mean you could not use Perl, Python or any other language
> and approach. Often the best choice depends on what toolset you are already
> familiar with and happy using.
{: .callout}

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
not strictly needed here, but will demonstrate an approach that is more 
flexible for your potential use cases than submitting using `srun` directly.

### Write the template job submission script

Firstly, we want to write a job submission script that will submit one instance
of a benchmark run.

> ## Writing a job submission script
> Use the [ARCHER2 online documentation](https://docs.archer2.ac.uk/user-guide/scheduler/)
> to produce a job submission script that runs the Sharpen application on a single core on
> a single node.
> > ## Solution
> > The following job submission script sets the same options that we used when using
> > `srun` directly.
> > ```
> > #!/bin/bash
> > 
> > # Slurm job options (job-name, compute nodes, job time)
> > #SBATCH --job-name=sharpen_bench
> > #SBATCH --time=0:20:0
> > #SBATCH --nodes=1
> > #SBATCH --tasks-per-node=1
> > #SBATCH --cpus-per-task=1
> > #SBATCH --account=ta012        
> > #SBATCH --partition=standard
> > #SBATCH --qos=standard
> > #SBATCH --reservation=ta012_89
> > 
> > # Setup the job environment (this module needs to be loaded before any other modules)
> > module load epcc-job-env
> > # Load the sharpen module
> > module load training/sharpen
> > 
> > srun --distribution=block:block --hint=nomultithread sharpen-mpi.x > bench_${SLURM_NTASKS}cores_run1.out
> > 
> > ```
> > {: .language-bash}
> {: .solution}
{: .challenge}

### Identify parameters that will vary in the script and write parameterised submission script

Now, we need to take our static script and identify all the locations where things will
change dynamically during automated setup and any additional parameters/data that we
need to include. Remember, we thought the following things would change:

 - Number of cores
 - Run number at this data point

and we need to remember to include these additional data for each run (because they are
not included in the output from Sharpen):

 - Date and time of the run
 - Run number at this data point

Looking at our basic script, we can see that the number of cores appears in the following
line:

```
#SBATCH --tasks-per-node=1
```

(It may also sometimes require an increase in the number of nodes if we use more cores than
are in a single node but we shall ignore this for now.)

The run number does not appear anywhere in our script at the moment, so we will need to 
find a way to include it. When we do this, it will also address including the run number
in the output in some way.

Finally, we need to capture the date/time in a way that can be used in our output. One 
convenient way to do this is to capture the time as *epoch* time (seconds since
1970-01-01 00:00:00 UTC). We can use the command `date +%s` to get this. So, we could
capture this in a script with something like:

```
timestamp=$(date +%s)
```
{: .language-bash}

(This captures the current epoch time into the `$timestamp` variable. Note that there
cannot be any spaces on either side of the `=` sign in bash scripts.)

We are going to use a bit of bash scripting to create a version of our job submission
script that will allow us to specify the values of the variables that we want to 
change as arguments and that will include the date. First we will show the complete
script and then we will explain what is going on.

Here is the script (which I saved in `run_benchmark.sh`):

```
# Capture the arguments
ncores=$1
nruns=$2

sbatch <<EOF
#!/bin/bash

# Slurm job options (job-name, compute nodes, job time)
#SBATCH --job-name=sharpen_bench
#SBATCH --time=0:20:0
#SBATCH --nodes=1
#SBATCH --tasks-per-node=${ncores}
#SBATCH --cpus-per-task=1
#SBATCH --account=ta012        
#SBATCH --partition=standard
#SBATCH --qos=standard
#SBATCH --reservation=ta012_89
 
# Setup the job environment (this module needs to be loaded before any other modules)
module load epcc-job-env
# Load the sharpen module
module load training/sharpen

for irun in \$(seq 1 ${nruns})
do
   timestamp=\$(date +%s)
   srun --distribution=block:block --hint=nomultithread sharpen-mpi.x > bench_\${SLURM_NTASKS}cores_run\${irun}_\${timestamp}.out
done
EOF
```
{: .language-bash}

To use this script to run three copies of the benchmark on a single
core, we would use the command:

```
bash run_benchmark.sh 1 3
```
{: .language-bash}
```
Submitted batch job 65158
```
{: .output}

The first argument to the script specifies the number of cores and the second argument,
the number of copies to run. In the script itself, these are captured into the
variables `$ncores` and `$nruns` by the lines:

```
ncores=$1
nruns=$2
```

We then use a form of bash redirection called a *here document* to pass the script to the `sbatch` command with the
values of `$ncores` and `$nruns` substituted in in the correct places: the `SBATCH` option
for `$ncores` and the `seq` command for `$nruns`. One thing to note in the *here document*
is that we must *escape* the `$` for variables we want to still be variables in the 
script by preceding them with a backslash otherwise bash will try to substitute them
in the same way it does for the `$ncores` and `$nruns` variables. You can see this
in action in the `for` line, `timestamp` line and the `srun` line: we want these variables to be
interpreted when the script *runs* so they need to be escaped, the unescaped variables
will be interpreted when the script is submitted.

Remember, that the value of the walltime specified for the job now needs to be long
enough for however many repeats of the runs we want to run (20 minutes is plenty for
the Sharpen application). (You could, of course, calculate this in the script based
on a base runtime and the specified number of repeats if you so wish.)

Now we have a script that can dynamically take the values we want for benchmarking, and
we already have a script that can automatically extract the data from all benchmark
runs, the final step is to setup the script that can automate the submission of the
different benchmark cases.

### Write the benchmark automation script and submit the jobs

To automate the submission, we need a script that could call our parameterised submission
script with the different parameters needed. For this simple case, we will encode
the different parameters in the script itself but for more complex cases it may be
worth considering keeping the parameters in a separate file that can be read by the 
script.

In this case, the only parameter we want to vary is the number of cores for the
benchmark (we are going to use 3 repetitions per core count). Here is an example 
of a short bash script that would implement this:

```
# Set the core counts we want to benchmark
core_counts="1 2 4 8 16 32 64 128"
# Set the number of repeat runs per core count
nrep=3

# Loop over core counts, submitting the jobs
for cores in $core_counts
do
   bash run_benchmark.sh $cores $nrep
done
```
{: .language-bash}

Assuming this has been saved as `launch_benchmark.sh` in the same directory as our
`run_benchmark.sh` script, we should now be able to submit the full set of benchmark
runs with a single command:

```
bash launch_benchmark.sh
```
{: .language-bash}
```
Submitted batch job 65174
Submitted batch job 65175
Submitted batch job 65176
Submitted batch job 65177
Submitted batch job 65178
Submitted batch job 65179
Submitted batch job 65180
Submitted batch job 65181
```
{: .output}

You can see that there are 8 jobs submitted, one for each of the core counts we specified
in the script.

> ## Modify the parameterised submission script to allow for multinode runs
> Can you modify the parameterised job submission script to allow it to use multiple nodes
> rather than just a single node so the number of cores used for the benchmark can go 
> beyond 128 cores?
> 
> > ## Solution
> > As for everything, there are many different ways to do this. Possibly the simplest 
> > solution is to expand the number of arguments the script takes to allow you to 
> > specify the number of nodes and the number of cores to use per node. An example
> > solution would be:
> > ```
> # Capture the arguments
> > nnodes=$1
> > ncorespernode=$2
> > nruns=$3
> > 
> > sbatch <<EOF
> > #!/bin/bash
> > 
> > # Slurm job options (job-name, compute nodes, job time)
> > #SBATCH --job-name=sharpen_bench
> > #SBATCH --time=0:20:0
> > #SBATCH --nodes=${nnodes}
> > #SBATCH --tasks-per-node=${ncorespernode}
> > #SBATCH --cpus-per-task=1
> > #SBATCH --account=ta012        
> > #SBATCH --partition=standard
> > #SBATCH --qos=standard
> > #SBATCH --reservation=ta012_89
> >  
> > # Setup the job environment (this module needs to be loaded before any other modules)
> > module load epcc-job-env
> > # Load the sharpen module
> > module load training/sharpen
> > 
> > for irun in \$(seq 1 ${nruns})
> > do
> >    timestamp=\$(date +%s)
> >    srun --distribution=block:block --hint=nomultithread sharpen-mpi.x > bench_\${SLURM_NNODES}nodes_\${SLURM_NTASKS}cores_run\${irun}_\${timestamp}.out
> > done
> > EOF
> > ```
> > {: .language-bash}
> > This would be used with something like (for 2 nodes with 128 cores used per node, 256
> > cores in total, 3 repeats):
> > ```
> > bash run_benchmark.sh 2 128 3
> > ```
> > {: .language-bash}
> {: .solution}
{: .challenge}

## More complex automation: potential frameworks

If you are planning to reuse benchmarks many times in the future and/or have complex requirements
for the different parameters you need to use in your benchmarking you may find it worth 
investing the time and effort to use an existing test or benchmark framework. The use
of such frameworks is beyond the scope of this course but a couple of potential options that
have been used successfully in the past are:

- [ReFrame](https://reframe-hpc.readthedocs.io/en/stable/) - an HPC regression testing framework
  developed by [CSCS](http://www.cscs.ch/) that also includes options to capture performance
  data and log it.
- [JUBE](https://www.fz-juelich.de/ias/jsc/EN/Expertise/Support/Software/JUBE/jube.html#:~:text=The%20JUBE%20benchmarking%20environment%20provides,Centre%20of%20Forschungszentrum%20J%C3%BClich%2C%20Germany.) - an HPC benchmarking framework
  developed at the [Jülich Supercomputing Centre](https://www.fz-juelich.de/ias/jsc/EN/Home/home_node.html).
- [Dakota](https://dakota.sandia.gov/) - advanced parametric study and optimisation tool.

In the final section of this course, we take a brief look at *Profiling* performance of 
applications on HPC systems.

{% include links.md %}
