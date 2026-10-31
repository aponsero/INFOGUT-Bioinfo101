---
title: "Part 2: The bioinformatics pipeline"
---

# Part 2: The bioinformatics pipeline
{: .no_toc}

<div class="panel" markdown="1">
**Format:** lecture notes, no software needed.<br>
**Estimated time:** 20 minutes.<br>
**Author:** Petra Polakovičová, Institute for Clinical and Experimental Medicine, Czech Republic.<br>
**Before you start:** this page builds on [Part 1]({{ site.baseurl }}/modules/microbiome/part-1-from-sample-to-sequencing-data/), in particular FASTQ files and the differences between 16S and shotgun data.
</div>

**On this page**

* TOC
{:toc}

---

## 1. A production line in a factory

A bioinformatics pipeline works like a production line. Raw material goes in at one end, passes through a series of stations, and a finished product comes out at the other end. Each station takes the output of the previous one, does one job, and passes its own output on.

<figure>
  <img src="{{ site.baseurl }}/modules/microbiome/images/pipeline-factory.png" alt="A production line: raw sequencing data, quality control, trimming and filtering, mapping or assembly or denoising, final edits and checks, processed data">
  <figcaption>The bioinformatics pipeline as a production line.</figcaption>
</figure>

| Station | What it does |
|---|---|
| **Raw sequencing data** | The FASTQ files from the sequencer |
| **Quality control (QC)** | Checks whether the reads are trustworthy |
| **Trimming / filtering** | Removes low-quality bases, adapters, primers and bad reads |
| **Mapping / assembly / denoising** | The core step, which differs between 16S and shotgun data (sections 3 and 4) |
| **Final edits and checks** | Taxonomic assignment, removal of contaminants, checks on the output |
| **Processed data** | The count table and the taxonomy table (section 5) |

The first two stations are basically the same for any type of sequencing data. The core step is where 16S and shotgun pipelines go different ways.

### Real pipelines

In practice, these steps are chained together by workflow tools, so that the same pipeline can be run in the same way on every dataset. Two examples from nf-core, a community collection of curated pipelines, show the same production line for each type of data:

<figure>
  <img src="{{ site.baseurl }}/modules/microbiome/images/nfcore-ampliseq.png" alt="Overview of the nf-core/ampliseq pipeline: quality control, inference of ASVs, taxonomic classification, filtering, visualisation and reporting">
  <figcaption>nf-core/ampliseq, a pipeline for amplicon (16S) data. CC-BY 4.0, design originally by Zandra Fagernäs.</figcaption>
</figure>

<figure>
  <img src="{{ site.baseurl }}/modules/microbiome/images/nfcore-taxprofiler.png" alt="Metro map of the nf-core/taxprofiler pipeline: quality control, adapter trimming, host removal, then several taxonomic classifiers in parallel">
  <figcaption>nf-core/taxprofiler, a pipeline for taxonomic profiling of shotgun metagenomic data. Credit: nf-core/taxprofiler.</figcaption>
</figure>

You do not need to understand every box. Notice that both start with quality control and trimming, and both end with tables and reports. Workflow tools such as Nextflow, used by nf-core, are covered in the [Reproducible research module]({{ site.baseurl }}/modules/reproducible-research/programme/).

---

## 2. Quality control: we have reads, but are they trustworthy?

Before using the reads, we check them, much as you would check a scanned document before relying on it: is it clean and readable, or stained and blurred?

<figure>
  <img src="{{ site.baseurl }}/modules/microbiome/images/quality-control.png" alt="A FASTQ file compared to a low-quality and a good-quality scanned document, and two quality plots: one where quality drops along the read and is trimmed, and one with good quality throughout">
  <figcaption>Quality control: reads can be like a low-quality or a good-quality scanned document. The plots show the quality score (y-axis) at each position along the reads (x-axis); poor-quality ends are trimmed off.</figcaption>
</figure>

### Reading a quality plot

The most common QC tool, FastQC, summarises the quality of all reads in a file as a plot like the ones above:

- The **x-axis** is the position along the read, from the first base to the last.
- The **y-axis** is the quality score (Q, see [Part 1]({{ site.baseurl }}/modules/microbiome/part-1-from-sample-to-sequencing-data/)).
- The background is coloured **green** for good quality, **orange** for reasonable quality and **red** for poor quality.

