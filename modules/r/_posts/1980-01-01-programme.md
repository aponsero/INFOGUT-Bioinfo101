---
title: Module overview
---

# R and RStudio

{% include module_overview.html %}

## What you will learn

By the end of this module, you will be able to:

- explain the difference between R and RStudio, and find your way around the RStudio window
- run code in an R notebook and read what R answers
- store values in variables, and work with vectors and data frames
- read CSV files into R, look at them before analysing them, and save results without touching the original data
- handle tables with the six `dplyr` verbs, and chain several steps
- tell biological replicates from technical repetitions, and check whether two replicates agree
- build a `phyloseq` object, plot relative abundances, compare groups with a t-test, and describe alpha and beta diversity
- store microbiome data in a `TreeSummarizedExperiment` and use miaverse functions to transform it and compare samples

## Prerequisites

None. This module assumes you have never written code.

## What you need

- **R and RStudio**, installed on your computer: see [Setup]({{ site.baseurl }}/modules/about/setup/).
- **Some R packages** for Parts 4 to 7. Each page lists the ones it needs, and [Setup]({{ site.baseurl }}/modules/about/setup/#install-the-r-packages) shows how to install them.
- **The notebook and data for each part**, downloaded from its page. Each download contains everything that part needs, so you can start with any of them.

## The data

All the hands-on parts use the same small dataset: six samples from an in vitro colon model experiment, in which two dairy products were fermented with human gut bacteria. The files contain the organic acids measured in each sample, the insulin released by pancreatic cells exposed to the fermented liquid, and which bacteria were present. [Part 3]({{ site.baseurl }}/modules/r/part-3-our-own-data/) describes the experiment and the files.

The same dataset is used in [Metabolomics, Part 2]({{ site.baseurl }}/modules/metabolomics/part-2-hands-on-demo-in-r/), which gives more detail on the original study.

## Tutorials

Follow the parts in order. Each one builds on the previous.

| Part | Content | Estimated time |
|---|---|---|
| [Part 1: Getting to know R and RStudio]({{ site.baseurl }}/modules/r/part-1-getting-to-know-r-and-rstudio/) | What R and RStudio are, the four panes, notebooks, packages. No code yet | 20 min |
| [Part 2: First steps in R]({{ site.baseurl }}/modules/r/part-2-first-steps-in-r/) | R as a calculator, variables, vectors, data types, data frames, getting help | 45 to 60 min |
| [Part 3: Our own data]({{ site.baseurl }}/modules/r/part-3-our-own-data/) | The experiment, reading CSV files, exploring real tables, saving results | 30 to 45 min |
| Part 4: Handling tables with dplyr | The six `dplyr` verbs, chaining steps | In preparation |
| Part 5: Do the two replicates agree? | Biological replicates and technical repetitions, comparing replicates | In preparation |
| Part 6: Microbiome data in R | `phyloseq`, relative abundance, first plots with `ggplot2`, t-test, alpha and beta diversity | In preparation |
| Part 7: TreeSummarizedExperiment and miaverse | The TreeSE object, transformations, common and rare taxa, comparing distances | In preparation |

Taught at the Madrid 2026 school as: Introduction to R and RStudio, followed by Hands-on R, sessions 1 to 4.
