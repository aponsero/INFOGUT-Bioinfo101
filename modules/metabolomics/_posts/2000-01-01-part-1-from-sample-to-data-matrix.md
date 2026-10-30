---
title: "Part 1: From sample to data matrix"
---

# Part 1: From sample to data matrix
{: .no_toc}

<div class="panel" markdown="1">
**Format:** lecture notes, no software needed.<br>
**Estimated time:** 45 minutes.<br>
**Author:** Anna Mascellani Bergo, Czech University of Life Sciences Prague.
</div>

**On this page**

* TOC
{:toc}

---

## 1. The guiding principle

<div class="panel" markdown="1">
Start with the biological question, design the required evidence, protect its quality, and only then build the story.
</div>

---

## 2. Why is the workflow important?

The quality of a biological conclusion depends on the whole workflow. A sophisticated instrument cannot recover information lost through poor sampling, degradation, an unsuitable extraction, or an experimental design that confounds treatment with processing batch. Quality must be protected from collection through storage, preparation, acquisition, processing and interpretation. A numerical table is the outcome of these decisions, rather than an independent description of biology.

---

## 3. What is metabolomics?

**Metabolomics is the systematic measurement of metabolites in biological samples.** Metabolites are small molecules, often below approximately 1,000 Da, that participate in or result from biochemical reactions.

Metabolites can originate from the experimental substrate, microbial metabolism, host metabolism, or interactions among these sources. In a colon-model experiment, a compound may already be present in the medium, substrate or inoculum before fermentation starts. Its detection after incubation does not automatically establish that it was newly produced by the microorganisms.

Metabolomics is close to the observable phenotype because metabolite concentrations can change with microbial activity, diet, environmental conditions and experimental treatment. It therefore helps us describe the **chemical state of a sample at the time of measurement**.

---

## 4. What information does a colon model produce?

A colon-model experiment can generate several complementary layers of data.

| Data layer | Examples | What it helps us understand |
|---|---|---|
| Experimental metadata | Treatment, concentration, incubation time, donor, vessel and replicate | What was done, to which sample, and under which conditions |
| Microbiome data | Taxa, genes and metabolic pathways | Community composition and functional potential |
| Metabolite data | Substrates remaining and fermentation products | The measured chemical phenotype |
| Phenotypic or functional measurements | Measurements selected for the experimental question | Functional outcomes and the context of the other data layers |

These layers become useful together only when their sample identifiers and experimental context can be matched correctly.

### Why measure metabolites if we know which microorganisms are present?

**Presence does not mean activity.** A microorganism may be present but metabolically inactive under the conditions being studied.

**Potential does not confirm production.** Genes associated with butyrate production indicate a capacity for that process. Measuring butyrate provides evidence about its concentration and, with appropriate baseline measurements and controls, its accumulation.

**Similar communities can behave differently.** Differences in substrate availability, microbial interactions or environmental conditions can produce different metabolite profiles even when community composition appears similar.

### Functional potential versus observed phenotype

| Microbiome sequencing | Metabolomics |
|---|---|
| Which microorganisms and genes are present? | Which small molecules are present? |
| What could the community potentially do? | What chemical phenotype is observed? |
| Describes composition and, depending on the sequencing approach, functional potential | Describes the measured metabolic phenotype |

The presence of a species or a metabolic gene does not prove that a pathway was active. Equally, the presence of a metabolite does not identify the organism that produced it. Taxonomic profiling and direct measurement of genes also provide different levels of evidence about functional potential.

<div class="panel" markdown="1">
**Concentration is not a direct measurement of flux.** A concentration reflects the balance of inputs, production, consumption and losses up to the sampling time. A compound can be produced and then consumed by another organism. A low final concentration therefore does not necessarily mean low production.
</div>

---

## 5. Different metabolic routes can lead to the same compound

Adding metabolomics does not remove every ambiguity. Several organisms or pathways can contribute to the same compound, and microorganisms can use products released by other community members. This exchange is often called **cross-feeding**.

### Example 1: increased butyrate, propionate and total SCFAs