With Illumina sequencing, quality typically decreases towards the end of the reads. **Trimming** cuts off the low-quality ends (the scissors in the figure), so that only reliable bases are kept. Reads that are too short or too poor after trimming are removed altogether.

### What QC looks for

- **Low-quality reads**, or low-quality ends of reads
- **Sequencing artifacts**: adapter sequences left on the reads, primer sequences, duplicated reads
- **Host DNA contamination**: in shotgun data, reads from human (or other host) DNA are usually removed before further analysis
- **Other** problems specific to the run or the protocol

---

## 3. The 16S rRNA amplicon workflow

For 16S data, the core step is **denoising**, followed by **taxonomic assignment**.

### Is this really a different microbe, or just a typo?

Between the sample and the sequences, two steps introduce errors: the PCR that amplifies the 16S gene, and the sequencing itself.

<figure>
  <img src="{{ site.baseurl }}/modules/microbiome/images/amplicon-errors.png" alt="From the bulk sample through PCR and high-throughput sequencing to ASVs: PCR errors, chimeras, primer bias and sequencing errors add false variants, which bioinformatics removes">
  <figcaption>From the bulk sample to amplicon sequence variants (ASVs): PCR and sequencing add errors and chimeras, which the bioinformatics step needs to remove. [CITATION NEEDED: figure source]</figcaption>
</figure>

| Problem | Consequences | Solution: denoising |
|---|---|---|
| PCR errors | Inaccurate taxonomic identification | Corrects sequencer errors |
| Sequencing errors | Overestimated diversity | Removes chimeras |
| Chimeras | | Corrects point errors from PCR |

A **chimera** is an artificial sequence made of two parts from two different real sequences, formed when an incomplete PCR product acts as a primer on another template. It looks like a new organism, but it does not exist in the sample.

A single changed base can be a real difference between two microbes, or just a typo:

<div class="panel" markdown="1">
`TCTGGAGGAATACCGGTGGC` &nbsp;versus&nbsp; `TCTGAAGGAATACCGGTGGC`
</div>

If every typo were counted as a different microbe, the sample would appear to contain many more taxa than it really does, and the identifications would be wrong.

### Denoising with DADA2

**Denoising** decides which sequence differences are real and which are errors. The result is a set of **amplicon sequence variants (ASVs)**: exact sequences, which can differ from each other by a single base. Older pipelines instead grouped sequences into **OTUs** (operational taxonomic units), clusters of sequences that are at least 97% identical; ASVs give finer resolution and can be compared directly between studies.

<figure>
  <img src="{{ site.baseurl }}/modules/microbiome/images/dada2-workflow.png" alt="DADA2 workflow: filtering and trimming, error model learning and ASV inference for each read, merging of paired reads, sequence table construction, runs merging, chimera removal and taxonomy assignment, with the corresponding DADA2 functions">
  <figcaption>The DADA2 workflow, with the R function used at each step.</figcaption>
</figure>

DADA2 works in three stages:

1. **It learns what errors look like in your run.** From the data themselves, it estimates how often each type of base substitution happens at each quality score (`learnErrors()`).
2. **It then separates real biology from noise, base by base.** For each unique sequence, it asks whether it is too abundant to be explained as an error of a more abundant sequence. If not, it is treated as an error and merged into its parent (`dada()`).
3. **It finishes by merging, tabulating and removing chimeras.** Forward and reverse reads are merged (`mergePairs()`), the counts of each ASV in each sample are collected in a table (`makeSequenceTable()`), and chimeras are removed (`removeBimeraDenovo()`).

Tutorial: [benjjneb.github.io/dada2/tutorial.html](https://benjjneb.github.io/dada2/tutorial.html). Other denoising tools include Deblur and UNOISE3.

### Okay, we have a clean sequence, but who is it?

An ASV is still just a sequence. To give it a name, **taxonomic assignment** needs two things:

- a **classifier**: the method that compares your sequence to known ones;
- a **database**: the collection of reference sequences with known taxonomy.

<div class="panel" markdown="1">
`TCTGGAGGAATACCGGTGGC` &nbsp;becomes&nbsp; `TCTGAAGGAATACCGGTGGC` (after denoising) &nbsp;becomes&nbsp; *Escherichia* (after classification)
</div>

### Taxonomic classifiers

<figure>
  <img src="{{ site.baseurl }}/modules/microbiome/images/taxonomic-classifiers.png" alt="Two types of classifiers: k-mer based, which split a sequence into short words and count them, and sequence-identity based, which align the query to reference sequences and score the alignments">
  <figcaption>Taxonomic classifiers are based either on k-mers (A) or on sequence identity (B).</figcaption>
</figure>

**A) K-mer based classifiers** cut the sequence into all its short words of length k (in the figure, 3-mers: TTC, TCT, CTT...). They compare the set of words in your sequence with the sets found in the reference sequences for each taxon, and assign the taxon whose words match best. They are fast, and usually report a confidence score for each level of the taxonomy. Examples: `assignTaxonomy` (DADA2), `idtaxa`, RDP classifier, QIIME feature-classifier `classify-sklearn`, SINTAX, Kraken2, SPINGO.

