---
title: "Part 3: Multiple comparisons and effect sizes"
---

# Part 3: Multiple comparisons and effect sizes
{: .no_toc}

<div class="panel" markdown="1">
**Format:** lecture notes, no software needed.<br>
**Author:** Eliana Ibrahimi, University of Tirana, Albania.<br>
**Before you start:** this page builds on [Part 2]({{ site.baseurl }}/modules/statistics/part-2-testing-differences/), in particular type I errors and p-values.
</div>

**On this page**

* TOC
{:toc}

---

## 1. One test at α = 0.05 is safe. A thousand tests is not.

| | Family-wise error rate (FWER) | False discovery rate (FDR) |
|---|---|---|
| **Controls** | P(at least one false positive), anywhere | Expected proportion of false positives among significant results |
| **Correction** | Bonferroni (or Holm, a less conservative stepwise version) | Benjamini-Hochberg |
| **Best when** | One false positive would be a serious problem (e.g. a confirmatory claim) | You expect many true effects and can tolerate a small, known fraction of false leads |
| **Trade-off** | Conservative: real effects get missed as m grows | Sacrifices some FWER control for much greater power |

Exploratory 16S or shotgun screens of hundreds of taxa are almost always FDR territory. Benjamini-Hochberg dominates this field.

### Bonferroni controls FWER; Benjamini-Hochberg controls FDR

<div class="panel" markdown="1">
**Bonferroni (FWER, conservative):** α<sub>adjusted</sub> = α / m, where m is the number of tests.

Guarantees FWER ≤ α. Simple, but extremely conservative when m is large: real effects get missed.
</div>

<div class="panel" markdown="1">
**Benjamini-Hochberg (FDR, the omics standard):** rank the m p-values from smallest to largest, p<sub>(1)</sub> ≤ ... ≤ p<sub>(m)</sub>. Find the largest rank k such that p<sub>(k)</sub> ≤ (k / m) × α, and reject the hypotheses with ranks 1 to k.

Controls the expected FDR at α. More power than Bonferroni: the standard choice for omics-scale testing.
</div>

*Benjamini & Hochberg (1995), J R Stat Soc B: the original FDR paper, one of the most cited papers in statistics.*

---

## 2. Worked example: Benjamini-Hochberg on 10 taxa

Rank the p-values from smallest to largest, compare each one to its own threshold, and find the last one that still passes.

| Rank (i) | Taxon | p-value | BH threshold (i/10 × 0.05) | Result |
|---:|---|---:|---:|---|
| 1 | Taxon 1 | 0.001 | 0.005 | Significant |
| 2 | Taxon 2 | 0.004 | 0.010 | Significant |
| 3 | Taxon 3 | 0.006 | 0.015 | Significant |
| 4 | Taxon 4 | 0.011 | 0.020 | Significant |
| 5 | Taxon 5 | 0.019 | 0.025 | Significant |
| 6 | Taxon 6 | 0.028 | 0.030 | Significant |
| 7 | Taxon 7 | 0.045 | 0.035 | Not significant |
| 8 | Taxon 8 | 0.060 | 0.040 | Not significant |
| 9 | Taxon 9 | 0.130 | 0.045 | Not significant |
| 10 | Taxon 10 | 0.290 | 0.050 | Not significant |

6 of 10 taxa remain significant after BH correction, against 7 that would pass an uncorrected p < 0.05 cutoff. Taxon 7 (p = 0.045) looks significant alone, but fails once you account for testing 10 hypotheses.

---

## 3. Statistically significant is not the same as biologically important

1. A p-value says nothing about how big or meaningful the difference is.
2. With enough samples, even a practically irrelevant difference becomes "statistically significant".
3. Effect size answers the question a p-value can't: how large is the difference?
4. Always report both the p-value and an effect size.

| | Significant | Not significant |
|---|---|---|
| **Large effect** | Real and important | Likely underpowered |
| **Small effect** | Statistically, not practically, important | Nothing here |

### The exact same tiny effect, at different sample sizes

Fixed throughout: a 0.3-percentage-point difference in relative abundance, d = 0.15 (most people would call this irrelevant).

| n per group | p-value | Significant at α = 0.05? |
|---:|---:|---|
| 10 | 0.74 | No |
| 50 | 0.45 | No |
| 100 | 0.29 | No |
| 500 | 0.018 | Yes |
| 1,000 | 0.0008 | Yes |

Push n to 5,000 and the same effect reaches p < 1×10<sup>−13</sup>.

---

## 4. Effect size measures