Suppose an experimental condition shows higher butyrate, propionate and total short-chain fatty acids (SCFAs) than the control. The direct observation is higher measured concentrations of these products under that condition. This could be caused by different species such as *Ruminococcus*, *Bifidobacterium*, *Roseburia*, *Faecalibacterium* and *Eubacterium/Anaerobutyricum*. This is not a one-to-one assignment of each organism to each measured acid: organisms may contribute directly or through the substrates they provide to others.

The observed pattern alone cannot tell us which organism produced a compound, which route dominated, or whether the difference reflects increased production, reduced consumption, or both. Interpretation needs the substrate, incubation time, baseline, controls and other available data.

**Total SCFA also needs a definition.** Record which compounds were included and their units. The six organic acids in the [demo]({{ site.baseurl }}/modules/metabolomics/part-2-hands-on-demo-in-r/) are not interchangeable with a standard "total SCFA" measurement; do not simply sum all six and label the result total SCFA.

### Example 2: aromatic amino acids and fermentation products

The second example connects phenylalanine (Phe), tyrosine (Tyr) and tryptophan (Trp) with products such as phenylacetate, phenylpropionate, *p*-cresol and indole. *Clostridium* species, *Peptostreptococcus* species, *Bacteroides* species and *Escherichia coli*/Enterobacteriaceae are examples relevant to these transformations.

Again, a metabolite is not a unique label for a taxon or a pathway. A pattern involving substrates and several products is more informative than an isolated change, especially when interpreted alongside the experimental conditions.

### Saccharolytic and proteolytic fermentation profiles

The overall metabolic picture can often be interpreted in relation to broad metabolic processes. For in vitro gut models, one example is the balance between saccharolytic and proteolytic fermentation profiles.

**Saccharolytic fermentation** involves microbial use of carbohydrates and can generate organic acids, including SCFAs, and gases. **Proteolysis** breaks proteins into peptides and amino acids; subsequent amino-acid metabolism can generate a range of fermentation products, among them branched-chain fatty acids (BCFA), phenols and indoles.

| Broad process | Substrates considered | Examples of associated measurements |
|---|---|---|
| Saccharolytic fermentation | Carbohydrates | Lactate, acetate, propionate, butyrate and gases |
| Proteolytic fermentation and amino-acid metabolism | Proteins, peptides and amino acids | Branched-chain fatty acids, ammonia, amines, phenolic compounds, indole-related compounds and sulfides |

These are broad profiles, not mutually exclusive categories. SCFAs can occur in more than one metabolic context, and the two types of metabolism can coexist in a community.

<div class="panel" markdown="1">
Instead of assigning a pathway from one metabolite, look for a pattern across several metabolites and consider the experimental context.
</div>

<figure>
  <img src="{{ site.baseurl }}/modules/metabolomics/images/fermentation-profiles.jpg" alt="Products of saccharolytic fermentation (lactate, acetate, propionate, butyrate) and proteolytic fermentation (amines, phenols, sulfides, ammonia, BCFA), with SCFA, biomass and gases shared">
  <figcaption>Saccharolytic and proteolytic fermentation profiles share some outcomes. Figure reproduced from Boven et al. (2024), DOI 10.1080/10408398.2024.2352523.</figcaption>
</figure>

---

## 6. From the biological question to the analytical strategy

The interpretation problem leads directly to a practical question: **what should we measure?**

If we already know which compounds are needed to answer our question, a focused assay may be appropriate. If we need to explore a wider response, broader profiling may be useful. The strategy determines which part of the sample's chemistry will be visible.

### Targeted, semi-targeted and untargeted approaches

| Strategy | Main purpose | What is measured | Main consideration |
|---|---|---|---|
| Targeted | Test a focused question | Specific, predefined metabolites | Optimise and validate measurement of the chosen compounds |
| Semi-targeted | Combine a defined focus with some exploration | A selected set of known metabolites, sometimes alongside additional signals | State what the method actually covers; usage of this term varies |
| Untargeted | Explore a broad chemical response | Many detectable features within the method's analytical window | Many features may remain unidentified or only provisionally annotated |

