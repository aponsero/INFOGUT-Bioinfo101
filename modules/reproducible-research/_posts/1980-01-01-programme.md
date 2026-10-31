---
title: Module overview
---

# Reproducible research

{% include module_overview.html %}

**Lecture goal:** to give you a practical grasp of tracking your code, managing software environments and running workflows, so that you understand how they make computational analysis FAIR, and why adopting them is worth it.

The [FAIR data module]({{ site.baseurl }}/modules/fair-data/programme/) was about organising and documenting your **data**. This module is about the other half of reproducibility: your **code**, the **software** it runs with, and the **steps** that turn raw data into results.

This module explains the concepts. You do not need to install or run anything: the pages show what the tools look like when they run.

## What you will learn

- Why version control matters, and how Git and GitHub keep the history of your code
- What a software environment is, and how Conda installs and isolates the exact versions of the tools you need
- What a workflow manager is, and how Nextflow chains analysis steps into a pipeline that runs in parallel and can resume after a crash

## Prerequisites

- The [Command line module]({{ site.baseurl }}/modules/command-line/programme/): commands, scripts and loops.
- The [FAIR data module]({{ site.baseurl }}/modules/fair-data/programme/) is recommended.

## Lecture

| Part | Content | Estimated time |
|---|---|---|
| [Part 1: Version control with GitHub]({{ site.baseurl }}/modules/reproducible-research/part-1-version-control-with-github/) | In preparation | |
| [Part 2: Software environments with Conda]({{ site.baseurl }}/modules/reproducible-research/part-2-software-environments-with-conda/) | The three "hells" of a bioinformatician, what Conda is, channels, environment files, Conda and pip | 20 min |
| [Part 3: Workflows with Nextflow]({{ site.baseurl }}/modules/reproducible-research/part-3-workflows-with-nextflow/) | The assembly line, the four building blocks, `-resume`, and a quality-control pipeline compared with a Bash script, shown running | 20 min |

Taught at the Madrid 2026 school as two sessions: GitHub, and Environments and workflows (Conda, Nextflow).
