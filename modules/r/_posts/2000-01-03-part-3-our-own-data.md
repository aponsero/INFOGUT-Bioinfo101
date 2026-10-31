---
title: "Part 3: Our own data"
---

# Part 3: Our own data
{: .no_toc}

<div class="panel" markdown="1">
**Estimated time:** 30 to 45 minutes.<br>
**You need:** R and RStudio (no extra packages), and the session 1 notebook and data: [download r-session1.zip]({{ site.baseurl }}/modules/r/data/r-session1.zip). This is the same download as [Part 2]({{ site.baseurl }}/modules/r/part-2-first-steps-in-r/): this page covers sections 10 to 12 of the notebook.<br>
**Before you start:** this page uses `$`, `[ , ]`, `==`, `table()` and `str()` from [Part 2]({{ site.baseurl }}/modules/r/part-2-first-steps-in-r/).<br>
**Authors:** Andrés Méndez Hurtado, Adrián Martín Segura, Alba Pérez Cuervo, Laura Judith Marcos Zambrano, Matti Ruuskanen.
</div>

**On this page**

* TOC
{:toc}

---

## 10. The experiment we are going to analyse

Two food products (dairy fermentates, in powder form) were tested to see whether they could help control blood glucose. Each product went through three stages:

| Stage | What happens |
|---|---|
| 1. Artificial digestion | Mouth, stomach and intestine are reproduced in the lab, to simulate what would happen if the product were eaten. |
| 2. Fermentation | 24 hours with human gut bacteria. The bacteria transform the product and produce organic acids, mainly short chain fatty acids: butyric, propionic and acetic acid. |
| 3. Pancreatic cells | The resulting liquid is put in contact with human pancreatic cells, and the insulin they release is measured. |

**Hypothesis:** if a product makes the bacteria produce more butyric and propionic acid, the cells should release more insulin.

### The design: 3 conditions, 2 replicates

| Condition | What it is |
|---|---|
| Control | The bacteria with no product added |
| A | Product A (high in lactose, low in protein) |
| B | Product B (very high in protein, no lactose) |

Each condition was fermented in **two independent vessels**, R1 and R2. These are **biological replicates**: two real repetitions of the experiment. That gives 6 samples in total:

| | Replicate 1 | Replicate 2 |
|---|---|---|
| Control | Control_R1 | Control_R2 |
| A | A_R1 | A_R2 |
| B | B_R1 | B_R2 |

### Not all measurements are taken at the same stage

| Measurement | How many times | Number of values |
|---|---|---|
| Acids and bacteria | Once in each vessel | 6 vessels = 6 measurements |
| Insulin | The liquid from each vessel was put on the cells on 3 separate occasions, with a different batch of cells each time (the `experiment` column) | 6 vessels x 3 = 18 measurements |

So the 3 insulin experiments are **not** 3 new fermentations: they are the **same** fermented liquid measured three times. They tell us how repeatable the cell assay is, not how repeatable the biology is. Telling those two things apart is one of the goals of this course, and Part 5 comes back to it.

---

## 11. Reading a CSV file

A CSV is a plain text file where values are separated by commas. You can open it with Notepad or any text editor. The function to read one is `read.csv()`.

```r
acids     <- read.csv("input/acids.csv")
insulin   <- read.csv("input/insulin.csv")
abundance <- read.csv("input/abundance.csv")
```

Nothing is printed, but three tables appear in the **Environment** pane.

`"input/acids.csv"` means: from the folder I am standing in, go into `input` and take `acids.csv`. This is a **relative path**. It is better than writing the full path, because the code then works on any computer, as long as the folder keeps its organisation.

If `read.csv()` fails with `cannot open file`, nine times out of ten the working directory is wrong. Check `getwd()` and `list.files()` first.

### The four files

| File | Contents | Rows |
|---|---|---|
| `acids.csv` | The six organic acids measured in each sample, in µg/ml | 6: one per sample |
| `insulin.csv` | Insulin released by the cells, in ng per million cells and hour | 18: 6 samples x 3 experiments |
| `abundance.csv` | Which bacteria were in each sample, as a % of the total. All 190 species detected in the study | 190: one per species |
| `taxonomy.csv` | The record card of each species: kingdom, phylum, class, order, family and genus | 190: one per species |

### input/acids.csv

```r
acids
```

