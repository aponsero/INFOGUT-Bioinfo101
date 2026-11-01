---
title: "Part 6: Microbiome data in R"
---

# Part 6: Microbiome data in R
{: .no_toc}

<div class="panel" markdown="1">
**Estimated time:** 45 to 60 minutes.<br>
**You need:** R and RStudio with the packages `dplyr`, `ggplot2`, `vegan` and `phyloseq` (see [Setup]({{ site.baseurl }}/modules/about/setup/#install-the-r-packages)), and the session 3 notebook and data: [download r-session3.zip]({{ site.baseurl }}/modules/r/data/r-session3.zip).<br>
**Before you start:** this page uses `filter()`, `group_by()`, `summarise()` and `arrange()` from [Part 4]({{ site.baseurl }}/modules/r/part-4-handling-tables-with-dplyr/). The data are described in [Part 3]({{ site.baseurl }}/modules/r/part-3-our-own-data/), and the concepts of relative abundance and diversity in the [Statistics]({{ site.baseurl }}/modules/statistics/programme/) and [Microbiome]({{ site.baseurl }}/modules/microbiome/programme/) modules.<br>
**Authors:** Andrés Méndez Hurtado, Adrián Martín Segura, Alba Pérez Cuervo, Laura Judith Marcos Zambrano, Matti Ruuskanen.
</div>

Relative abundance, diversity and simple statistics, on 2 replicates per condition (Control, A, B).

**On this page**

* TOC
{:toc}

---

## How to use this page

1. [Download r-session3.zip]({{ site.baseurl }}/modules/r/data/r-session3.zip) and unzip it. You get a folder called `r-session3` containing the notebook `R_course_session3_STUDENT.Rmd`, an `input` folder with `abundance.csv` and `taxonomy.csv`, and an empty `output` folder.
2. Open the notebook in RStudio and run each chunk with the green "play" button.
3. Complete the three exercises. Each one makes a small change to code that has already been explained.

| Section | Content |
|---|---|
| 0 and 1 | Inputs, metadata and the phyloseq object |
| 2 | Relative abundance and a t-test |
| 3 | Composition and taxonomic rank |
| 4 | Alpha diversity, t-test and plots |
| 5 | Beta diversity and PCoA |

---

## 0. Packages and inputs

```r
# Run only if the packages are not installed:
# install.packages(c("dplyr", "ggplot2", "vegan"))
# install.packages("BiocManager")
# BiocManager::install("phyloseq")

library(dplyr)
library(ggplot2)
library(phyloseq)
library(vegan)

abundance <- read.csv("input/abundance.csv", check.names = FALSE)
# Remove taxa with no observations
abundance <- abundance[-which(rowSums(as.matrix(abundance[-1])) == 0), ]
taxonomy  <- read.csv("input/taxonomy.csv", check.names = FALSE)

dim(abundance)
head(abundance)
head(taxonomy)
```

Loading the packages prints a few messages (for example the `dplyr` one explained in [Part 4]({{ site.baseurl }}/modules/r/part-4-handling-tables-with-dplyr/#setup)): they are not errors.

```
[1] 162   7
                       species Control_R1 Control_R2    A_R1    A_R2    B_R1    B_R2
1      Bifidobacterium bifidum     7.1296     5.0689 12.3905  6.5088 12.3610 19.6386
2    Lactobacillus acidophilus     0.0000     1.5076  3.2884  3.0498  1.4205  7.8599
3 Bifidobacterium adolescentis     9.4723     2.5209 13.0599 11.8849  9.1369 10.9719
4       Bifidobacterium longum     9.9385     4.5449 10.9524 13.7371  9.5213 10.0156
5  Catenibacterium tridentinum     2.5612     0.3143  2.0679  2.1465  2.1397  2.1901
6             Escherichia coli    32.3609    56.7068  8.6475 11.1042 11.3136 15.2895
```

The second line removes the species that are 0 in all six samples ([Part 3]({{ site.baseurl }}/modules/r/part-3-our-own-data/#inputabundancecsv) counted 28 of them): 190 species become 162. Read it from the inside out: `rowSums(...) == 0` finds the empty rows, `which()` gives their positions, and the minus sign removes them. `check.names = FALSE` keeps the column names exactly as they are in the file.

`head(taxonomy)` shows the first rows of the taxonomy table, as in Part 3.

**What is ggplot2?** It is the most widely used R package for making plots. It works in layers: you give it the data, map variables to axes or colours with `aes()`, and add elements with `+`. You will build your first plot in section 2.

---

## 1. Create the phyloseq object

A phyloseq object combines three tables in one object, with everything needed for the analysis:

| Part | Contains | In phyloseq |
|---|---|---|
| Abundance | Rows = species, columns = samples (162 x 6) | `otu_table()` |
| Taxonomy | From kingdom to species, for every taxon | `tax_table()` |
| Metadata | Product (Control, A, B) and replicate (R1, R2) for every sample | `sample_data()` |

The species name is what links the abundance table and the taxonomy table; the sample name links the abundance table and the metadata.

### Abundance and taxonomy

```r
# Abundance matrix: rows = species; columns = samples
abundance_matrix <- as.matrix(abundance[, -1])
rownames(abundance_matrix) <- abundance$species

abundance_matrix[1:5, ]
colSums(abundance_matrix)   # approximately 100% per sample
```

```
                             Control_R1 Control_R2    A_R1    A_R2    B_R1    B_R2
Bifidobacterium bifidum          7.1296     5.0689 12.3905  6.5088 12.3610 19.6386
Lactobacillus acidophilus        0.0000     1.5076  3.2884  3.0498  1.4205  7.8599
Bifidobacterium adolescentis     9.4723     2.5209 13.0599 11.8849  9.1369 10.9719
Bifidobacterium longum           9.9385     4.5449 10.9524 13.7371  9.5213 10.0156
Catenibacterium tridentinum      2.5612     0.3143  2.0679  2.1465  2.1397  2.1901
Control_R1 Control_R2       A_R1       A_R2       B_R1       B_R2 
   97.0681    87.6628   100.0000    98.4569    95.3876    99.9998 
```

1. **Drop the first column.** It holds the species names, which become the row names instead. A matrix can only hold one type of data: here, numbers.
2. **Check the totals.** Each sample adds up to about 100% (the unclassified reads explained in Part 3 are missing).

```r
# Taxonomy matrix
taxonomy_matrix <- as.matrix(
  taxonomy[, c("kingdom", "phylum", "class", "order",
               "family", "genus", "species")]
)
rownames(taxonomy_matrix) <- taxonomy$species
```

3. **Same row names.** The species names are the row names of both matrices: that is how phyloseq matches each abundance to its taxonomy.

### Metadata and the phyloseq object

```r
# Sample metadata
metadata <- data.frame(
  sample = colnames(abundance_matrix),
  product = c("Control", "Control", "A", "A", "B", "B"),
  replicate = c("R1", "R2", "R1", "R2", "R1", "R2")
)

metadata$product <- factor(metadata$product,
                           levels = c("Control", "A", "B"))
rownames(metadata) <- metadata$sample

# Combine the three parts
ps <- phyloseq(
  otu_table(abundance_matrix, taxa_are_rows = TRUE),
  tax_table(taxonomy_matrix),
  sample_data(metadata)
)
ps
```

```
phyloseq-class experiment-level object
otu_table()   OTU Table:         [ 162 taxa and 6 samples ]
sample_data() Sample Data:       [ 6 samples by 3 sample variables ]
tax_table()   Taxonomy Table:    [ 162 taxa by 7 taxonomic ranks ]
```

**Group order.** `factor(..., levels = ...)` turns `product` into a **factor**: a categorical variable with a fixed order. Without it, R sorts groups alphabetically (A, B, Control, as in Part 4). With it, Control always appears first, in tables and plots.

**Three pieces, one object.** `otu_table()`, `tax_table()` and `sample_data()` are combined with `phyloseq()`. `taxa_are_rows = TRUE` tells phyloseq that species are in the rows. The taxonomy table has 190 species, but phyloseq keeps only the 162 that are also in the abundance table.

---

## 2. Relative abundance and a t-test

**Relative abundance** is the percentage of the community occupied by a taxon. A value of 10 means approximately 10% of the sample, not ten bacterial cells. One sample = 100%.

We compare *Blautia wexlerae* between Control and product A.

```r
abundance_long <- psmelt(ps)

blautia <- filter(abundance_long,
                  OTU == "Blautia wexlerae",
                  product %in% c("Control", "A"))
blautia$product <- droplevels(blautia$product)
blautia
```

```
               OTU     Sample Abundance     sample product replicate  kingdom
1 Blautia wexlerae       A_R1    1.0982       A_R1       A        R1 Bacteria
2 Blautia wexlerae       A_R2    0.9956       A_R2       A        R2 Bacteria
3 Blautia wexlerae Control_R2    0.6451 Control_R2 Control        R2 Bacteria
4 Blautia wexlerae Control_R1    0.5369 Control_R1 Control        R1 Bacteria
      phylum      class          order          family   genus          species
1 Firmicutes Clostridia Lachnospirales Lachnospiraceae Blautia Blautia wexlerae
2 Firmicutes Clostridia Lachnospirales Lachnospiraceae Blautia Blautia wexlerae
3 Firmicutes Clostridia Lachnospirales Lachnospiraceae Blautia Blautia wexlerae
4 Firmicutes Clostridia Lachnospirales Lachnospiraceae Blautia Blautia wexlerae
```

| Step | What it does |
|---|---|
| `psmelt()` | Turns the phyloseq object into one long table: one row per taxon and sample, with the abundance, the metadata and the taxonomy side by side |
| `filter()` | Keeps one species and two groups. `%in%` means "is one of": `product %in% c("Control", "A")` is TRUE for Control and for A |
| `droplevels()` | Removes the empty group B from the factor, so that it does not appear in plots and tests |

Look at the means before testing them:

```r
blautia_groups <- group_by(blautia, product)
summarise(blautia_groups,
          mean_abundance = mean(Abundance))
```

```
# A tibble: 2 × 2
  product mean_abundance
  <fct>            <dbl>
1 Control          0.591
2 A                1.05 
```

`<fct>` means that `product` is a factor. Control comes first, as set in the metadata.

### Our first ggplot

```r
ggplot(blautia,
       aes(x = product, y = Abundance, color = product)) +
  geom_point(size = 3) +
  stat_summary(fun = mean, geom = "point",
               shape = 18, size = 5, color = "black") +
  labs(title = "Relative abundance of Blautia wexlerae",
       subtitle = "Black diamond = group mean",
       x = "Product",
       y = "Relative abundance (%)") +
  theme_minimal() +
  theme(legend.position = "none")
```

<figure>
<img src="{{ site.baseurl }}/modules/r/images/blautia-wexlerae.png" style="max-width:560px" alt="Dot plot of the relative abundance of Blautia wexlerae: the two Control samples are around 0.54 and 0.65%, the two A samples around 1.0 and 1.1%, with a black diamond for each group mean.">
</figure>

| Layer | What it adds |
|---|---|
| `ggplot()` + `aes()` | The data, and what goes on each axis and colour |
| `geom_point()` | One point per sample |
| `stat_summary()` | The group mean, as a black diamond (`shape = 18`) |
| `labs()` | Title and axis labels |
| `theme_minimal()`, `theme()` | The look of the plot; here, no legend |

### t-test: do the means differ?

A t-test compares the mean of one numerical variable between two groups.

- H0: the two population means are equal.
- H1: the two population means are different.

```r
t_blautia <- t.test(Abundance ~ product, data = blautia)
t_blautia
```

```

	Welch Two Sample t-test

data:  Abundance by product
t = -6.1149, df = 1.9944, p-value = 0.02589
alternative hypothesis: true difference in means between group Control and group A is not equal to 0
95 percent confidence interval:
 -0.7775539 -0.1342461
sample estimates:
mean in group Control       mean in group A 
               0.5910                1.0469 
```

`Abundance ~ product` reads "Abundance as a function of product". R uses Welch's t-test by default, which does not assume that the two groups have the same variance.

Find the p-value. Is it below 0.05?

<details markdown="1">
<summary>Answer</summary>

p = 0.026, below 0.05: in this example, we reject H0.

With only two samples per group, this is an exploratory teaching example. The [Experimental design module]({{ site.baseurl }}/modules/experimental-design/part-2-power-and-sample-size/) explains why so few replicates give so little power.

</details>

---

## 3. Community composition

One stacked bar represents one complete sample. Each colour is the relative abundance of one taxon, and the bar adds up to about 100%.

### Teacher example: family level

These two values control the plot:

```r
rank_to_plot <- "family"
number_to_show <- 10
```

**Grouping by rank and picking the top families:**

```r
# Group species at the selected rank
ps_rank <- tax_glom(ps, taxrank = rank_to_plot)
composition_long <- psmelt(ps_rank)

# Select the taxonomy column called "family"
composition_long$Taxon <- as.character(
  composition_long[[rank_to_plot]]
)

# Find the most abundant families
taxon_means <- group_by(composition_long, Taxon)
taxon_means <- summarise(taxon_means,
                         mean_abundance = mean(Abundance),
                         .groups = "drop")
taxon_means <- arrange(taxon_means, desc(mean_abundance))
top_taxa <- head(taxon_means$Taxon, number_to_show)
```

| Step | What it does |
|---|---|
| `tax_glom()` | Adds up the species that belong to the same family |
| `[[rank_to_plot]]` | Takes the column whose name is stored in `rank_to_plot`, here `family`. `$` needs the name typed out; `[[ ]]` accepts a variable |
| `group_by()` + `summarise()` + `arrange()` | The mean abundance of each family over the six samples, from highest to lowest |
| `head(..., number_to_show)` | The names of the 10 most abundant families |

**Combining the rest as "Other":**

```r
# Combine the remaining families as "Other"
composition_long$Taxon <- ifelse(
  composition_long$Taxon %in% top_taxa,
  composition_long$Taxon,
  "Other"
)

composition_plot_data <- group_by(
  composition_long, Sample, product, Taxon
)
composition_plot_data <- summarise(
  composition_plot_data,
  Abundance = sum(Abundance),
  .groups = "drop"
)

composition_plot_data$Sample <- factor(
  composition_plot_data$Sample,
  levels = metadata$sample
)
```

| Step | What it does |
|---|---|
| `ifelse()` | In the top 10? Keep the name. If not, call it "Other" |
| `sum(Abundance)` | A single "Other" block per sample |
| `factor(levels = ...)` | Keeps the samples in the metadata order |

`.groups = "drop"` removes the grouping from the result once the summary is done.

**The stacked bar chart:**

```r
ggplot(composition_plot_data,
       aes(x = Sample, y = Abundance, fill = Taxon)) +
  geom_col() +
  facet_grid(~ product, scales = "free_x", space = "free_x") +
  labs(title = paste("Microbial composition at",
                     rank_to_plot, "level"),
       subtitle = paste("Top", number_to_show,
                        "taxa; all others are Other"),
       x = "Sample",
       y = "Relative abundance (%)",
       fill = rank_to_plot) +
  theme_minimal() +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))
```

<figure>
<img src="{{ site.baseurl }}/modules/r/images/composition-family.png" alt="Stacked bar chart of the six samples, grouped by product, showing the ten most abundant families and Other. Enterobacteriaceae dominates the two Control samples; Bifidobacteriaceae dominates the A and B samples.">
</figure>

`geom_col()` draws bars whose height is the value; `fill = Taxon` stacks one colour per family. `facet_grid(~ product)` makes one panel per product. `paste()` glues pieces of text together, so the title follows `rank_to_plot`.

Why does every bar total approximately 100%?

<details markdown="1">
<summary>Answer</summary>

Each bar is one complete sample: every taxon of that sample is in it, either under its own name or inside "Other". The bars are not exactly 100% because of the reads that could not be classified (Part 3): Control_R2, at about 88%, is the clearest case.

</details>

### Exercise 1: species level

Change the graph from the top 10 families to the top **15 species**. Fill in two values, and rerun all the composition chunks.

```r
# > your answer:
rank_to_plot <- "________"
number_to_show <- __
```

<details markdown="1">
<summary>Answer</summary>

```r
rank_to_plot <- "species"
number_to_show <- 15
# Then rerun the tax_glom / top, "Other" and ggplot chunks
```

<figure>
<img src="{{ site.baseurl }}/modules/r/images/composition-species.png" alt="Stacked bar chart of the six samples showing the fifteen most abundant species and Other. Escherichia coli takes most of the two Control samples.">
</figure>

Two changes, and the same code works for any taxonomic rank. The title, subtitle and legend update automatically, because they use `rank_to_plot` and `number_to_show`.

</details>

---

## 4. Alpha diversity

Alpha diversity describes the microbial community **within one sample**.

| Index | What it measures |
|---|---|
| Observed | Number of detected species (abundance above zero) |
| Shannon | Considers both richness and how evenly abundances are distributed |

```r
alpha <- data.frame(
  sample = colnames(abundance_matrix),
  Observed = colSums(abundance_matrix > 0),
  Shannon = vegan::diversity(t(abundance_matrix),
                             index = "shannon")
)

alpha <- dplyr::left_join(alpha, metadata, by = "sample")
alpha
```

```
      sample Observed  Shannon product replicate
1 Control_R1       44 2.489175 Control        R1
2 Control_R2       71 1.780579 Control        R2
3       A_R1      101 3.327179       A        R1
4       A_R2      112 3.393235       A        R2
5       B_R1      161 3.496686       B        R1
6       B_R2       39 2.711256       B        R2
```

- `abundance_matrix > 0` gives TRUE for every species detected in a sample; `colSums()` counts them per sample, as `sum(numbers > 3)` did in Part 2.
- `t()` transposes the matrix (rows become columns), because `diversity()` expects samples in rows.
- `vegan::` and `dplyr::` say explicitly which package the function comes from.
- `left_join()` adds the metadata columns to each sample, matching them by the `sample` column.

`Observed` is detected richness, not Chao1. It counts the species with an abundance above zero.

### Exercise 2: t-test on Shannon

Repeat the Control-versus-A t-test using Shannon diversity. Replace both blanks with the name of the correct column.

```r
alpha_CA <- filter(alpha,
                   product %in% c("Control", "A"))
alpha_CA$product <- droplevels(alpha_CA$product)

# Calculate the group means
alpha_groups <- group_by(alpha_CA, product)
summarise(alpha_groups,
          mean_Shannon = mean(________))

# Run the t-test
t_shannon <- t.test(________ ~ product, data = alpha_CA)
t_shannon
```

Is the p-value below 0.05?

<details markdown="1">
<summary>Answer</summary>

Both blanks are `Shannon`:

```r
alpha_CA <- filter(alpha,
                   product %in% c("Control", "A"))
alpha_CA$product <- droplevels(alpha_CA$product)

alpha_groups <- group_by(alpha_CA, product)
summarise(alpha_groups,
          mean_Shannon = mean(Shannon))

t_shannon <- t.test(Shannon ~ product, data = alpha_CA)
t_shannon
```

```
# A tibble: 2 × 2
  product mean_Shannon
  <fct>          <dbl>
1 Control         2.13
2 A               3.36

	Welch Two Sample t-test

data:  Shannon by product
t = -3.4435, df = 1.0174, p-value = 0.1765
alternative hypothesis: true difference in means between group Control and group A is not equal to 0
95 percent confidence interval:
 -5.56814  3.11748
sample estimates:
mean in group Control       mean in group A 
             2.134877              3.360207 
```

p = 0.177, above 0.05: we do not reject H0. This small dataset provides no evidence of a difference in mean Shannon diversity.

The means look quite different (2.13 and 3.36), but the two Control samples are far apart (1.78 and 2.49): with two samples per group, that spread is enough to make the test inconclusive.

</details>

### Teacher example: observed richness

```r
ggplot(alpha,
       aes(x = product, y = Observed, color = product)) +
  stat_summary(fun.min = min, fun.max = max,
               geom = "linerange", color = "grey60") +
  geom_point(size = 3) +
  stat_summary(fun = mean, geom = "point",
               shape = 18, size = 5, color = "black") +
  labs(title = "Observed species richness",
       subtitle = "Black diamond = mean; grey line = range",
       x = "Product",
       y = "Observed species") +
  theme_minimal() +
  theme(legend.position = "none")
```

<figure>
<img src="{{ site.baseurl }}/modules/r/images/observed-richness.png" style="max-width:560px" alt="Observed species per product: Control samples at 44 and 71, A at 101 and 112, B at 161 and 39, each with a grey line for the range and a black diamond for the mean.">
</figure>

The new layer is the first `stat_summary()`: `linerange` draws a grey line from each group's minimum to its maximum. Look at B: its two replicates are 39 and 161 species, the widest range of the three products.

### Exercise 3: Shannon plot

Copy the previous plot by completing these three blanks: the plotted variable, the title and the y-axis label.

```r
# > your answer:
ggplot(alpha,
       aes(x = product, y = ________, color = product)) +
  stat_summary(fun.min = min, fun.max = max,
               geom = "linerange", color = "grey60") +
  geom_point(size = 3) +
  stat_summary(fun = mean, geom = "point",
               shape = 18, size = 5, color = "black") +
  labs(title = "________________",
       subtitle = "Black diamond = mean; grey line = range",
       x = "Product",
       y = "________________") +
  theme_minimal() +
  theme(legend.position = "none")
```

<details markdown="1">
<summary>Answer</summary>

```r
ggplot(alpha,
       aes(x = product, y = Shannon, color = product)) +
  stat_summary(fun.min = min, fun.max = max,
               geom = "linerange", color = "grey60") +
  geom_point(size = 3) +
  stat_summary(fun = mean, geom = "point",
               shape = 18, size = 5, color = "black") +
  labs(title = "Shannon diversity",
       subtitle = "Black diamond = mean; grey line = range",
       x = "Product",
       y = "Shannon index") +
  theme_minimal() +
  theme(legend.position = "none")
```

<figure>
<img src="{{ site.baseurl }}/modules/r/images/shannon-diversity.png" style="max-width:560px" alt="Shannon index per product: Control around 1.8 and 2.5, A around 3.3 and 3.4, B around 2.7 and 3.5.">
</figure>

Three changes: the y variable, the title and the y-axis label. All the layers stay the same.

</details>

---

## 5. Beta diversity: a brief PCoA

Alpha diversity describes one sample. Beta diversity asks **how different the complete communities are between samples**.

1. **Samples:** the complete composition of each one.
2. **Bray-Curtis:** a distance between every pair of samples.
3. **PCoA:** the distances shown on two axes. Nearby points have more similar communities; separated points have more different communities.

```r
pcoa <- ordinate(ps, method = "PCoA", distance = "bray")

plot_ordination(ps, pcoa, type = "samples",
                color = "product") +
  geom_point(size = 4) +
  labs(title = "PCoA of Bray-Curtis distances",
       subtitle = "Closer samples have more similar compositions",
       color = "Product") +
  theme_minimal()
```

<figure>
<img src="{{ site.baseurl }}/modules/r/images/pcoa-bray.png" style="max-width:620px" alt="PCoA of Bray-Curtis distances. The first axis explains 71.1% of the variation and the second 14.4%. A_R1, A_R2 and B_R1 sit close together on the right; B_R2 is above them; Control_R1 is at the top centre and Control_R2 alone on the far left.">
</figure>

The percentages on the axes are the share of the variation in the distances that each axis shows: here 71.1% on the first axis and 14.4% on the second.

R may also print a warning saying that `aes_string()` is deprecated. It comes from inside the phyloseq package, not from your code, and does not affect the plot.

Which samples are close? Which sample appears most different?

<details markdown="1">
<summary>Answer</summary>

A_R1, A_R2 and B_R1 sit close together: their compositions are similar. B_R2 is not far from them. The two Control samples are apart from the rest and from each other, and Control_R2 is the most different sample, alone at the far left of the first axis. This matches the composition plot, where Escherichia coli takes up most of Control_R2.

PCoA is a visualisation, not a statistical test.

</details>

---

## Take-home message

| Concept | In one line |
|---|---|
| Relative abundance | The percentage occupied by one taxon |
| t-test | Compares one numerical variable between two groups |
| Composition | Can be shown at different taxonomic ranks |
| Alpha diversity | Describes one sample |
| Beta diversity | Compares complete communities between samples |
| Caution | With two replicates per group, the results are exploratory |

**Next:** [Part 7: TreeSummarizedExperiment and miaverse]({{ site.baseurl }}/modules/r/part-7-treesummarizedexperiment-and-miaverse/).