Targeted assays can offer high sensitivity and reliable quantification for the selected compounds. Untargeted profiling expands the search space, but a detected feature is not necessarily a distinct, identified metabolite.

### Can one method measure everything?

**No single method sees it all.** Metabolites differ in polarity, volatility, solubility, molecular structure and concentration. Sample preparation and instrument settings further define what can be detected.

"Untargeted" therefore means broad exploration within an analytical window, not complete measurement of every compound in a sample. Complementary methods can expand coverage.

---

## 7. Choosing the analytical platform

This section introduces proton nuclear magnetic resonance spectroscopy (¹H NMR), liquid chromatography–mass spectrometry (LC–MS) and gas chromatography–mass spectrometry (GC–MS). The demo in Part 2 uses targeted high-performance liquid chromatography (HPLC).

<figure>
  <img src="{{ site.baseurl }}/modules/metabolomics/images/analytical-platforms.jpg" alt="Comparison of NMR and mass spectrometry (LC-MS and GC-MS) platforms">
  <figcaption>Key analytical peculiarities of the most commonly used techniques in metabolomics.</figcaption>
</figure>

### What each platform measures

**¹H NMR** records signals from hydrogen nuclei in different chemical environments. Spectra show signal intensity against chemical shift, expressed in parts per million (ppm). Signal positions, splitting patterns and integrals help with assignment and quantification.

**LC–MS** separates compounds by liquid chromatography and detects ions by mass spectrometry. A signal is described by retention time, mass-to-charge ratio (*m/z*) and intensity. Fragmentation spectra (MS/MS), where available, add structural information.

**GC–MS** separates volatile compounds, or compounds made suitable for analysis through derivatisation, before mass-spectrometric detection. Retention behaviour and fragmentation patterns support identification.

**HPLC** is a separation method and is not synonymous with LC–MS. An HPLC assay can use another detector. In a targeted organic-acid assay, calibrated chromatographic peak responses can be converted into a table of compound measurements. The detector and calibration details must come from the method documentation.

### Platform comparison

Sensitivity and coverage depend strongly on the sample, preparation, instrument and method; the numerical ranges below are illustrative, not guaranteed performance specifications.

| Aspect | ¹H NMR | LC–MS | GC–MS |
|---|---|---|---|
| Indicative sensitivity | Often µM for routine profiling | Can reach nM–pM for suitable analytes and methods | Can reach nM for suitable volatile or derivatised compounds |
| Illustrative metabolite coverage | 30–80 | 200 to more than 1,000 | 100–500 |
| Quantification | Signal integrals can be quantitative under appropriate acquisition and referencing conditions | Usually needs appropriate standards and calibration for absolute concentrations | Usually needs appropriate standards and calibration for absolute concentrations |
| Sample preparation | Often relatively simple and fast | Method-dependent; often moderately complex | Can be more complex, particularly when derivatisation is required |
| Reproducibility and matrix effects | Often highly reproducible; acquisition conditions and matrix-related spectral changes still matter | Matrix effects and instrumental drift require careful control | Can be highly reproducible when standardised; derivatisation may introduce variation |
| Structural information | Chemical shifts and coupling patterns support assignment; overlap can limit interpretation | MS/MS supports annotation; isomer discrimination can be difficult | Fragmentation patterns and spectral libraries can strongly support identification |

---

## 8. Sample preparation and acquisition

The sample journey follows five stages: **collection, storage, preparation, acquisition and raw data**. Decisions at every stage affect the final matrix.

<figure>
  <img src="{{ site.baseurl }}/modules/metabolomics/images/sample-journey.jpg" alt="The sample journey: collection, storage, preparation, acquisition and raw data">
</figure>

### Design, randomisation and confounding

Plan the experiment around the question and the comparisons needed to answer it. Consider sample size and power before collecting data, and distinguish **biological replicates** from **technical replicates** (see the [Experimental design module]({{ site.baseurl }}/modules/experimental-design/programme/)).