```
      sample product replicate   formic   acetic propionic   lactic  butyric  citric
1 Control_R1 Control         1 34660.87 2485.879   305.614 4585.646  398.641 640.505
2 Control_R2 Control         2 35620.77 2367.957   567.952 4379.197  584.875 659.495
3       A_R1       A         1 29186.73 3587.407   407.037 3509.921 1635.671 711.395
4       A_R2       A         2 29401.88 3189.795   633.849 2614.445  839.359 991.664
5       B_R1       B         1 22333.59 3562.859  2431.971  358.035 2383.543 672.428
6       B_R2       B         2 26137.49 3331.052  1889.117  371.603 2255.989 580.663
```

Six rows, one per sample, and nine columns: three that describe the sample, then one per acid. Now look at it with the functions from Part 2:

```r
str(acids)
```

```
'data.frame':	6 obs. of  9 variables:
 $ sample   : chr  "Control_R1" "Control_R2" "A_R1" "A_R2" ...
 $ product  : chr  "Control" "Control" "A" "A" ...
 $ replicate: int  1 2 1 2 1 2
 $ formic   : num  34661 35621 29187 29402 22334 ...
 $ acetic   : num  2486 2368 3587 3190 3563 ...
 $ propionic: num  306 568 407 634 2432 ...
 $ lactic   : num  4586 4379 3510 2614 358 ...
 $ butyric  : num  399 585 1636 839 2384 ...
 $ citric   : num  641 659 711 992 672 ...
```

All the acid columns arrived as numbers (`num`). If one of them had arrived as `chr`, something in the file would not be a number, and that is the moment to find out.

```r
dim(acids)
names(acids)
```

```
[1] 6 9
[1] "sample"    "product"   "replicate" "formic"    "acetic"    "propionic" "lactic"   
[8] "butyric"   "citric"   
```

```r
summary(acids)
```

```
    sample            product            replicate       formic          acetic    
 Length:6           Length:6           Min.   :1.0   Min.   :22334   Min.   :2368  
 Class :character   Class :character   1st Qu.:1.0   1st Qu.:26900   1st Qu.:2662  
 Mode  :character   Mode  :character   Median :1.5   Median :29294   Median :3260  
                                       Mean   :1.5   Mean   :29557   Mean   :3087  
                                       3rd Qu.:2.0   3rd Qu.:33346   3rd Qu.:3505  
                                       Max.   :2.0   Max.   :35621   Max.   :3587  
   propionic          lactic          butyric           citric     
 Min.   : 305.6   Min.   : 358.0   Min.   : 398.6   Min.   :580.7  
 1st Qu.: 447.3   1st Qu.: 932.3   1st Qu.: 648.5   1st Qu.:645.3  
 Median : 600.9   Median :3062.2   Median :1237.5   Median :666.0  
 Mean   :1039.3   Mean   :2636.5   Mean   :1349.7   Mean   :709.4  
 3rd Qu.:1575.3   3rd Qu.:4161.9   3rd Qu.:2100.9   3rd Qu.:701.7  
 Max.   :2432.0   Max.   :4585.6   Max.   :2383.5   Max.   :991.7  
```

Two last checks: how many samples per product, and are there missing values?

```r
table(acids$product)
sum(is.na(acids))       # any missing values? (should be 0)
```

```

      A       B Control 
      2       2       2 
[1] 0
```

`is.na()` gives `TRUE` for every missing value, and `sum()` counts the `TRUE`s, exactly like `sum(numbers > 3)` in Part 2.

### input/insulin.csv

Insulin released by the cells, in ng per million cells and hour. 6 samples x 3 experiments = 18 rows.

```r
head(insulin)
nrow(insulin)
```

```
      sample product replicate experiment insulin
1 Control_R1 Control         1          1  33.882
2 Control_R1 Control         1          2  40.650
3 Control_R1 Control         1          3  73.967
4 Control_R2 Control         2          1  44.441
5 Control_R2 Control         2          2  65.467
6 Control_R2 Control         2          3  80.851
[1] 18
```

`table()` with two columns checks the design in one line: every product should appear twice (R1 and R2) in each of the three experiments.

```r
table(insulin$product, insulin$experiment)
```

```
         
          1 2 3
  A       2 2 2
  B       2 2 2
  Control 2 2 2
```

### input/abundance.csv

Which bacteria were in each sample, as a % of the total. All 190 species detected in the study, one per row.

```r
dim(abundance)
head(abundance)
```

