---
title: "Part 2: Testing differences"
---

# Part 2: Testing differences
{: .no_toc}

<div class="panel" markdown="1">
**Format:** lecture notes, no software needed.<br>
**Author:** Eliana Ibrahimi, University of Tirana, Albania.<br>
**Before you start:** this page builds on [Part 1]({{ site.baseurl }}/modules/statistics/part-1-exploring-microbiome-data/), in particular compositional data and diversity measures.
</div>

**On this page**

* TOC
{:toc}

---

## 1. A refresher on hypothesis testing

Statistical hypothesis testing is defined as assessing the evidence provided by the data against the null hypothesis. It follows four steps:

1. **Formulate the hypotheses:** H<sub>0</sub>, the null hypothesis, and H<sub>1</sub>, the alternative.
2. **Collect the data** and summarize them.
3. **Compute the test statistic.**
4. **Compute the p-value** and make a decision.

- The null hypothesis (H<sub>0</sub>) is "no real difference".
- A test statistic (t, U, F...) is just a standardized distance between what you observed and what H<sub>0</sub> predicts. The p-value is that distance translated into a probability.

### Two ways a test can be wrong

Type I and type II errors are the α and the β from the sample-size calculation in the [Experimental design module]({{ site.baseurl }}/modules/experimental-design/part-2-power-and-sample-size/).

| | Don't reject H<sub>0</sub> | Reject H<sub>0</sub> |
|---|---|---|
| **H<sub>0</sub> true** | Correct: true negative | Type I error (α): false positive |
| **H<sub>0</sub> false** | Type II error (β): false negative | Correct: power = 1 − β |

---

## 2. Microbiome data usually fails normality and equal-variance checks

Microbiome data are **compositional**, **sparse** (many zeros), **overdispersed** and **non-normal**.

| Parametric tests | Non-parametric tests |
|---|---|
| Assume a normal distribution and equal variance across groups | Assume nothing about the shape of the distribution |
| Paired / unpaired t-test (2 groups) | Wilcoxon signed-rank (paired, 2 groups) |
| One-way ANOVA (3+ groups) | Mann-Whitney U / Wilcoxon rank-sum (unpaired, 2 groups) |
| | Kruskal-Wallis (3+ groups) |

### Checking assumptions, and letting your design pick the test

1. **Shapiro-Wilk** tests normality directly; a quick histogram or Q-Q plot does the same job by eye.
2. **Levene's test** checks equal variance across groups. Unequal variance alone can distort a standard t-test's p-value, even when means genuinely differ.

---

## 3. Comparing diversity indices between groups

Shannon, Simpson and richness are single numbers per sample: treat them like any continuous variable, carefully.

1. Alpha diversity indices (Shannon, Simpson, observed richness) collapse a whole community into one number per sample.
2. For independent samples with non-normal or skewed values, use the **Wilcoxon rank-sum** test (2 groups) or the **Kruskal-Wallis** test (3+ groups); no distributional assumption is required.
3. Report the median difference and a non-parametric effect size alongside the p-value (see [Part 3]({{ site.baseurl }}/modules/statistics/part-3-multiple-comparisons-and-effect-sizes/)).

*Example in the lecture: analysis of Chao1 alpha diversity measures, Xia et al., 2018, Springer Series in Statistics.*

---

## 4. When the same vessel is sampled more than once

<figure>
  <img src="{{ site.baseurl }}/modules/statistics/images/repeated-sampling.png" alt="Three vessels, A, B and C, each sampled at day 0, day 3 and day 7">
  <figcaption>Three vessels, each sampled three times: nine samples, but only three independent units.</figcaption>
</figure>

1. Treating all 9 points as 9 independent samples overstates your effective sample size and can bias the test.
2. A **linear mixed-effects model** adds a random intercept per vessel: `diversity ~ treatment + (1 | vessel)`. The vessel term absorbs each vessel's baseline.
3. A **repeated-measures ANOVA** is a special case for normally distributed data; for skewed data, use a mixed model on rank-transformed values.

