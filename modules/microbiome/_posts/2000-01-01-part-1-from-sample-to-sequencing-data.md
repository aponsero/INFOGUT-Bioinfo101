---
title: "Part 1: From sample to sequencing data"
---

# Part 1: From sample to sequencing data
{: .no_toc}

<div class="panel" markdown="1">
**Format:** lecture notes, no software needed.<br>
**Estimated time:** 10 minutes.<br>
**Author:** Petra Polakovičová, Institute for Clinical and Experimental Medicine, Czech Republic.
</div>

**On this page**

* TOC
{:toc}

---

## 1. What data does a bioinformatician work with?

A sample from your colon model goes into a sequencer. What comes out is not a list of microbes: it is a very large text file of DNA sequences. The job of the bioinformatics pipeline is to turn that text file into an **abundance table**: which taxa are in each sample, and how much of each.

<figure>
  <img src="{{ site.baseurl }}/modules/microbiome/images/from-sample-to-table.png" alt="A sample tube goes into a sequencer, which produces a FASTQ file of millions of reads; the reads are turned into an abundance table with taxa in rows and samples in columns">
  <figcaption>From the sample to the sequencing output (a FASTQ file of millions of short text strings, made of A, C, G and T, with read quality scores) and then to an abundance table.</figcaption>
</figure>

The two ends of this process look very different:

- **The FASTQ file** holds millions of short DNA sequences, called **reads**, each with a quality score for every base. It says nothing about which microbe each read came from.
- **The abundance table** has one row per taxon and one column per sample. Each cell is the number of reads (or the proportion of reads) assigned to that taxon in that sample.

Everything in between is the pipeline, described in [Part 2]({{ site.baseurl }}/modules/microbiome/part-2-the-bioinformatics-pipeline/).

---

## 2. The FASTQ file

A FASTQ file is a plain text file. Each sequencing read takes **four lines**:

1. A **header**, starting with `@`. It identifies the read and can include information from the sequencing run (instrument, run, flow cell, position on the flow cell).
2. The **sequence** itself, made of A, C, G and T (and N for a base that could not be called).
3. A **spacer**: a line starting with `+`.
4. The **quality** of each base, encoded as one ASCII character per base. This line is exactly as long as the sequence.

<figure>
  <img src="{{ site.baseurl }}/modules/microbiome/images/fastq-anatomy.png" alt="Three reads of a FASTQ file, with the header, sequence and quality lines of the first read highlighted">
  <figcaption>Three reads from a FASTQ file; the first one is highlighted: header (yellow), sequence (orange), quality (green). Source: <a href="https://biocorecrg.github.io/PhD_course_genomics_format_2021/fastq.html">biocorecrg.github.io/PhD_course_genomics_format_2021/fastq.html</a></figcaption>
</figure>

### Quality scores

Each character on the quality line stands for a number, the **Phred quality score (Q)**, which expresses how likely it is that the base was called incorrectly. The higher the score, the more reliable the base.

| Character | Q | Probability of an incorrect base call | Base call accuracy |
|---|---:|---:|---:|
| `+` | 10 | 0.1 (1 in 10) | 90% |
| `5` | 20 | 0.01 (1 in 100) | 99% |
| `?` | 30 | 0.001 (1 in 1,000) | 99.9% |

Every 10 points of Q means a ten times smaller chance of error. Q30 is a common benchmark for good-quality bases.

**Try it:** if you have a FASTQ file on your computer, `head -8 file.fastq` shows its first two reads (see the [Command line module]({{ site.baseurl }}/modules/command-line/part-2-asking-questions-of-your-files/)). Files are often compressed (`.fastq.gz`); in that case, use `gunzip -c file.fastq.gz | head -8`.

---

## 3. Two ways to sequence a microbial community

Colon model studies mostly use one of two approaches. They start from the same DNA extract, but they read very different parts of it.

<figure>
  <img src="{{ site.baseurl }}/modules/microbiome/images/16s-vs-shotgun.png" alt="Left: 16S rRNA amplicon sequencing produces multiple copies of fragments from one target gene. Right: shotgun metagenomics produces short sequence fragments from all the DNA">
  <figcaption>16S rRNA amplicon sequencing (left) and shotgun metagenomics sequencing (right). Adapted from: Happy Belly Bioinformatics.</figcaption>
</figure>

**16S rRNA amplicon sequencing** targets a single gene. The 16S rRNA gene is present in all bacteria and archaea. It contains conserved regions, which are almost identical across species, alternating with variable regions, which differ between taxa. PCR primers bind to the conserved regions and amplify one or a few variable regions (for example V3–V4 or V4). The result is many copies of fragments from one target gene, and the differences between those fragments tell taxa apart.

**Shotgun metagenomics** sequences everything. All the DNA in the sample, from every organism, is broken into short random fragments that are all sequenced. Nothing is targeted, so the reads come from all genes of all genomes present: bacteria and archaea, but also viruses, fungi and any host DNA.

### How much data?

| | 16S rRNA amplicon sequencing | Shotgun metagenomics |
|---|---|---|
| Data per sample | about 20–50 MB | about 3–10 GB |
| In novels' worth of text | about 40–100 novels | about 6,000–20,000 novels |

These comparisons assume a novel of about 0.5 MB of plain text (roughly 100,000 words). A shotgun sample holds roughly 60 to 500 times more data than a 16S sample, which affects storage, computing time and cost.

---

## 4. 16S vs shotgun: what can each tell you?

| What can it tell you? | 16S rRNA amplicon sequencing | Shotgun metagenomics |
|---|---|---|
| Best naming precision | Genus (species sometimes) | Species (strain sometimes) |
| Bacteria and archaea | Yes | Yes |
| Viruses and fungi | No | Yes |
| Who's there, and how much (relative abundance) | Yes | Yes |
| Rebuild whole genomes from the mix (MAGs) | No | Yes |
| What the microbes can do (genes, pathways) | No (prediction only) | Yes |
| Data per sample | ~20–50 MB (cheap) | ~3–10 GB (expensive) |

Most of this table follows from what each method reads:

- 16S reads cover a few hundred bases of one gene, so closely related species can have identical sequences: this limits naming precision.
- The 16S gene does not exist in viruses, and fungi are usually studied with a different marker (ITS), so 16S does not see them.
- Shotgun reads come from whole genomes, so they can be assembled into genomes and they directly show which genes are present.

### Predicting function from names

With 16S data, functions can only be **predicted**: tools look up which genes are usually found in the genomes of the taxa you detected. The problem is that two members of the same species can have very different gene content.

<figure>
  <img src="{{ site.baseurl }}/modules/microbiome/images/pug-and-wolf.png" alt="A pug and a wolf, both labelled Canis lupus">
  <figcaption>Trying to predict behaviour based on species: same species, wildly different behaviour.</figcaption>
</figure>

A pug and a wolf are the same species. Knowing only the species name, you would make poor predictions about how either of them behaves. Predicting what a microbial community does from a list of 16S names has the same limitation: it is an informed guess, not a measurement. If function is central to your question, shotgun metagenomics (or measuring the metabolites directly, see the [Metabolomics data module]({{ site.baseurl }}/modules/metabolomics/programme/)) gives much stronger evidence.

---

## What's next

[Part 2: The bioinformatics pipeline]({{ site.baseurl }}/modules/microbiome/part-2-the-bioinformatics-pipeline/) follows the reads through the pipeline, from quality control to the final tables.