| Measure | Use it for | Rough interpretation |
|---|---|---|
| Cohen's d / Hedges' g | Comparing means of a continuous variable (diversity index, metabolite level) | 0.2 small, 0.5 medium, 0.8 large |
| Log2 fold change | Per-taxon differential abundance | LFC = 1: the taxon is 2× more abundant |
| PERMANOVA R² | Community-level (beta diversity) comparisons | Share of total community variance explained by the factor |
| Cliff's delta / rank-biserial r | Non-parametric comparisons (Wilcoxon / Mann-Whitney) | 0.11 small, 0.28 medium, 0.43 large |
| Odds ratio / risk ratio | Presence/absence or binary outcomes | 1 = no association; different from 1 = significant [TO CHECK] |

---

## 5. A confidence interval does more work than a p-value

1. A 95% confidence interval (CI) on your effect size shows both its magnitude and your uncertainty about it; a p-value collapses both into a single yes/no.
2. A narrow CI far from zero and a wide CI that barely excludes zero can share the same p-value.
3. For multiple comparisons, adjust the CIs too: an unadjusted CI is just as optimistic as an unadjusted p-value.

<figure>
  <img src="{{ site.baseurl }}/modules/statistics/images/confidence-intervals.png" alt="Three confidence intervals around zero: narrow and far from zero, wide and crossing zero, narrow and near zero">
  <figcaption>Three ways a confidence interval can look: narrow and far from zero (precise and clearly real), wide and crossing zero (could be nothing, could be large), narrow and near zero (real, but tiny).</figcaption>
</figure>

---

## 6. Software to implement the analysis

R packages for microbiome analysis, most of them available from [Bioconductor](https://www.bioconductor.org/):

- mia
- phyloseq
- vegan
- biom
- DESeq and DESeq2
- metagenomeSeq
- ANCOM
- ALDEx and ALDEx2

---

## 7. Take home

1. Check normality and equal variance before defaulting to a t-test or ANOVA: microbiome data usually fails both.
2. **Alpha diversity:** rank-based tests, or mixed models for repeated measures. **Beta diversity:** PERMANOVA on the distance matrix. Ordination shows the pattern; it doesn't test it.
3. **Differential abundance** needs compositional-aware tools (ALDEx2, ANCOM-BC), not per-taxon t-tests, and not rarefying.
4. **Correct for multiple comparisons**, and know whether you're controlling FWER (Bonferroni) or FDR (Benjamini-Hochberg).
5. A p-value and an effect size answer different questions: **report both, every time**, even after correction.

---

## 8. Resources and courses

| Resource | Link |
|---|---|
| Fundamentals of Biostatistics with R | [github.com/e-ibrahimi/Fundamentals-of-Biostatistics-with-R-](https://github.com/e-ibrahimi/Fundamentals-of-Biostatistics-with-R-) |
| Generalized linear models with R | [github.com/e-ibrahimi/Generalized-Linear-Models-with-R](https://github.com/e-ibrahimi/Generalized-Linear-Models-with-R) |
| Beginner Microbiome Analysis | [bioinformaticsdotca.github.io/BMB_2025](https://bioinformaticsdotca.github.io/BMB_2025/) |
| Advanced Microbiome Analysis | [bioinformaticsdotca.github.io/AMB_2025](https://bioinformaticsdotca.github.io/AMB_2025/) |
| Orchestrating Microbiome Analysis with Bioconductor | [microbiome.github.io/OMA](https://microbiome.github.io/OMA/resources.html#data-containers-1) |
| Ibrahimi, E., Wolford, B.N. Ten essential tips for robust statistics in cell biology. Nat Cell Biol 27, 1884–1886 (2025) | [doi.org/10.1038/s41556-025-01801-y](https://doi.org/10.1038/s41556-025-01801-y) |

---

## References

References as cited in the lecture slides:

- Bardenhorst et al. (2022)
- Benjamini & Hochberg (1995), J R Stat Soc B
- Fernandes et al. (2014), Microbiome
- Gloor et al. (2017), Frontiers in Microbiology. [doi.org/10.3389/fmicb.2017.02224](https://doi.org/10.3389/fmicb.2017.02224)
- Jurgens (2018)
- Kelley et al. (2016), PLoS One, 11(1)
- Love, Huber & Anders (2014), Genome Biology
- McMurdie & Holmes (2014), PLoS Computational Biology. [doi.org/10.1371/journal.pcbi.1003531](https://doi.org/10.1371/journal.pcbi.1003531)
- Robinson, McCarthy & Smyth (2010), Bioinformatics
- Schloss (2023), mSphere. [doi.org/10.1128/msphere.00355-23](https://doi.org/10.1128/msphere.00355-23)
- Wu et al. (2016), Gut, 65(1)
- Xia et al. (2018), Springer Series in Statistics

[TO BE ADDED: full references]
