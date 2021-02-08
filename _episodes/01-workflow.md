---
title: "Understanding your HPC workflow"
teaching: 30
exercises: 15
questions:
- "What are the components of my HPC research workflow?"
- "What impact would performance improvements have on the different components?"
- "How can I understand the dependencies between the different components?"
objectives:
- "Gain a better understanding of my HPC research workflow."
- "Appreciate the potential impact of performance improvements in the HPC research workflow."
keypoints:
- "Your HPC research workflow consists of many components."
- "The potential performance improvement depends on the whole workflow, not just the individual component."
---

> ## And what do you do?
> Talk to your neighbour or add a few sentences to the etherpad describing the research
> area you work in and why you have come on this course.
{: .challenge}

In this section we will look at research workflows that involve an HPC component. It
is useful to keep your own workflow in mind throughout. Some of the steps or components
we describe may not be relevant to your own workflow and you may have additional steps or
components but, hopefully, the general ideas will make sense and help you analyse your
own workflow.

What do we mean by *workflow* here? In this case, we mean the individual components and
steps you take from starting a piece of work to generating actual data that you can use
to "do" research. As this is an HPC course, we are assuming that the use of HPC is an 
integral part of your workflow.

As an example, consider the following simple workflow to
explore the effect of defects in a crystal structure (we know that this is simplified
and does not contain all the steps that might be interesting):

 - Step 1: Source crystal structure from experimental data or previous modelling studies
 - Step 2: Convert crystal structure into input format for modelling using VASP
 - Step 3: Define the input parameters for the planned calculations
 - Step 4: Upload the input files for the VASP calculations to the HPC system
 - Step 5: Run geometry relaxation on the input system
 - Step 6: Analyse output to verify that relaxed (optimised) geometry is reasonable
 - Step 7: Run calculations on relaxed structure to generate properties of interest
 - Step 8: Download output from HPC system to local resource for analysis
 - Step 9: Analyse output from calculations to generate meaningful results for research

Even this overly-simplified example has more steps than you might imagine and you can
also see that some steps might be repeated as required during the actual use of this
workflow. For example, you may need to try a variety of different input configurations
for any of the calculations to obtain a reasonable model of the system of interest.

> ## Your workflow
> Talk to your neighbour (or rubber duck) and explain your research workflow. In particular,
> try to break it down into separate steps or *components*. Which of the components are the
> most time consuming and why? Why do the components have to be in the order you have
> described them?
{: .challenge}

## What is worth optimising?

When thinking about your workflow, you should try and assess which components are the most
time consuming - this may not be a single component, your workflow could have multiple 
components with similar lengths.

Is the use of HPC one of the most time consuming elements? If not, then improving the
performance (or time taken) by the HPC component may not actually yield any improvement
in the time taken for your workflow.

This situation is analogous to transatlantic flight (and to parallel performance on HPC
systems, as we shall see later!). Transatlantic flight is essentially a very simple 
workflow:

- Step 1: Travel to the origin airport
- Step 2: Bag drop and navigate security control
- Step 3: Board aeroplane
- Step 4: Fly
- Step 5: Disembark aeroplane
- Step 6: Passport control and baggage reclaim
- Step 7: Leave airport and continue journey

For our example, consider that the "Fly" component is what we are looking at improving
(say, by designing a faster aeroplane). At the moment, the components take the following
times:

