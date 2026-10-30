---
title: Module overview
---

# Metabolomics data

{% include module_overview.html %}

**Central question:** how do samples from a colon model become a metabolomics data matrix, and what must we check before interpreting it?

This module introduces metabolomics data to people with little or no previous experience. It follows the complete journey from an in vitro colon model sample to a processed data table, and finishes with a short R demonstration on a small, real colon-model dataset.

## What you will learn

By the end of this module, you should be able to:

1. Explain what metabolomics measures and how it complements microbiome sequencing.
2. Distinguish targeted, semi-targeted and untargeted metabolomics.
3. Recognise the main analytical platforms and the types of signals they generate.
4. Describe how analytical signals are converted into a processed data matrix.
5. Recognise metadata, measurement and annotation tables.
6. Identify essential checks required before biological interpretation.

## Prerequisites

- **Part 1:** none.
- **Part 2:** basic R and the tidyverse (reading a CSV, `mutate`, `pivot_longer`, `ggplot`), as covered in the [R and RStudio module]({{ site.baseurl }}/modules/r/programme/).

## Lecture and demo

| Part | Content | Estimated time |
|---|---|---|
| [Part 1: From sample to data matrix]({{ site.baseurl }}/modules/metabolomics/part-1-from-sample-to-data-matrix/) | What metabolomics measures, analytical strategies and platforms, sample preparation, data tables, normalisation, the reality check | 45 min |
| [Part 2: Hands-on demo in R]({{ site.baseurl }}/modules/metabolomics/part-2-hands-on-demo-in-r/) | Exploring a targeted organic-acid dataset from a colon-model study | 45 to 60 min |

## Demo files

[Download the demo dataset and R script (zip)]({{ site.baseurl }}/modules/metabolomics/data/metabolomics-demo.zip)

## Acknowledgements

These teaching materials were developed drawing on contributions from Dr. Jaroslav Havlik and Dr. Saetbyeol Lee, Czech University of Life Sciences Prague.

Taught at the Madrid 2026 school as: Intro to metabolomics concepts.