Biological replicates represent independent biological or experimental units at the level relevant to the question. Technical replicates repeat a measurement or processing step on the same underlying material. Repeated injections do not create additional independent biological samples.

Randomise preparation and acquisition where appropriate, and distribute conditions across batches. Record potential confounding factors. If every control is measured in one batch and every treatment in another, a treatment difference may be indistinguishable from a batch effect.

### Collection and storage

Record collection time, location, preservation method, storage conditions and relevant handling steps. Snap freezing can be part of a validated protocol, but the preservation approach should suit the sample and intended measurements.

Light, temperature, microorganisms and enzymes can alter samples after collection. Metabolites do not all degrade at the same rate, so inconsistent handling may change composition as well as overall abundance.

The practical message is to ask **how the sampling was performed**, rather than only whether someone followed the protocol. Detailed records reveal deviations that a yes/no answer can miss.

### Extraction and measurement

An extraction must recover the compounds relevant to the question. Optimisation may be needed, and a protocol that works for one chemical class may not work for another.

Record extraction batches, acquisition batches, internal standards, quality-control samples, run order and any deviations from the protocol. Continue to consider degradation during preparation and time spent awaiting measurement.

| Quality-related material or record | What it helps assess |
|---|---|
| Blank | Background signals and contamination introduced by reagents or processing |
| Quality-control sample (QC) | Analytical consistency, repeatability and drift |
| Internal standard | Aspects of preparation or measurement performance, depending on when and how it is added |
| Batch and run-order metadata | Whether technical patterns overlap with experimental conditions |

QCs are particularly important for following LC–MS performance over a run. They do not replace independent biological replication.

---

## 9. What does raw data look like?

Raw instrument output is not yet the table used for statistics.

| Platform | Raw-data representation | Processing needed before a measurement table |
|---|---|---|
| LC–MS | Signals distributed across retention time, *m/z* and intensity | Peak detection, integration, alignment and feature correspondence across samples |
| GC–MS | Chromatographic peaks with associated mass spectra | Peak detection, separation of overlapping contributions where needed, integration and alignment |
| NMR | Spectra with signal intensity against chemical shift | Spectral processing, referencing, appropriate alignment and integration, binning or compound fitting |
| Targeted HPLC | Chromatographic detector responses | Peak assignment, integration and calibration for the chosen acids |

---

## 10. From analytical signals to a data matrix

For a **targeted workflow**, predefined compounds guide signal assignment and integration. QC inspection and quality filtering establish which measurements are usable. Appropriate normalisation, if needed, precedes statistical analysis and biological interpretation.

For an **untargeted workflow**, processing first creates features. QC inspection, blank assessment and quality filtering remove unreliable or background signals. Further filtering and any justified normalisation prepare the matrix for analysis. Annotation may occur at several stages and may be revisited when interpreting features of interest.

These steps are iterative: an unexpected result may lead us back to the chromatogram or spectrum. Do not treat a processed matrix as unquestionable simply because software exported it.

### The three linked tables

A usable metabolomics dataset normally needs three linked components.

| Table | One row represents | Typical columns | Link to the other tables |
|---|---|---|---|
| Sample metadata | A sample | Sample ID, condition, donor, time point, replicate, batch and run order | Sample ID |
| Measurement matrix | A sample in the usual samples-by-variables layout | Sample ID and metabolite or feature measurements | Sample ID and feature/compound identifiers |
| Annotation table | A feature or compound | Feature ID, proposed identity, retention time, *m/z*, unit and identification confidence, as applicable | Feature/compound identifier |

In a small targeted CSV, metadata and measurements may be stored together. Conceptually, they still have different roles. A treatment label is not a metabolite, and a replicate number should not accidentally enter a numerical metabolite matrix.

Sample identifiers should be unique at the intended row level and should match across tables. The annotation table explains what each numerical variable means; the metadata explains the sample and comparison it belongs to.

### Features and annotation confidence

A **feature** is a reproducible analytical signal. Several features can originate from one compound, and overlapping compounds can contribute to a signal. Feature counts are therefore not necessarily metabolite counts.