**B) Sequence-identity based classifiers** align your sequence against the reference sequences, base by base, and use the best matching references (or a consensus of several good matches) to assign a name. Examples: QIIME feature-classifier `consensus-blast` and `consensus-vsearch`, `addSpecies` (DADA2), BLAST.

When no reference is close enough, a good classifier stops at a higher level (for example, family instead of genus) rather than guessing.

### Taxonomy databases

| Database | Link |
|---|---|
| SILVA | [www.arb-silva.de](https://www.arb-silva.de/) |
| Greengenes2 | [ftp.microbio.me/greengenes_release](https://ftp.microbio.me/greengenes_release/) |
| RDP | [sourceforge.net/projects/rdp-classifier](https://sourceforge.net/projects/rdp-classifier/) |
| GTDB | [gtdb.ecogenomic.org](https://gtdb.ecogenomic.org/) |

A name is only as good as the reference it comes from. Databases differ in which organisms they contain and in how they name them; the same taxon can carry different names in different databases or releases (for example, the phylum Firmicutes is called Bacillota in recent releases). Always report which classifier, database and database version you used: it is part of your metadata (see the [FAIR data module]({{ site.baseurl }}/modules/fair-data/programme/)).

---

## 4. The shotgun metagenomics workflow

For shotgun data, the core step takes one of two routes: **A) assembly and binning**, or **B) mapping to a reference database**.

<figure>
  <img src="{{ site.baseurl }}/modules/microbiome/images/shotgun-strategies.png" alt="From the organisms in a sample to sequencing reads, then either A) assembly and binning into MAGs, or B) mapping to reference genes or genomes for taxonomic quantification">
  <figcaption>Two strategies for shotgun metagenomic data: A) assembly and binning into metagenome-assembled genomes (MAGs); B) mapping reads to a reference database for taxonomic quantification.</figcaption>
</figure>

### A) Assembly and binning

**Assembly** puts the short reads back together into longer sequences, called **contigs**, by finding reads that overlap. It is like doing a jigsaw puzzle from the pieces alone, without the picture on the box.

Assembling one genome is a single puzzle. A metagenome is hundreds of puzzles mixed together in one pile, with some puzzles represented by many copies of each piece and others by only a few.

<div style="display: flex; gap: 16px; flex-wrap: wrap;">
  <figure style="flex: 1 1 240px;">
    <img src="{{ site.baseurl }}/modules/microbiome/images/one-genome-assembly.png" alt="A cat sitting next to a single small puzzle">
    <figcaption>One genome assembly. Credits: Vojtech Barton, RECETOX, Brno, Czech Republic.</figcaption>
  </figure>
  <figure style="flex: 1 1 240px;">
    <img src="{{ site.baseurl }}/modules/microbiome/images/metagenomic-assembly.png" alt="Many cats in a huge pile of mixed puzzle pieces">
    <figcaption>Metagenomic assembly. Credits: Vojtech Barton, RECETOX, Brno, Czech Republic.</figcaption>
  </figure>
</div>

**Binning** then sorts the contigs into groups that probably come from the same organism, using features such as their sequence composition and how abundant they are across samples. Each group is a **metagenome-assembled genome (MAG)**.

MAGs are valuable because they can reveal organisms that have never been cultured or sequenced before. But **MAGs are often contaminated and incomplete**: a bin can miss parts of its genome, or include contigs from other organisms. Their quality must therefore be checked before use.

