---
title: Module overview
---

# Microbiome data

{% include module_overview.html %}

**Lecture goal:** to learn the basic bioinformatics steps, and understand what "data processing and analysis" really means in microbiome research.

This module is a lecture rather than a hands-on tutorial: there is no code to run. It follows sequencing data from the machine to the abundance table that you will analyse, for the two most common approaches in colon model studies: 16S rRNA amplicon sequencing and shotgun metagenomics.

## What you will learn

- What data a bioinformatician works with: FASTQ files and abundance tables
- The differences between 16S rRNA amplicon sequencing and shotgun metagenomics, and what each can tell you
- The main steps of a bioinformatics pipeline: quality control, trimming, denoising or assembly or mapping, taxonomic assignment
- What the final count and taxonomy tables contain, and where your biological analysis begins

## Prerequisites

None. If you have done the [Command line module]({{ site.baseurl }}/modules/command-line/programme/), you will recognise the text files described here, and you can look inside them with `head`.

## Lecture

| Part | Content | Estimated time |
|---|---|---|
| [Part 1: From sample to sequencing data]({{ site.baseurl }}/modules/microbiome/part-1-from-sample-to-sequencing-data/) | FASTQ files and quality scores, 16S rRNA amplicon sequencing vs shotgun metagenomics | 10 min |
| [Part 2: The bioinformatics pipeline]({{ site.baseurl }}/modules/microbiome/part-2-the-bioinformatics-pipeline/) | Quality control, the 16S workflow (denoising, taxonomic assignment), the shotgun workflow (assembly, mapping), the final tables | 20 min |

## Going further

What you do with the final tables is covered in the [Statistics module]({{ site.baseurl }}/modules/statistics/programme/).

Taught at the Madrid 2026 school as: Intro to microbiome data for colon model experiments.