```
[1] 190   7
                       species Control_R1 Control_R2    A_R1    A_R2    B_R1    B_R2
1      Bifidobacterium bifidum     7.1296     5.0689 12.3905  6.5088 12.3610 19.6386
2    Lactobacillus acidophilus     0.0000     1.5076  3.2884  3.0498  1.4205  7.8599
3 Bifidobacterium adolescentis     9.4723     2.5209 13.0599 11.8849  9.1369 10.9719
4       Bifidobacterium longum     9.9385     4.5449 10.9524 13.7371  9.5213 10.0156
5  Catenibacterium tridentinum     2.5612     0.3143  2.0679  2.1465  2.1397  2.1901
6             Escherichia coli    32.3609    56.7068  8.6475 11.1042 11.3136 15.2895
```

**Careful:** 190 rows. Typing `abundance` on its own floods the console. With big tables, look at them with `dim()` and `head()` instead.

Two things worth knowing about this table.

**1. The columns do not add up to exactly 100.**

```r
colSums(abundance[, -1])
```

```
Control_R1 Control_R2       A_R1       A_R2       B_R1       B_R2 
   97.0681    87.6628   100.0000    98.4569    95.3876    99.9998 
```

`colSums()` adds up each column. The `[, -1]` drops the first column (`species`), which is text: `colSums()` cannot add up words.

Why not 100? To find out which bacteria are in a sample, its DNA is sequenced and the pieces are compared against a database of known species. Some pieces do not match anything and stay **unclassified**, so they cannot appear in the table: we do not know which species they belong to. Each column therefore adds up to 100 minus the % of unclassified reads, which in this study ranges from 0% to 12.3%. It is not an error.

**2. Some species are 0 in all six samples.** They were found in the original study, which had 26 samples, but not in the six used here.

```r
sum(rowSums(abundance[, -1]) == 0)
```

```
[1] 28
```

`rowSums()` adds up each row; `== 0` asks which totals are zero; `sum()` counts them. 28 of the 190 species are absent from all six samples.

### input/taxonomy.csv

`abundance.csv` tells you **how much** of each species there is. `taxonomy.csv` tells you **what** each species is: kingdom, phylum, class, order, family and genus. It has 190 rows, in the **same order** as `abundance.csv`: row 7 of `taxonomy` describes row 7 of `abundance`, which is what lets you match the two tables species by species.

```r
taxonomy <- read.csv("input/taxonomy.csv")
head(taxonomy)
dim(taxonomy)
```

```
                       species  kingdom         phylum               class
1      Bifidobacterium bifidum Bacteria Actinomycetota       Actinomycetes
2    Lactobacillus acidophilus Bacteria     Firmicutes             Bacilli
3 Bifidobacterium adolescentis Bacteria Actinomycetota       Actinomycetes
4       Bifidobacterium longum Bacteria Actinomycetota       Actinomycetes
5  Catenibacterium tridentinum Bacteria     Firmicutes    Erysipelotrichia
6             Escherichia coli Bacteria Pseudomonadota Gammaproteobacteria
               order             family           genus
1  Bifidobacteriales Bifidobacteriaceae Bifidobacterium
2    Lactobacillales   Lactobacillaceae   Lactobacillus
3  Bifidobacteriales Bifidobacteriaceae Bifidobacterium
4  Bifidobacteriales Bifidobacteriaceae Bifidobacterium
5 Erysipelotrichales   Coprobacillaceae Catenibacterium
6   Enterobacterales Enterobacteriaceae     Escherichia
[1] 190   7
```

When a table is too wide for the console, R prints the remaining columns underneath, with the row numbers repeated.

How many species belong to each phylum?

```r
table(taxonomy$phylum)
```

```

         Actinomycetota            Bacteroidota              Firmicutes 
                     20                      33                     121 
         Fusobacteriota       Methanobacteriota          Mycoplasmatota 
                      2                       1                       1 
         Pseudomonadota Thermodesulfobacteriota       Verrucomicrobiota 
                      9                       2                       1 
```

All the species of the genus *Bifidobacterium*:

```r
taxonomy[taxonomy$genus == "Bifidobacterium", ]
```