| Step | Example tools |
|---|---|
| Assembly | MEGAHIT, metaSPAdes (short reads); metaFlye, Canu (long reads) |
| Binning | MetaBAT2, MaxBin2 |
| Quality check of MAGs | CheckM, CheckM2: estimate how **complete** each MAG is and how **contaminated** it is |
| Taxonomic classification of MAGs | GTDB-Tk: assigns a taxonomy to each MAG using the GTDB database |

### B) Mapping to a reference database

Instead of rebuilding genomes, the reads can be compared directly with a database of known genes or genomes. Each read that matches is assigned to the taxon it came from, and the counts are turned into relative abundances (**taxonomic quantification**). This is faster and simpler than assembly, and is the most common way to get a taxonomic profile from shotgun data.

Its limitation is the database itself:

<figure>
  <img src="{{ site.baseurl }}/modules/microbiome/images/reference-mapping-limits.png" alt="When a bacterium is not in the reference database, its reads are falsely mapped to a different genome, or they are eliminated">
  <figcaption>When a bacterium isn't in the reference database, its reads are falsely mapped to a different genome, or they are eliminated.</figcaption>
</figure>

Mapping can only find what the database already contains. Reads from an organism that is missing from the database are either assigned to the closest relative that is there (a wrong name), or discarded (the organism disappears from your results). Examples of reference-based profilers: Kraken2, mOTUs, MetaPhlAn.

---

## 5. From machine output to biology

Whatever the route, the pipeline ends with two linked tables.

**The count table** has one row per taxon (or ASV) and one column per sample. Each cell is the number of reads assigned to that taxon in that sample:

| TaxonID | S1 | S2 | S3 | S6 | S7 | ... | S120 |
|---|---:|---:|---:|---:|---:|---:|---:|
| Taxon 1 | 0 | 7 | 348 | 0 | 0 | 0 | 364 |
| Taxon 2 | 10 | 5 | 46 | 6 | 47 | 0 | 62 |
| Taxon 3 | 8 | 10 | 0 | 42 | 0 | 60 | 0 |
| Taxon 4 | 6 | 0 | 0 | 95 | 96 | 668 | 0 |
| Taxon 5 | 0 | 8 | 79 | 0 | 0 | 35 | 34 |
| ... | 2 | 0 | 3 | 0 | 12 | 0 | 0 |
| Taxon 978 | 450 | 0 | 55 | 5 | 32 | 0 | 12 |

**The taxonomic table** has one row per taxon, with its classification at each rank:

<div class="table-scroll" markdown="1">

| TaxonID | Kingdom | Phylum | Class | Order | Family | Genus |
|---|---|---|---|---|---|---|
| Taxon 1 | Bacteria | Bacteroidota | Bacteroidia | Bacteroidales | Bacteroidaceae | *Bacteroides* |
| Taxon 2 | Bacteria | Bacteroidota | Bacteroidia | Bacteroidales | Prevotellaceae | *Prevotella* |
| Taxon 3 | Bacteria | Bacillota | Clostridia | Lachnospirales | Lachnospiraceae | *Roseburia* |
| Taxon 4 | Bacteria | Bacillota | Clostridia | Oscillospirales | Oscillospiraceae | *Faecalibacterium* |
| Taxon 5 | Bacteria | Bacillota | Bacilli | Lactobacillales | Lactobacillaceae | *Lactobacillus* |
| ... | Bacteria | Verrucomicrobiota | Verrucomicrobiia | Verrucomicrobiales | Akkermansiaceae | *Akkermansia* |
| Taxon 978 | Archaea | Methanobacteriota | Methanobacteria | Methanobacteriales | Methanobacteriaceae | *Methanobrevibacter* |

</div>

The two tables are linked by the TaxonID. Together with a third table describing the samples (the sample metadata, see the [FAIR data module]({{ site.baseurl }}/modules/fair-data/programme/)), they are everything you need for the analysis.

Notice how many zeros the count table contains: many taxa are found in only some of the samples. This sparsity, and the fact that the counts are relative rather than absolute, are why microbiome data need the statistical methods described in the [Statistics module]({{ site.baseurl }}/modules/statistics/programme/).

<div class="panel" markdown="1">
**This is where bioinformatics ends and your biology begins.**
</div>

**See a real example:** the [Metabolomics data module demo files]({{ site.baseurl }}/modules/metabolomics/data/metabolomics-demo.zip) include `abundance.csv` and `taxonomy.csv`, the same two tables from a shotgun metagenomics study of a colon model. There, the abundances are relative abundances (percentages) rather than read counts.
