---
title: "Part 7: TreeSummarizedExperiment and miaverse"
---

# Part 7: TreeSummarizedExperiment and miaverse
{: .no_toc}

<div class="panel" markdown="1">
**Estimated time:** 45 to 60 minutes.<br>
**You need:** R and RStudio with the packages `phyloseq`, `mia`, `miaViz`, `ComplexHeatmap`, `scater` and `patchwork` (see [Setup]({{ site.baseurl }}/modules/about/setup/#install-the-r-packages)), and the session 4 notebook and data: [download r-session4.zip]({{ site.baseurl }}/modules/r/data/r-session4.zip).<br>
**Before you start:** this page builds on the phyloseq object of [Part 6]({{ site.baseurl }}/modules/r/part-6-microbiome-data-in-r/). If you start here, the notebook rebuilds it for you.<br>
**Authors:** Andrés Méndez Hurtado, Adrián Martín Segura, Alba Pérez Cuervo, Laura Judith Marcos Zambrano, Matti Ruuskanen.
</div>

Advanced data objects: TreeSummarizedExperiment and the miaverse.

**On this page**

* TOC
{:toc}

---

## How to use this page

1. [Download r-session4.zip]({{ site.baseurl }}/modules/r/data/r-session4.zip) and unzip it. You get a folder called `r-session4` containing the notebook `R_course_session4_STUDENT.Rmd`, an `input` folder with `abundance.csv` and `taxonomy.csv`, and an empty `output` folder.
2. Open the notebook in RStudio and run each chunk with the green "play" button.
3. Complete the two exercises. The first one makes a small change to code already explained. For the second one, you first have to find and copy the relevant code from the examples.

---

## 0. Packages and inputs

```r
# Run only if the packages are not installed:
# install.packages(c("BiocManager", "patchwork"))
# BiocManager::install(c("phyloseq", "mia", "miaViz", "ComplexHeatmap", "scater"))

library(phyloseq)
library(mia)
library(TreeSummarizedExperiment)
library(miaViz)
library(ComplexHeatmap)
library(scater)
library(patchwork)
```

These packages load many others (Bioconductor packages are built on top of each other), so R prints long lists of messages about packages being attached and functions being masked. That is normal. Only a line starting with `Error` means something went wrong.

`TreeSummarizedExperiment` is installed together with `mia`.

---

## 1. Reading in raw data

Microbiome data can come in many different formats. As with phyloseq, we can read in data from various sources as .csv files. However, we first have to make sure they are correctly formatted before they can be included in the TSE (TreeSummarizedExperiment) object.

Let's read in the abundance table and the taxonomy table and check their formatting. We will also construct the sample data (here called "metadata") as in [Part 6]({{ site.baseurl }}/modules/r/part-6-microbiome-data-in-r/).

### The abundance table

The abundance table should be a numeric matrix with feature IDs (here, the species names) as row names and sample IDs as column names.

```r
abundance <- read.csv("input/abundance.csv", check.names = FALSE)
str(abundance)
head(rownames(abundance))
head(colnames(abundance))
```

```
'data.frame':	190 obs. of  7 variables:
 $ species   : chr  "Bifidobacterium bifidum" "Lactobacillus acidophilus" "Bifidobacterium adolescentis" "Bifidobacterium longum" ...
 $ Control_R1: num  7.13 0 9.47 9.94 2.56 ...
 $ Control_R2: num  5.069 1.508 2.521 4.545 0.314 ...
 $ A_R1      : num  12.39 3.29 13.06 10.95 2.07 ...
 $ A_R2      : num  6.51 3.05 11.88 13.74 2.15 ...
 $ B_R1      : num  12.36 1.42 9.14 9.52 2.14 ...
 $ B_R2      : num  19.64 7.86 10.97 10.02 2.19 ...
[1] "1" "2" "3" "4" "5" "6"
[1] "species"    "Control_R1" "Control_R2" "A_R1"       "A_R2"       "B_R1"      
```

The printouts show that the row names are just numbers, and that the species names are in a column. We need to move the feature IDs to the row names, and convert the object to the matrix format:

```r
rownames(abundance) <- abundance$species
abundance <- as.matrix(abundance[-1])
```

The second line also removes the first column (`species`): we no longer need it, and it is text.

### The taxonomy table

The taxonomy table should be a data frame, whose row names match the row names of the abundance table.

```r
taxonomy  <- read.csv("input/taxonomy.csv", check.names = FALSE)
str(taxonomy)
head(rownames(taxonomy))
```

```
'data.frame':	190 obs. of  7 variables:
 $ species: chr  "Bifidobacterium bifidum" "Lactobacillus acidophilus" "Bifidobacterium adolescentis" "Bifidobacterium longum" ...
 $ kingdom: chr  "Bacteria" "Bacteria" "Bacteria" "Bacteria" ...
 $ phylum : chr  "Actinomycetota" "Firmicutes" "Actinomycetota" "Actinomycetota" ...
 $ class  : chr  "Actinomycetes" "Bacilli" "Actinomycetes" "Actinomycetes" ...
 $ order  : chr  "Bifidobacteriales" "Lactobacillales" "Bifidobacteriales" "Bifidobacteriales" ...
 $ family : chr  "Bifidobacteriaceae" "Lactobacillaceae" "Bifidobacteriaceae" "Bifidobacteriaceae" ...
 $ genus  : chr  "Bifidobacterium" "Lactobacillus" "Bifidobacterium" "Bifidobacterium" ...
[1] "1" "2" "3" "4" "5" "6"
```

Put the columns in taxonomic order, from kingdom to species, and again move the species names to the row names:

```r
taxonomy <- taxonomy[, c("kingdom", "phylum", "class", "order",
               "family", "genus", "species")]
rownames(taxonomy) <- taxonomy$species

# Make sure that the row names match before merging
identical(rownames(abundance), rownames(taxonomy))
```

```
[1] TRUE
```

`identical()` answers TRUE only if the two are exactly the same: same names, in the same order.

### The sample metadata

Here, the row names should match the column names of the abundance table.

```r
metadata <- data.frame(
  sample = colnames(abundance),
  product = c("Control", "Control", "A", "A", "B", "B"),
  replicate = c("R1", "R2", "R1", "R2", "R1", "R2")
)

metadata$product <- factor(metadata$product,
                           levels = c("Control", "A", "B"))
rownames(metadata) <- metadata$sample
```

### Building the TreeSE object

```r
# Construct a (minimal) TreeSE object
tse_taxa <- TreeSummarizedExperiment(
    assays = SimpleList(counts = abundance),
    colData = metadata,
    rowData = taxonomy
)

# Some bacteria are not present in any sample, so remove them
tse_taxa <- subsetByPrevalent(tse_taxa, prevalence = 0)

# What does it look like?
tse_taxa
```

```
class: TreeSummarizedExperiment 
dim: 162 6 
metadata(0):
assays(1): counts
rownames(162): Bifidobacterium bifidum Lactobacillus acidophilus ...
  Candidatus Stoquefichus sp SB1 Ruminococcus callidus
rowData names(7): kingdom phylum ... genus species
colnames(6): Control_R1 Control_R2 ... B_R1 B_R2
colData names(3): sample product replicate
reducedDimNames(0):
mainExpName: NULL
altExpNames(0):
rowLinks: NULL
rowTree: NULL
colLinks: NULL
colTree: NULL
```

The object holds 162 taxa (the 28 absent from all six samples were removed) and 6 samples. The printout lists its **slots**: the abundance data in `assays`, the taxonomy in `rowData`, the sample information in `colData`. The empty slots (`reducedDimNames`, `rowTree` and others) are places for more data: ordinations, phylogenetic trees, other data layers.

---

## 2. Converting a phyloseq object

The notebook recreates the phyloseq object of Part 6, if it is not in the environment already. A phyloseq object can then be converted into a TreeSE object with a single command. All the data in a phyloseq object fits *inside* the TSE object, with room to spare: this is useful later.

```r
# Check whether the phyloseq object exists. "!" inverts the result of exists()
if (!exists("ps")) {
  # Abundance matrix: remove taxa with 0 in all samples
  abundance_matrix <- abundance[-which(rowSums(abundance) == 0), ]

  # Taxonomy matrix
  taxonomy_matrix <- as.matrix(
    taxonomy[, c("kingdom", "phylum", "class", "order",
                 "family", "genus", "species")]
  )
  rownames(taxonomy_matrix) <- taxonomy$species

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
}

ps
```

```
phyloseq-class experiment-level object
otu_table()   OTU Table:         [ 162 taxa and 6 samples ]
sample_data() Sample Data:       [ 6 samples by 3 sample variables ]
tax_table()   Taxonomy Table:    [ 162 taxa by 7 taxonomic ranks ]
```

`if (...) { ... }` runs the code between the curly brackets only when the condition is TRUE: here, only when `ps` does not exist yet.

Now convert it, and check whether the result is identical to the TSE built from the raw data:

```r
tse_taxa_ps <- convertFromPhyloseq(ps)

# Are the objects identical?
identical(tse_taxa, tse_taxa_ps)
```

```
[1] TRUE
```

There is still a terminological issue in the TSE object: the data are relative abundances, but the assay is called "counts". Fix the name:

```r
names(assays(tse_taxa)) <- "relabundance"
```

---

## 3. How to access data

The terminology and the access functions are slightly different between phyloseq and TSE. Briefly, the data from your experiment are sampled objects x observed things. Here, the observed things are microbial taxa identified at the species level. Both the samples and the taxa can have further data associated with them, such as chemical data for the samples, or phylogenetic and taxonomic data for the taxa.

TSE keeps **samples in columns** and **taxa in rows**. The observations of samples x taxa, here relative abundances, are kept in the **assay** slot.

| | phyloseq | TreeSummarizedExperiment |
|---|---|---|
| Abundances | `otu_table()` | `assay()` |
| Sample information (columns) | `sample_data()` | `colData()` |
| Taxonomy (rows) | `tax_table()` | `rowData()` |

```r
# The assay slot contains what was the "otu_table" in phyloseq
head(assay(tse_taxa, "relabundance"))
summary(tse_taxa, assay.type = "relabundance")
```

```
                             Control_R1 Control_R2    A_R1    A_R2    B_R1    B_R2
Bifidobacterium bifidum          7.1296     5.0689 12.3905  6.5088 12.3610 19.6386
Lactobacillus acidophilus        0.0000     1.5076  3.2884  3.0498  1.4205  7.8599
Bifidobacterium adolescentis     9.4723     2.5209 13.0599 11.8849  9.1369 10.9719
Bifidobacterium longum           9.9385     4.5449 10.9524 13.7371  9.5213 10.0156
Catenibacterium tridentinum      2.5612     0.3143  2.0679  2.1465  2.1397  2.1901
Escherichia coli                32.3609    56.7068  8.6475 11.1042 11.3136 15.2895
$samples
# A tibble: 1 × 6
  total_counts min_counts max_counts median_counts mean_counts stdev_counts
         <dbl>      <dbl>      <dbl>         <dbl>       <dbl>        <dbl>
1         579.       87.7        100          97.8        96.4         4.65

$features
# A tibble: 1 × 3
  total singletons per_sample_avg
  <int>      <int>          <dbl>
1   162          0             88
```

`summary()` summarises the samples (the column totals: here between 87.7 and 100%, as seen in Part 3) and the features (162 taxa, on average 88 per sample). The names say "counts", but with this assay they are percentages.

```r
# Samples are columns, so what was "sample_data" in phyloseq is now in colData
head(colData(tse_taxa))
```

```
DataFrame with 6 rows and 3 columns
                sample  product   replicate
           <character> <factor> <character>
Control_R1  Control_R1  Control          R1
Control_R2  Control_R2  Control          R2
A_R1              A_R1  A                R1
A_R2              A_R2  A                R2
B_R1              B_R1  B                R1
B_R2              B_R2  B                R2
```

```r
# Taxa are rows, so what was "tax_table" in phyloseq is now in rowData
head(rowData(tse_taxa))
```

```
DataFrame with 6 rows and 7 columns
                                 kingdom         phylum               class
                             <character>    <character>         <character>
Bifidobacterium bifidum         Bacteria Actinomycetota       Actinomycetes
Lactobacillus acidophilus       Bacteria     Firmicutes             Bacilli
Bifidobacterium adolescentis    Bacteria Actinomycetota       Actinomycetes
Bifidobacterium longum          Bacteria Actinomycetota       Actinomycetes
Catenibacterium tridentinum     Bacteria     Firmicutes    Erysipelotrichia
Escherichia coli                Bacteria Pseudomonadota Gammaproteobacteria
                                          order             family           genus
                                    <character>        <character>     <character>
Bifidobacterium bifidum       Bifidobacteriales Bifidobacteriaceae Bifidobacterium
Lactobacillus acidophilus       Lactobacillales   Lactobacillaceae   Lactobacillus
Bifidobacterium adolescentis  Bifidobacteriales Bifidobacteriaceae Bifidobacterium
Bifidobacterium longum        Bifidobacteriales Bifidobacteriaceae Bifidobacterium
Catenibacterium tridentinum  Erysipelotrichales   Coprobacillaceae Catenibacterium
Escherichia coli               Enterobacterales Enterobacteriaceae     Escherichia
                                            species
                                        <character>
Bifidobacterium bifidum      Bifidobacterium bifi..
Lactobacillus acidophilus    Lactobacillus acidop..
Bifidobacterium adolescentis Bifidobacterium adol..
Bifidobacterium longum       Bifidobacterium longum
Catenibacterium tridentinum  Catenibacterium trid..
Escherichia coli                   Escherichia coli
```

A `DataFrame` is the Bioconductor version of a data frame: it shows each column's type under its name, and shortens long text with `..`.

The whole object can also be subset by indexing, as with data frames: `[rows, columns]`. This gives the first 5 rows (taxa) and the fifth and sixth columns (samples):

```r
tse_taxa[1:5, c(5, 6)]
```

```
class: TreeSummarizedExperiment 
dim: 5 2 
metadata(0):
assays(1): relabundance
rownames(5): Bifidobacterium bifidum Lactobacillus acidophilus
  Bifidobacterium adolescentis Bifidobacterium longum Catenibacterium
  tridentinum
rowData names(7): kingdom phylum ... genus species
colnames(2): B_R1 B_R2
colData names(3): sample product replicate
reducedDimNames(0):
mainExpName: NULL
altExpNames(0):
rowLinks: NULL
rowTree: NULL
colLinks: NULL
colTree: NULL
```

The result is still a TSE object, with all its slots subset together: 5 taxa with their taxonomy, 2 samples with their metadata.

### Exercise 1: selecting only specific taxa

Select only the genus *Blautia* from the TSE object, and make it into a new object for further study. Fill in the blanks.

```r
tse_blautia <- tse_taxa[which(rowData(tse_taxa)$_______ == "________"),]
tse_blautia
```

How many different species of the *Blautia* genus do we find in our samples?

<details markdown="1">
<summary>Answer</summary>

```r
tse_blautia <- tse_taxa[which(rowData(tse_taxa)$genus == "Blautia"),]
tse_blautia
```

```
class: TreeSummarizedExperiment 
dim: 9 6 
metadata(0):
assays(1): relabundance
rownames(9): Blautia wexlerae Blautia massiliensis ex Durand et al 2017 ...
  Blautia intestinalis Blautia caecimuris
rowData names(7): kingdom phylum ... genus species
colnames(6): Control_R1 Control_R2 ... B_R1 B_R2
colData names(3): sample product replicate
reducedDimNames(0):
mainExpName: NULL
altExpNames(0):
rowLinks: NULL
rowTree: NULL
colLinks: NULL
colTree: NULL
```

`dim: 9 6`: 9 *Blautia* species, in the 6 samples. To see their names, use `rownames(tse_blautia)`.

The condition `rowData(tse_taxa)$genus == "Blautia"` is TRUE for the *Blautia* rows; `which()` turns it into row positions, and `[rows, ]` keeps those rows and all the columns.

</details>

---

## 4. What is the use of TSEs?

Phyloseq has been the "industry standard" for over a decade in microbiome research. However, the format is limited, especially with the need to juggle multiple objects. There is also no possibility of adding additional multiomic data layers, such as metabolomics, transcriptomics and proteomics. TreeSE has been extended to also handle both multiomic and time series data. These are slightly more advanced and complex functionalities, and beyond the scope of this course. Furthermore, there are plenty of ready-made, easy-to-use functions for plotting and analysis that work directly on the containers.

In the following examples, we will look at a few simple transformations and subsets of the data, and look for qualitative differences between the sample groups using beta diversity.

Microbiome data are inherently compositional. Some correlations and patterns might emerge or remain hidden in your data only because of how the increased relative abundance of one taxon affects the relative abundance of the others. Recently, the topic of data transformation has been hotly debated. We will try one compositional transform (robust Centered Log-Ratio) and one simple transform (presence/absence) that can be used in further analyses. While the presence-absence transformation simplifies the data greatly, it is also very robust. No compositional transform has been shown to improve analytical performance in comparison with total sum scaling (relative abundance).

### Summarising the data

Which taxa dominate each sample group?

```r
summarizeDominance(tse_taxa, assay.type = "relabundance",
                   rank = "genus", group = "product")
```

```
# A tibble: 3 × 4
# Groups:   product [3]
  product dominant_taxa       n rel_freq
  <fct>   <chr>           <int>    <dbl>
1 Control Escherichia         2        1
2 A       Bifidobacterium     2        1
3 B       Bifidobacterium     2        1
```

At genus level, *Escherichia* is the most abundant genus in both Control samples (`n = 2`, a relative frequency of 1, that is all of them), and *Bifidobacterium* in all the A and B samples.

### Prevalence of common taxa

```r
tse_taxa <- addPrevalentAbundance(tse_taxa, assay.type = "relabundance",
                                  prevalence = 0.5, detection = 0.01)
plotHistogram(tse_taxa, col.var = "prevalent_abundance")
```

A taxon is counted as **prevalent** when it is detected above the `detection` threshold in more than the `prevalence` fraction of the samples: here, in more than half of the samples. The detection threshold is in the units of the assay: these relative abundances are percentages, so `detection = 0.01` means 0.01%.

`addPrevalentAbundance()` adds a new column, `prevalent_abundance`, to `colData`: for each sample, the share of its abundance that belongs to prevalent taxa.

<figure>
<img src="{{ site.baseurl }}/modules/r/images/prevalent-abundance-histogram.png" style="max-width:560px" alt="Histogram of prevalent_abundance for the six samples, with values between 0.95 and 1.">
</figure>

All six samples are between 0.95 and 1: almost all of their abundance comes from taxa that are found in most samples. R also prints a message that `stat_bin()` uses 30 bins by default; it is not an error.

### Transformations

```r
# Transform relative abundance to robust Centered Log-Ratio
# (for compositional analysis) and to presence/absence
tse_taxa <- transformAssay(tse_taxa, assay.type = "relabundance", method = "rclr")
tse_taxa <- transformAssay(tse_taxa, assay.type = "relabundance", method = "pa")
```

Each transformation is stored as a new assay in the same object, next to the original: `tse_taxa` now has three assays, `relabundance`, `rclr` and `pa`. This is the "room to spare" mentioned in section 2.

### Jaccard dissimilarity

First, calculate the Jaccard dissimilarity from the presence/absence data. It shows how similar or dissimilar the samples are, based only on which taxa are present.

```r
tse_taxa <- addDissimilarity(tse_taxa, assay.type = "pa", method = "jaccard")

Heatmap(
    metadata(tse_taxa)[["jaccard"]],
    heatmap_legend_param = list(title = "Jaccard dissimilarity")
)
```

<figure>
<img src="{{ site.baseurl }}/modules/r/images/jaccard-heatmap.png" style="max-width:620px" alt="Heatmap of Jaccard dissimilarities between the six samples, with dendrograms. A_R1 and A_R2 are the most similar pair; B_R1 clusters with them. B_R2 clusters with Control_R1.">
</figure>

The dissimilarity matrix is stored in the `metadata` slot of the object. `Heatmap()` colours each pair of samples (blue = similar, red = dissimilar) and reorders the samples so that similar ones are next to each other: the trees on the sides show that grouping.

### Common and rare taxa

Divide the data into common and rare taxa, based on prevalence and detection thresholds:

```r
tse_common <- subsetByPrevalent(tse_taxa, assay.type = "relabundance", rank = "species",
                                prevalence = 0.5, detection = 0.1)
tse_rare <- subsetByRare(tse_taxa, assay.type = "relabundance", rank = "species",
                         prevalence = 0.5, detection = 0.1)
```

With these thresholds (above 0.1% in more than half of the samples), 47 of the 162 species are common and 115 are rare. You can check this with `nrow(tse_common)` and `nrow(tse_rare)`.

### Ordination of common and rare taxa

Calculate a PCoA for both subsets, as in Part 6, and also a compositional beta-diversity metric, the robust Aitchison distance (the Euclidean distance on the `rclr` data):

```r
tse_common <- addMDS(
    tse_common,
    FUN = getDissimilarity,
    method = "bray",
    assay.type = "relabundance",
    name = "MDS_bray"
)
tse_common <- addMDS(
    tse_common,
    FUN = getDissimilarity,
    method = "euclidean",
    assay.type = "rclr",
    name = "MDS_rob_aitchison"
)

tse_rare <- addMDS(
    tse_rare,
    FUN = getDissimilarity,
    method = "bray",
    assay.type = "relabundance",
    name = "MDS_bray"
)
tse_rare <- addMDS(
    tse_rare,
    FUN = getDissimilarity,
    method = "euclidean",
    assay.type = "rclr",
    name = "MDS_rob_aitchison"
)
```

Each ordination is stored in the object's `reducedDim` slot, under the name given with `name`.

Plot the four ordinations, each with a title, and arrange them with the `patchwork` package for easier comparison:

```r
p1 <- plotReducedDim(tse_common, "MDS_bray", colour_by = "product", theme_size=11) +
  ggtitle("Core species PCoA")
p2 <- plotReducedDim(tse_rare, "MDS_bray", colour_by = "product", theme_size=11) +
  ggtitle("Rare species PCoA")
p3 <- plotReducedDim(tse_common, "MDS_rob_aitchison", colour_by = "product", theme_size=11) +
  ggtitle("Core species Robust Aitchison")
p4 <- plotReducedDim(tse_rare, "MDS_rob_aitchison", colour_by = "product", theme_size=11) +
  ggtitle("Rare species Robust Aitchison")

(p1+p2) / (p3+p4)
```

<figure>
<img src="{{ site.baseurl }}/modules/r/images/beta-diversity-four-panels.png" alt="Four ordination plots of the six samples coloured by product: Bray-Curtis PCoA of core species and of rare species (top), robust Aitchison of core species and of rare species (bottom).">
</figure>

With `patchwork`, `+` puts plots side by side and `/` puts them on top of each other.

What can we see from the results? Are there differences between the PCoA (Bray-Curtis) and the robust Aitchison results? Which beta-diversity plots support the observations from the Jaccard heatmap?

<details markdown="1">
<summary>Discussion</summary>

- **Jaccard heatmap** (which taxa are present): A_R1 and A_R2 are the most similar pair, and B_R1 is close to them. B_R2 is closest to Control_R1, and far from its own replicate B_R1.
- **Bray-Curtis PCoA** (top row): the A samples and B_R1 sit together, and the two Controls are apart. B_R2 is near the A and B samples for core species, but far from every other sample for rare species.
- **Robust Aitchison** (bottom row): for both core and rare species, B_R2 sits with the two Controls, as in the Jaccard heatmap.

So the robust Aitchison plots support the Jaccard heatmap better than the Bray-Curtis ones. Bray-Curtis is driven by the most abundant taxa, where B_R2 resembles the other product samples (*Bifidobacterium* dominates it). Jaccard only looks at which taxa are present, and B_R2, like the Controls, has few detected species (39, against 101 to 161 in the other product samples: see the alpha diversity in Part 6).

The choice of distance changes the picture. With two replicates per group, these are qualitative observations, not statistical results.

</details>

### Exercise 2: simple analysis of one genus

Say we want to look at the differences in the relative abundances of *Blautia* and *Bifidobacterium* species. First create a subset object for *Bifidobacterium* (the *Blautia* object already exists from Exercise 1), then plot two heatmaps with `Heatmap()`. How do the clustering patterns of the samples differ between these taxa?

You can reuse code from the examples above.

<details markdown="1">
<summary>Answer</summary>

```r
tse_bifido <- tse_taxa[which(rowData(tse_taxa)$genus == "Bifidobacterium"),]

Heatmap(
    assay(tse_blautia, "relabundance"),
    heatmap_legend_param = list(title = "Relative abundance (%)"),
    column_title = "Blautia"
)

Heatmap(
    assay(tse_bifido, "relabundance"),
    heatmap_legend_param = list(title = "Relative abundance (%)"),
    column_title = "Bifidobacterium"
)
```

The first line is Exercise 1 with another genus; the heatmaps reuse the Jaccard example, with the `relabundance` assay of each subset instead of the dissimilarity matrix. `column_title` adds a title.

<figure>
<img src="{{ site.baseurl }}/modules/r/images/heatmap-blautia.png" alt="Heatmap of the relative abundance of nine Blautia species in the six samples. The A and B samples cluster together, and the two Control samples cluster together.">
</figure>

<figure>
<img src="{{ site.baseurl }}/modules/r/images/heatmap-bifidobacterium.png" alt="Heatmap of the relative abundance of seven Bifidobacterium species in the six samples. B_R2 stands apart from the other samples.">
</figure>

- ***Blautia***: the samples cluster by group. The two Controls are together, with less *Blautia* overall, and the A and B samples are together.
- ***Bifidobacterium***: the samples do not cluster by product. B_R2 stands apart, mostly because of its high *B. bifidum* (19.6%, against 5.1 to 12.4% in the other samples).

Remember that each heatmap is scaled to its own genus: the colours of the two cannot be compared directly (*Blautia* species stay below 1.3%, *Bifidobacterium* species reach 19.6%).

</details>

---

## What to take away

- A TreeSummarizedExperiment keeps samples in columns and taxa in rows: abundances in `assay()`, sample information in `colData()`, taxonomy in `rowData()`.
- A phyloseq object converts into a TSE with `convertFromPhyloseq()`.
- One TSE can hold several assays (raw, transformed), dissimilarities and ordinations, all kept together and subset together.
- The choice of transformation and distance changes what you see: look at more than one.

This is the end of the R and RStudio module. To go further, the [Orchestrating Microbiome Analysis](https://bioconductor.org/books/release/OMA/) online book covers the miaverse in depth.