---

## 5. PERMANOVA tests whole-community differences

Ordination shows you the pattern. PERMANOVA is what actually tests it.

1. Distances between samples come from Bray-Curtis, Jaccard or UniFrac (a full matrix, not one number).
2. PERMANOVA partitions the variance in the distance matrix by group, using **permutation** rather than a theoretical distribution to get a p-value.
3. It is flexible to the dissimilarity measure, makes no assumption of multivariate normality, and is not sensitive to differences in correlation structure among groups. It can include random effects, interaction terms and hierarchical structures. [TO CHECK]

*Example in the lecture: slight differences between the gut microbiota of omnivores and vegans, significant with unweighted UniFrac (PERMANOVA, p = 0.007) but not with the weighted analysis (p = 0.15). Wu et al., Gut, 2016, 65(1).*

### Which distance metric, for which question

| Metric | Uses abundance? | Weights rare taxa | Good default when... |
|---|---|---|---|
| Jaccard | No, presence/absence only | Heavily (every taxon counts equally) | You care which taxa are present or absent, not how much |
| Bray-Curtis | Yes, abundance-weighted | Lightly | You care about shifts in the dominant community structure |
| UniFrac (unweighted) | No, presence/absence + tree | Heavily | You care which lineages are present, using phylogeny |
| UniFrac (weighted) | Yes, abundance + tree | Lightly | You care about abundance-weighted phylogenetic shifts |

PERMANOVA doesn't care which distance matrix it receives, only that the choice matches your question.

### ANOSIM

ANOSIM is a non-parametric test which operates on a **ranked** dissimilarity matrix. The null hypothesis is that the similarities within sites are smaller than or equal to the similarities between sites.

*Example in the lecture: testing the association of microbiome composition between treatments and among time points within treatments, using weighted and unweighted UniFrac distances. Kelley et al., 2016, PLoS One, 11(1).*

---

## 6. Compositional data analysis

Don't want to throw away data?

- Don't rarefy.
- Apply a **CLR (centred log-ratio) transformation**:
  - it balances all features and avoids dominance by high-abundance taxa;
  - it requires the addition of "pseudocounts": zeros are converted to some "small" number.

### Purpose-built methods for compositional data

1. Purpose-built methods model the compositional structure directly instead of testing raw relative abundances one taxon at a time. Their mechanics are beyond the scope of this module, but know they exist and why they're preferred.
2. The more important choice is using one of them at all, instead of a naive per-taxon test.

| Operation | Standard approach | Compositional approach |
|---|---|---|
| Normalization | Rarefaction, "DESeq" | Centred log-ratio (CLR) |
| Distance | Bray-Curtis, UniFrac, Jensen-Shannon | Aitchison |
| Ordination | PCoA (abundance) | PCA (variance) |
| Multivariate comparison | PERMANOVA, ANOSIM | PERMANOVA, ANOSIM |
| Correlation | Pearson, Spearman | SparCC, SpiecEasi |
| Differential abundance | metagenomeSeq, LEfSe, DESeq | ALDEx2, ANCOM |

*Love, Huber & Anders (2014), Genome Biology (DESeq2); Robinson, McCarthy & Smyth (2010), Bioinformatics (edgeR); Fernandes et al. (2014), Microbiome (ALDEx2).*

### Converting relative abundance to absolute abundance

1. Quantify the number of cells or the amount of DNA in each sample:
   - qPCR
   - flow cytometry
   - spike-in of a known number of cells or amount of DNA
2. Multiply each sample by its quantity to obtain absolute numbers, and avoid compositionality.

---

## What's next

[Part 3: Multiple comparisons and effect sizes]({{ site.baseurl }}/modules/statistics/part-3-multiple-comparisons-and-effect-sizes/) covers what happens when you run many tests at once, and how to report results.
