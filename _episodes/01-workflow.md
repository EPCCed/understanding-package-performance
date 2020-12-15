---
title: "Understanding your HPC workflow"
teaching: 30
exercises: 10
questions:
- "What are the components of my HPC research workflow?"
- "What impact would performance improvements have on the different components?"
- "How can I understand the dependencies between the different components?"
objectives:
- "Gain a better understanding of my HPC research workflow."
- "Appreciate the potential impact of performance improvements in the HPC research workflow."
keypoints:
- "Your HPC research workflow consists of many components - some of these are manual."
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
> Talk to your neighbour (or rubber duck) and explain your research workflow. In particulr,
> try to break it down into separate steps or *components*. Why do the components have to 
> be in the order you have described them?
{: .challenge}

## Components and dependencies

{% include links.md %}