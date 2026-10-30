---
title: "Part 2: Hands-on demo in R"
---

# Part 2: Hands-on demo in R
{: .no_toc}

<div class="panel" markdown="1">
**Estimated time:** 45 to 60 minutes.<br>
**You need:** R and RStudio with the `tidyverse` package, and the demo files: [download the demo dataset and R script (zip)]({{ site.baseurl }}/modules/metabolomics/data/metabolomics-demo.zip).<br>
**Before you start:** this page builds on [Part 1]({{ site.baseurl }}/modules/metabolomics/part-1-from-sample-to-data-matrix/), and assumes basic R and tidyverse skills (see the [R and RStudio module]({{ site.baseurl }}/modules/r/programme/)).<br>
**Author:** Anna Mascellani Bergo, Czech University of Life Sciences Prague.
</div>

**On this page**

* TOC
{:toc}

---

## 1. Study context and teaching subset

This example comes from Garcia-Gutierrez et al. (2026), *A microbiota-informed pipeline for in vitro characterisation of insulinotropic properties of dairy fermentates using static EndoC-βH1 and dynamic microfluidic models*, **Food Research International**, 242, 120207. DOI: [10.1016/j.foodres.2026.120207](https://doi.org/10.1016/j.foodres.2026.120207).

The study digested six dairy fermentates using the INFOGEST protocol, then incubated them with a **pooled faecal inoculum in a microMatrix colon model**. Fermentates were added at 1% and 5% (w/v), in duplicate wells, and incubated for 24 hours at 37 °C with pH maintained at 6.8. Unfed control wells contained diluted faecal slurry without added fermentate (Methods 2.2).

The study combined shallow whole-genome shotgun sequencing (WGS) with targeted HPLC measurements of six organic acids. Cell-free colonic fermentation supernatants were then tested for their effects on **insulin secretion by human EndoC-βH1 cells** in three independent experiments. A separate dynamic microfluidic assay used mouse pancreatic islets. Insulin was measured in these downstream cell assays, not produced or measured as a microbial metabolite in the colon-model wells.

The original data collection contained 26 samples. For this introductory exercise, it was reduced to six samples by retaining:

- Two fermentates, **Ferm1 and Ferm4**, selected for a complete teaching comparison. Both were included in the paper's dynamic assay, which also included Ferm6.
- The **5% concentration** for the two fermentates.
- The **T24 unfed control**, rather than the initial T0 control.
- Two replicates per condition, renamed **Control_R1/R2, A_R1/R2 and B_R1/R2**.

For clearer interpretation, the teaching labels use the paper's fermentate numbers and composition:

| CSV label | Label used in this demo | Protein | Lactose |
|---|---|---:|---:|
| Control | Unfed T24 control | | |
| A | Ferm1: moderate protein, higher lactose | 56% | 30% |
| B | Ferm4: high protein, low lactose | 90% | 0.8% |

Composition refers to the dairy ingredients before digestion (paper, Table 1), not to final concentrations in the fermentation wells. The mapping A to Ferm1 and B to Ferm4 follows the selected products and their matching organic-acid profiles in Figure 3 of the paper; the CSV itself does not encode the original product key.

| Samples | Conditions | Treatment concentration | Control | Replication | Variables |
|---:|---|---|---|---|---|
| 6 | Control, Ferm1, Ferm4 | 5% for Ferm1 and Ferm4 | Unfed, T24 | 2 per condition | 6 organic acids |

### The demo files

| File | Contents | Layout |
|---|---|---|
| `organic_acids_demo.R` | The code on this page | Run section by section in RStudio |
| `data/acids.csv` | Six organic acids: formic, acetic, propionic, lactic, butyric and citric | 6 sample rows; identifiers and acids in columns |
| `data/insulin.csv` | Insulin measurements from three experiments | 18 rows; each sample occurs in experiments 1, 2 and 3 |
| `data/abundance.csv` | Microbiome abundance values for 190 species | Species in rows; the six samples in columns |
| `data/taxonomy.csv` | Taxonomic assignments for the 190 species | Species, kingdom, phylum, class, order, family and genus |

The demonstration uses **`acids.csv`**. The other files provide opportunities to connect data layers later; they are not required to run the organic-acid code below.

<div class="panel" markdown="1">
**Measurement information.** The paper describes **duplicate fermentation wells prepared from a pooled faecal standard** (Methods 2.2.2). These are not two independent donors. The microbiome methods call the pairs technical replicates, while the Discussion describes the SCFA replication as biological (n = 2). For this exercise, the concrete description "duplicate fermentation wells" is more useful than assuming donor-level independence. Replicate labels do not establish pairing between conditions.

The HPLC methods report concentrations in **µg/mL**, subsequently converted to **mM** using each acid's molecular weight (Methods 2.3.3; Figure 3). All acid concentrations and concentration differences in this example are expressed in **mM**.
</div>

---

## 2. What can this small dataset tell us?

The small design is useful for learning to inspect individual measurements, reshape tables and describe patterns. Limited replication is a common practical issue in in vitro work and provides a useful opportunity to discuss the boundary between exploration and inference.

With two observations per condition, show both points rather than hiding them behind a group mean. We can compare observed values, calculate differences and generate hypotheses. Estimating variability precisely, assessing assumptions and generalising beyond this experiment require stronger support.

A formal test is not ruled out simply because there are two observations. Its validity depends on the experimental unit, independence, design and assumptions, and its information and power may be very limited. Treating technical repeats as independent biological samples would overstate replication. This demo therefore uses **descriptive summaries without p-values or claims of significant treatment effects**.

---

## 3. Getting set up

1. [Download the zip file]({{ site.baseurl }}/modules/metabolomics/data/metabolomics-demo.zip) and unzip it. Keep the files in the same arrangement: the script reads `data/acids.csv`.
2. Open `organic_acids_demo.R` in RStudio.
3. Set the working directory to the unzipped folder: **Session > Set Working Directory > To Source File Location**.
4. If needed, install the tidyverse once (this needs an internet connection):

```r
install.packages("tidyverse")
```

You can run the script section by section, or type the code from this page yourself.

---

## 4. Import and inspect the data

Before doing anything else, look at the table and identify the sample identifier, the experimental variables and the metabolite columns.

```r
library(tidyverse)

data_file <- "data/acids.csv"

acids <- read_csv(data_file, show_col_types = FALSE)

# Look at the table: one row per sample, one column per acid (in micrograms/mL)
acids
glimpse(acids)

# How many samples per condition?
acids |>
  count(product)
```

The columns are `sample`, `product`, `replicate`, then one column per acid. There are two samples for each of `Control`, `A` and `B`.

Looking at the table confirms its structure. It does not establish biological independence, correct units or analytical quality: those require experimental records.

### Convert the concentrations to mM

The CSV gives concentrations in µg/mL. Since µg/mL is the same as mg/L, dividing by each acid's molecular weight (g/mol) gives mmol/L, that is mM.

```r
acids <- acids |>
  mutate(
    formic    = formic / 46.03,
    acetic    = acetic / 60.05,
    propionic = propionic / 74.08,
    lactic    = lactic / 90.08,
    butyric   = butyric / 88.11,
    citric    = citric / 192.12
  )
```

The CSV file itself is not changed: only the table in R.

### Give the conditions clearer names

```r
# In the CSV, A is fermentate 1 (Ferm1) and B is fermentate 4 (Ferm4).
acids <- acids |>
  mutate(
    product = case_when(
      product == "A" ~ "Ferm1",
      product == "B" ~ "Ferm4",
      .default = product
    ),
    product = factor(product, levels = c("Control", "Ferm1", "Ferm4"))
  ) |>
  arrange(product, sample)

acids

# Labels used on the plots below ("\n" starts a new line)
product_labels <- c(
  "Control" = "Unfed T24\ncontrol",
  "Ferm1"   = "Ferm1\nModerate protein\nHigher lactose",
  "Ferm4"   = "Ferm4\nHigh protein\nLow lactose"
)
```

The six-sample teaching subset in wide format, acid concentrations in mM:

| sample | product | replicate | formic | acetic | propionic | lactic | butyric | citric |
|---|---|---:|---:|---:|---:|---:|---:|---:|
| Control_R1 | Control | 1 | 753.01 | 41.40 | 4.13 | 50.91 | 4.52 | 3.33 |
| Control_R2 | Control | 2 | 773.86 | 39.43 | 7.67 | 48.61 | 6.64 | 3.43 |
| A_R1 | Ferm1 | 1 | 634.08 | 59.74 | 5.49 | 38.96 | 18.56 | 3.70 |
| A_R2 | Ferm1 | 2 | 638.75 | 53.12 | 8.56 | 29.02 | 9.53 | 5.16 |
| B_R1 | Ferm4 | 1 | 485.20 | 59.33 | 32.83 | 3.97 | 27.05 | 3.50 |
| B_R2 | Ferm4 | 2 | 567.84 | 55.47 | 25.50 | 4.13 | 25.60 | 3.02 |

The sample IDs keep their original names (`A_R1`, `B_R1`...), so they still match the other files.

---

## 5. Wide and long formats

In the **wide format**, one row represents one sample and each acid has its own column. This is convenient for constructing a numerical matrix.

In the **long format**, one row represents one sample–acid measurement. The acid names become values in an `acid` column and the measurements enter a `value` column. Six samples and six acids produce 36 rows.

```r
acids_long <- acids |>
  pivot_longer(
    cols = formic:citric,
    names_to = "acid",
    values_to = "value"
  )

acids_long
```

The first rows of the long-format table (values in mM):

| sample | product | replicate | acid | value |
|---|---|---:|---|---:|
| Control_R1 | Control | 1 | formic | 753.01 |
| Control_R1 | Control | 1 | acetic | 41.40 |
| Control_R1 | Control | 1 | propionic | 4.13 |
| Control_R1 | Control | 1 | lactic | 50.91 |
| Control_R1 | Control | 1 | butyric | 4.52 |
| Control_R1 | Control | 1 | citric | 3.33 |
| Control_R2 | Control | 2 | formic | 773.86 |
| ... | | | | |

**Wide format supports matrix operations. Long format supports grouped summaries and faceted plots.** The information has not changed; only its organisation has changed.

To separate the numerical measurements from the sample metadata:

```r
sample_metadata <- acids |>
  select(sample, product, replicate)

measurement_matrix <- acids |>
  select(sample, formic:citric) |>
  column_to_rownames("sample") |>
  as.matrix()

dim(measurement_matrix)  # 6 samples x 6 organic acids
```

```
[1] 6 6
```

The matrix contains measurements only. Its row names retain the connection to metadata. A separate annotation table would document compound names, units, method and identification evidence; those details should come from the assay records.

---

## 6. Check for missing measurements

```r
acids_long |>
  group_by(acid) |>
  summarise(
    n_expected = n(),
    n_observed = sum(!is.na(value)),
    n_missing  = sum(is.na(value))
  )
```

| acid | n_expected | n_observed | n_missing |
|---|---:|---:|---:|
| acetic | 6 | 6 | 0 |
| butyric | 6 | 6 | 0 |
| citric | 6 | 6 | 0 |
| formic | 6 | 6 | 0 |
| lactic | 6 | 6 | 0 |
| propionic | 6 | 6 | 0 |

There are no missing values in this dataset. If values were missing, inspect their origin before interpreting the plot. A condition mean calculated from one available value is not a mean supported by two measurements. The plotting code below leaves out missing measurements without filling them in.

---

## 7. Organic-acid profiles

Each point represents one observed measurement. The horizontal bar represents the mean of the available measurements within a condition and acid. It is not a confidence interval.

```r
condition_colours <- c(
  "Control" = "#6C7A89",
  "Ferm1"   = "#00A6A6",
  "Ferm4"   = "#EF8354"
)

acids_long |>
  filter(!is.na(value)) |>
  ggplot(aes(x = product, y = value, colour = product)) +
  # one point per measurement, slightly spread so the points do not overlap
  geom_point(size = 3.5, position = position_jitter(width = 0.05, height = 0, seed = 42)) +
  # a horizontal bar at the mean of each condition (not a confidence interval)
  stat_summary(fun = mean, geom = "crossbar", width = 0.34, colour = "#172A3A") +
  # one panel per acid, each with its own y-axis scale
  facet_wrap(~ acid, scales = "free_y", nrow = 2) +
  scale_x_discrete(labels = product_labels) +
  scale_colour_manual(values = condition_colours) +
  labs(x = NULL, y = "Concentration (mM)") +
  theme_bw(base_size = 13) +
  theme(legend.position = "none")
```

<figure>
  <img src="{{ site.baseurl }}/modules/metabolomics/images/organic-acid-profiles.png" alt="Six panels, one per organic acid, showing individual measurements and means for the unfed control, Ferm1 and Ferm4">
  <figcaption>Organic-acid profiles of the unfed T24 control and selected 5% dairy fermentates. Points show individual fermentation wells; horizontal bars show condition means. Each panel has its own y-axis scale. Concentrations are in mM. Fermentate descriptors refer to ingredient composition.</figcaption>
</figure>

Each panel uses its own y-axis scale so that differences within an acid remain visible when acids have very different measurement ranges. In this dataset, formic acid values are much larger than the other acid measurements. With a shared scale, variation in the smaller measurements could be visually compressed.

Compare conditions **within a panel**. Equal-looking vertical gaps in different panels do not imply equal absolute differences. The small horizontal jitter separates overlapping points; it does not alter their measured values.

---

## 8. Duplicate differences versus differences between fermentates

For each acid and condition, define the **duplicate gap** as the absolute difference between R1 and R2. Define the **between-fermentate gap** as the absolute difference between the Ferm1 and Ferm4 means. Both gaps are expressed in mM for each acid.

A duplicate gap is not automatically technical variation: its meaning depends on what was replicated. Here it describes variation between the duplicate fermentation wells, not donor-to-donor variation or HPLC injection repeatability alone. Do not interpret a gap ratio as a test statistic or use it as a threshold for significance.

```r
# Mean and duplicate gap (difference between R1 and R2) for each condition and acid.
# This assumes two measurements per condition, which is the case in this dataset.
acid_summary <- acids_long |>
  group_by(product, acid) |>
  summarise(
    n_observed    = sum(!is.na(value)),
    mean_value    = mean(value, na.rm = TRUE),
    duplicate_gap = max(value, na.rm = TRUE) - min(value, na.rm = TRUE),
    .groups = "drop"
  )

# Absolute difference between the Ferm1 and Ferm4 means, for each acid
fermentate_gaps <- acid_summary |>
  filter(product %in% c("Ferm1", "Ferm4")) |>
  select(acid, product, mean_value) |>
  pivot_wider(names_from = product, values_from = mean_value) |>
  mutate(between_gap = abs(Ferm4 - Ferm1))

# Put both gaps in one table
gap_comparison <- acid_summary |>
  left_join(fermentate_gaps |> select(acid, between_gap), by = "acid")

gap_comparison
```

Condition means and absolute gaps, in mM:

| product | acid | n_observed | mean_value | duplicate_gap | between_gap |
|---|---|---:|---:|---:|---:|
| Control | acetic | 2 | 40.415 | 1.964 | 0.972 |
| Control | butyric | 2 | 5.581 | 2.114 | 12.283 |
| Control | citric | 2 | 3.383 | 0.099 | 1.171 |
| Control | formic | 2 | 763.433 | 20.854 | 109.902 |
| Control | lactic | 2 | 49.760 | 2.292 | 29.944 |
| Control | propionic | 2 | 5.896 | 3.541 | 22.140 |
| Ferm1 | acetic | 2 | 56.430 | 6.621 | 0.972 |
| Ferm1 | butyric | 2 | 14.045 | 9.038 | 12.283 |
| Ferm1 | citric | 2 | 4.432 | 1.459 | 1.171 |
| Ferm1 | formic | 2 | 636.418 | 4.674 | 109.902 |
| Ferm1 | lactic | 2 | 33.994 | 9.941 | 29.944 |
| Ferm1 | propionic | 2 | 7.025 | 3.062 | 22.140 |
| Ferm4 | acetic | 2 | 57.401 | 3.860 | 0.972 |
| Ferm4 | butyric | 2 | 26.328 | 1.448 | 12.283 |
| Ferm4 | citric | 2 | 3.261 | 0.478 | 1.171 |
| Ferm4 | formic | 2 | 526.516 | 82.640 | 109.902 |
| Ferm4 | lactic | 2 | 4.050 | 0.151 | 29.944 |
| Ferm4 | propionic | 2 | 29.165 | 7.328 | 22.140 |

```r
# Points are duplicate gaps, the dashed line is the between-fermentate gap
gap_comparison |>
  ggplot(aes(x = product, y = duplicate_gap, colour = product)) +
  geom_hline(
    data = fermentate_gaps,
    aes(yintercept = between_gap),
    linetype = "dashed",
    colour = "#172A3A"
  ) +
  geom_point(size = 3) +
  facet_wrap(~ acid, scales = "free_y", nrow = 2) +
  scale_x_discrete(labels = product_labels) +
  scale_colour_manual(values = condition_colours) +
  labs(x = NULL, y = "Absolute gap (mM)") +
  theme_bw(base_size = 13) +
  theme(legend.position = "none")
```

<figure>
  <img src="{{ site.baseurl }}/modules/metabolomics/images/duplicate-gaps.png" alt="Six panels comparing duplicate gaps for each condition with the gap between the Ferm1 and Ferm4 means">
  <figcaption>Within-condition duplicate gaps (points) compared with the absolute difference between the Ferm1 and Ferm4 means (dashed line). Each acid has its own y-axis scale. These gaps describe the observed data, not statistical significance.</figcaption>
</figure>

For acetic acid, the Ferm1–Ferm4 mean gap is only about 0.972 mM, while the duplicate gaps for Ferm1 and Ferm4 are 6.621 and 3.860 mM. A small ordering of the means should therefore not be presented as a clear treatment separation. For butyric acid, the Ferm1–Ferm4 mean gap is 12.283 mM, compared with duplicate gaps of 9.038 mM for Ferm1 and 1.448 mM for Ferm4. A larger gap still does not, by itself, establish a statistically supported effect.

---

## 9. Interpretation exercise

Look at the plots. Write down one observation and one limitation before opening the discussion prompts.

1. Which acids appear to differ most among Control, Ferm1 and Ferm4?
2. Which product has the highest observed propionate and butyrate values?
3. Where do the two replicates appear least consistent?
4. What information do you need before interpreting the values?
5. Which conclusions would require more replication?

<details markdown="1">
<summary>Discussion prompts</summary>

In the supplied data, both Ferm4 measurements have higher propionic and butyric acid values and lower lactic acid values than either Control measurement. The mean values for Ferm1 lie between Control and Ferm4 for these three acids. These are observed patterns, not evidence of a particular pathway or organism.

The two Ferm1 butyric-acid concentrations are 18.564 and 9.526 mM, a gap of 9.038 mM. The Ferm4 pair is closer: 27.052 and 25.604 mM, a gap of 1.448 mM. This is why it matters to display individual measurements alongside the mean.

Before making a biological interpretation, confirm the original product key, and inspect calibration and QC information. Remember that the duplicate wells share a pooled faecal inoculum. We know from the teaching subset description that Ferm1 and Ferm4 are the selected 5% conditions and Control is the unfed T24 reference. The data alone do not establish whether an observed difference arose from production, consumption or compounds already present in the fermentates.
</details>

---

## 10. Five take-home messages

1. **Metabolomics measures the small-molecule phenotype** and complements sequencing data.
2. **Targeted, semi-targeted and untargeted methods answer different questions.**
3. **Different platforms generate different signals but similar downstream tables.**
4. **Metadata, annotation and quality checks give the matrix its meaning.**
5. **A visible difference does not establish a statistically supported effect.**

---

## Going further

The zip file also contains the microbiome abundances (`abundance.csv`, `taxonomy.csv`) and the insulin measurements (`insulin.csv`) for the same six samples. Once you have completed the [Microbiome data module]({{ site.baseurl }}/modules/microbiome/programme/), you can try connecting these data layers using the shared sample IDs.