- Steps 1-3: 3 hours (1 hour travel to the airport, 2 hours to get onto the 'plane)
- Step 4: 6 hours
- Steps 5-7: 1 hour

At the moment, flying represents 60% of the time for the total workflow. If we could
design an aeroplane to reduce the flight time to 3 hours (let's call this 'plane,
Concorde) then we could reduce the total time to 7 hours and flying would represent
43% of the time for the total workflow. Any further reductions in flying time would
have less impact as the other workflow steps start to dominate.

Moving back to a research workflow involving HPC, if the HPC component of the 
workflow is a low percentage of the total time, then the motivation for optimising
it (at least in terms of improving the overall workflow performance) is not very high.

> ## Other motivations
> Of course, improving the time taken for the overall workflow is not the only
> reason for optimising and/or understanding the performance of the HPC component.
> You may also want to maximise the amount of modelling/simulation you can get
> for the resources you have been allocated. Or, you may want to understand performance
> to be better able to plan your use of resources in the future.
{: .callout} 

## Components and dependencies

As well as understanding the components that make up your workflow, you should also
aim to understand the *dependencies* between the components. By dependencies, we mean
how does progress with one component depend on the output or completion of another
component. In the simplified workflow examples we have thought about previously, the
different components were largely sequential - the previous component had to complete before
the next component could be started. In reality, this is not often true: there may be later
components that are not strictly dependent on completion of previous components. Or, 
our workflow may be made up of many copies of similar sequential workflows. (Or, even 
a combination of both of these options.)

When we do not have strict sequential dependencies between components of our workflows
then there is the potential to make the workflow more efficient by allowing different
components to overlap in time or run in parallel. For example, consider the following
simplified materials modelling workflow:

 - Step 1: Source crystal structures from experimental data or previous modelling studies
 - Step 2: Convert crystal structures into input format for modelling using VASP
 - Step 3: Define the input parameters for the planned calculations at a range of temperatures and pressures
 - Step 4: Upload the input files for the VASP calculations to the HPC system
 - Step 5: Run VASP calculations on HPC system
 - Step 6: Download output from HPC system to local resource for analysis
 - Step 7: Analyse output from individual VASP calculations
 - Step 8: Combine multiple analyses/output to generate meaningful results for research

As there are now multiple experimental crystal structures to start from and 
multiple independent calculations at different physical conditions for each of
the starting crystal structures we can rewrite the workflow in a way that 
exposes the dependencies and potential parallelism:

- For each crystal structure:
  - (1) Source crystal structures from experimental data or previous modelling studies
  - (2) Convert crystal structures into input format for modelling using VASP (depends on (1))
  - For each physical condition set:
    - (a) Define the input parameters for the planned calculations at a range of temperatures and pressures (depends on (2))
    - (b) Upload the input files for the VASP calculations to the HPC system (depends on (a))
    - (c) Run VASP calculations on HPC system (depends on (b))
    - (d) Download output from HPC system to local resource for analysis (depends on (c))
    - (e) Analyse output from individual VASP calculations (depends on (d))
- Combine multiple analyses/output to generate meaningful results for research (depends on completing all (e))

Assuming we have 5 different crystal structures and 10 different sets of physical conditions 
for each crystal structure we have the potential for running 5&times; steps (1) and (2) 
simultaneously and the potential for running 50&times; steps (a)-(e) simultaneously. 
Of course, all of this parallelism may not be realisable as some of the potentially parallel
steps may be manual and there may only be one person available to perform these steps. However,
with some planning and thought to automation before we start this workflow we may be able
to exploit this potential parallelism.

> ## Your workflow revisited
> Think about your workflow again. Can you identify any opportunities for parallelism
> and/or automation that you are not already exploiting? Is there enough to be gained
> from working to parallelise/automate that would make it worthwhile to take some time
> to do this?
{: .challenge}

As always with research, the temptation is to just get stuck in and start doing work. However,
it is worth taking a small amount of time at the start to think about your workflow and 
the dependencies between the different steps to see what potential there is for 
exploiting parallelism and automation to improve the efficiency and overall time taken
for the workflow. Note that often people think about this in terms of the overall 
research *project* workplan but then do not take the addtional step to think about the
workflows within the different project steps.

## Understanding performance of HPC components

For the rest of this course we are going to focus on understanding the performance
of HPC aspects of a research workflow and look at how we use that understanding to
make decisions about making our use (or future use) of HPC more efficient. Keep in 
mind the air travel analogy when we are doing this though - making the
HPC aspect more efficient may not have the impact you want on your overall workflow
if other aspects come to dominate the total time taken (in fact, we will see this 
issue rear its head again when we talk about the limits of parallel scaling later
in the course).


{% include links.md %}