According to the Metabolomics Standards Initiative (MSI) (Sumner et al. (2007), DOI: 10.1007/s11306-007-0082-2), confidence levels are attributed as follows:

| Level | Description | Evidence represented |
|---|---|---|
| 1 | Identified compound | Confirmation against an authentic standard using at least two independent, orthogonal properties measured under identical conditions |
| 2 | Putatively annotated compound | Assignment from physicochemical properties or spectral/library evidence without the level-1 standard confirmation |
| 3 | Putatively characterised compound class | Evidence supports a chemical class rather than a specific compound identity |
| 4 | Unknown compound | A distinguishable analytical signal without an assigned chemical identity |

Carry annotation confidence into interpretation. A tentative name should not silently become a confirmed compound in a pathway claim.

### Missing values and filtering

An untargeted table can contain up to 20,000 features or more, with missing entries and variables that need filtering. A missing value is not automatically a biological zero. It can reflect a signal below the detection limit, a processing failure, interference or another measurement problem.

Before deciding how to handle missing values, ask whether they cluster by feature, condition, batch or run order. Inspect blank responses and QC performance where these are available. Document the criteria used to remove features or samples, and avoid applying an unexplained universal threshold.

Filling in missing values can be appropriate for some analyses, but it requires an explicit assumption about why values are missing. It cannot repair a fundamentally inadequate measurement.

<figure>
  <img src="{{ site.baseurl }}/modules/metabolomics/images/feature-table.jpg" alt="Spreadsheet of an untargeted feature table with many empty cells">
  <figcaption>An example feature table before filtering. Large untargeted tables can contain many thousands of features.</figcaption>
</figure>

---

## 11. Normalisation: making samples comparable

**Sample-wise normalisation** is a row operation. One factor rescales all variables in a sample to account for a difference such as dilution or the amount of material measured.

For a measurement matrix with samples in rows, a simple representation is:

<div class="panel" markdown="1">
x<sub>ij</sub><sup>normalised</sup> = x<sub>ij</sub> / s<sub>i</sub>

where x<sub>ij</sub> is the measurement for variable j in sample i, and s<sub>i</sub> is that sample's normalisation factor.
</div>

For example, consider two aliquots of the same in vitro colon-model fermentation supernatant. If one is diluted twofold and the other fourfold before analysis, the more diluted aliquot will show lower measured concentrations even though both originated from the same biological sample. Multiplying the measured concentrations by their respective dilution factors restores them to the original-supernatant basis, making them comparable. This corrects a known sample-preparation difference; it does not remove genuine differences in metabolite accumulation between experimental conditions.

**Is normalisation always needed? No.** The decision depends on the experimental design, measurement units and scientific question. In a colon model, a change in total metabolite concentration may itself be an outcome of interest. Forcing samples to have equal totals could remove that information.

Sample-wise normalisation is also different from transformation or variable-wise scaling. These operations answer different statistical needs and should be documented separately. No processing step guarantees that every remaining difference is biological.

---

## 12. The reality check before interpretation

Following the "reality check" from Limonciel's book *The STORY Principle*, we establish whether the data are fit for the question before building a biological explanation.

| Dimension | Questions to ask |
|---|---|
| Data structure | Does the design contain the needed comparison? Are samples independent, paired or repeated? Do IDs and metadata match? |
| Data quality | Are measurements reliable? Are missingness, blanks, QC behaviour, drift and batch effects acceptable for the intended analysis? |
| Data quantity | Are there enough independent observations and sufficient analytical coverage to support the proposed conclusion? |
| Fitness for purpose | Which questions can this dataset answer, and which remain beyond it? |

A visible difference is a starting observation. Its interpretation depends on measurement reliability, within-condition variability, experimental independence and adequate replication. Descriptive exploration and statistically supported treatment effects are different levels of evidence.

---

## What's next

[Part 2: Hands-on demo in R]({{ site.baseurl }}/modules/metabolomics/part-2-hands-on-demo-in-r/) applies these ideas to a small targeted organic-acid dataset from a colon-model study.