```
                              species  kingdom         phylum         class
1             Bifidobacterium bifidum Bacteria Actinomycetota Actinomycetes
3        Bifidobacterium adolescentis Bacteria Actinomycetota Actinomycetes
4              Bifidobacterium longum Bacteria Actinomycetota Actinomycetes
14           Bifidobacterium animalis Bacteria Actinomycetota Actinomycetes
15  Bifidobacterium pseudocatenulatum Bacteria Actinomycetota Actinomycetes
99        Bifidobacterium catenulatum Bacteria Actinomycetota Actinomycetes
153         Bifidobacterium angulatum Bacteria Actinomycetota Actinomycetes
                order             family           genus
1   Bifidobacteriales Bifidobacteriaceae Bifidobacterium
3   Bifidobacteriales Bifidobacteriaceae Bifidobacterium
4   Bifidobacteriales Bifidobacteriaceae Bifidobacterium
14  Bifidobacteriales Bifidobacteriaceae Bifidobacterium
15  Bifidobacteriales Bifidobacteriaceae Bifidobacterium
99  Bifidobacteriales Bifidobacteriaceae Bifidobacterium
153 Bifidobacteriales Bifidobacteriaceae Bifidobacterium
```

Nothing new here: `table()` and `[ , ]` are exactly the tools from Part 2. The row numbers on the left are the original positions of these species in the table.

This is how a species-level result becomes a biological story: not "species X went up", but "the *Bifidobacterium* genus went up".

### Exercise 5

Using what you learned in Part 2 (no extra packages):

- a) pull out the `butyric` column of `acids`;
- b) compute the mean butyric acid of the 6 samples;
- c) keep only the rows of product `"B"`.

You need: `$`, `mean()`, `[ , ]`, `==`.

<details markdown="1">
<summary>Answer</summary>

```r
acids$butyric
mean(acids$butyric)
acids[acids$product == "B", ]
```

```
[1]  398.641  584.875 1635.671  839.359 2383.543 2255.989
[1] 1349.68
  sample product replicate   formic   acetic propionic  lactic  butyric  citric
5   B_R1       B         1 22333.59 3562.859  2431.971 358.035 2383.543 672.428
6   B_R2       B         2 26137.49 3331.052  1889.117 371.603 2255.989 580.663
```

</details>

### Exercise 6

Which sample has the **most** butyric acid? Hint: use `max()`, then a condition.

You need: `max()`, `==`, `[ , ]`.

<details markdown="1">
<summary>Answer</summary>

```r
max(acids$butyric)
acids[acids$butyric == max(acids$butyric), ]
```

```
[1] 2383.543
  sample product replicate   formic   acetic propionic  lactic  butyric  citric
5   B_R1       B         1 22333.59 3562.859  2431.971 358.035 2383.543 672.428
```

B_R1. The second line puts the calculation inside the condition: R first computes `max(acids$butyric)`, then keeps the row whose butyric value is equal to it.

</details>

---

## 12. Saving results: the output folder

**Golden rule: the input files are never modified.** Everything R produces goes into `output`. Results are disposable: they can be regenerated by re-running the code. The raw data cannot.

For example, save a small table with only the rows of product B:

```r
only_B <- acids[acids$product == "B", ]
write.csv(only_B, "output/product_B_only.csv", row.names = FALSE)
list.files("output")   # check that it is really there
```

```
[1] "product_B_only.csv"
```

`row.names = FALSE` stops R from adding an extra first column with 1, 2, 3... It is almost always what you want.

### Exercise 7

Save a file called `acids_control.csv` in `output`, containing only the Control rows. Then check that it is there.

You need: `$`, `[ , ]`, `==`, `write.csv()`, `list.files()`.

<details markdown="1">
<summary>Answer</summary>

```r
control_rows <- acids[acids$product == "Control", ]
write.csv(control_rows, "output/acids_control.csv", row.names = FALSE)
list.files("output")
```

```
[1] "acids_control.csv"  "product_B_only.csv"
```

You can also open the file from the **Files** pane to see what it contains.

</details>

---

## What to take away

| Code | What it does |
|---|---|
| `<-` | Store something in a variable |
| `c()` | Build a vector |
| `[ ]` | Positions, and conditions |
| `class()` | What type of data it is |
| `table()` | Count categories |
| `data.frame()` | The table of R |
| `$` | One column |
| `str()`, `head()`, `summary()` | Look at the data before analysing it |
| `read.csv("input/file.csv")` | Read data |
| `write.csv(..., "output/...")` | Save results |

And remember: never touch the input files.

**Next:** Part 4, handling tables with `dplyr`.
