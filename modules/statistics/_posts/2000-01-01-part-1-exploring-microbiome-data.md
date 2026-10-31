---
title: "Part 1: Exploring microbiome data"
---

# Part 1: Exploring microbiome data
{: .no_toc}

<div class="panel" markdown="1">
**Format:** lecture notes, no software needed.<br>
**Author:** Eliana Ibrahimi, University of Tirana, Albania.
</div>

**On this page**

* TOC
{:toc}

---

## 1. Relative vs absolute abundances

- Sequencing mostly gives **relative abundances** that sum to a fixed total. This is called **compositional data**.
- If one taxon's true absolute abundance rises, every other taxon's relative share must fall to keep the total at 100%.
- Classical statistical methods treat every taxon as independent, which compositional data, by construction, is not.

*Gloor et al. (2017), Frontiers in Microbiology.*

### One taxon's real change creates two fake ones

| Taxon | Sample 1 (absolute) | Sample 1 (relative) | Sample 2 (absolute) | Sample 2 (relative) |
|---|---:|---:|---:|---:|
| Taxon A (really doubled) | 50 | 50.0% | 100 | 66.7% |
| Taxon B (unchanged) | 30 | 30.0% | 30 | 20.0% |
| Taxon C (unchanged) | 20 | 20.0% | 20 | 13.3% |
| **Total** | **100** | **100%** | **150** | **100%** |

Taxon B and C's absolute abundance never changed, but their relative abundance dropped by 10 and 6.7 percentage points respectively, purely because Taxon A really doubled. A naive per-taxon test would call all three "significant."

---

## 2. Controlling for sequencing depth

- Sequencing depth varies from sample to sample.
  - It is not directly related to the original amount of cells or DNA.
  - More depth doesn't mean more microbes.
- Greater sequencing depth leads to greater taxa detection.
- We therefore need to control, or normalize, for sequencing depth.

### Rarefaction

**Rarefaction** subsamples sequencing reads without replacement to a fixed depth, so that different samples have the same read count. It is commonly used before alpha and beta diversity calculations (Bardenhorst et al., 2022).

### Does rarefying solve the problem?

Whether rarefying is appropriate has been debated in the literature:

- McMurdie & Holmes (2014), *Waste Not, Want Not: Why Rarefying Microbiome Data Is Inadmissible*, PLoS Computational Biology. [doi.org/10.1371/journal.pcbi.1003531](https://doi.org/10.1371/journal.pcbi.1003531)
- Gloor et al. (2017), *Microbiome Datasets Are Compositional: And This Is Not Optional*, Frontiers in Microbiology. [doi.org/10.3389/fmicb.2017.02224](https://doi.org/10.3389/fmicb.2017.02224)
- Schloss (2023), *Waste not, want not: revisiting the analysis that called into question the practice of rarefaction*, mSphere. [doi.org/10.1128/msphere.00355-23](https://doi.org/10.1128/msphere.00355-23)

Part 2 comes back to this with the alternatives designed for compositional data.

---

## 3. Diversity measures

**Alpha diversity** describes the diversity within one sample:

- Richness: the number of different species
- Evenness: whether the species are present in similar, even abundances
- Shannon index
- Simpson diversity
- Chao1: predicts the total number of species in a sample based on the rarefaction curve [TO CHECK]

**Beta diversity** describes the differences between samples: Bray-Curtis index, Jaccard index, Aitchison distance, UniFrac distances.

**Gamma diversity** estimates diversity within a whole region.

*Diagram of alpha, beta and gamma diversity: credit to Jurgens, 2018.*

---

## 4. Graphical exploration

Before testing anything, look at the data. Common graphical summaries for microbiome data include:

| Plot | What it shows |
|---|---|
| Abundance bar plots | The composition of each sample, often at genus or phylum level |
| Relative abundance distributions | How the abundance of individual taxa varies across samples |
| Richness plots | Alpha diversity per sample or per group |
| Phylogenetic trees | Which lineages are present, with their prevalence or abundance |
| Heatmaps | Abundance of many taxa across many samples at once |
| Networks | Associations between taxa or between samples |

The examples shown in the lecture were made with the GlobalPatterns and Esophagus example datasets, using the phyloseq and mia R packages.

### Ordination methods

Ordination summarises the differences between samples in a few dimensions, so that they can be plotted:

- Canonical Correspondence Analysis (CCA)
- Principal Coordinate Analysis (PCoA)
- Non-metric Multidimensional Scaling (NMDS)
- Redundancy Analysis (RDA)

Ordination shows you the pattern. It does not test it: that is the job of the methods in [Part 2]({{ site.baseurl }}/modules/statistics/part-2-testing-differences/).

---

## What's next

[Part 2: Testing differences]({{ site.baseurl }}/modules/statistics/part-2-testing-differences/) covers hypothesis testing, and how to test differences in diversity and abundance.
