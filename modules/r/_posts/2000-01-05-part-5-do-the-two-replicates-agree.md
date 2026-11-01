---
title: "Part 5: Do the two replicates agree?"
---

# Part 5: Do the two replicates agree?
{: .no_toc}

<div class="panel" markdown="1">
**Estimated time:** 20 to 30 minutes.<br>
**You need:** R and RStudio with the `dplyr` package, and the session 2 notebook and data: [download r-session2.zip]({{ site.baseurl }}/modules/r/data/r-session2.zip). This is the same download as [Part 4]({{ site.baseurl }}/modules/r/part-4-handling-tables-with-dplyr/): this page covers sections 4 and 5 of the notebook.<br>
**Before you start:** this page uses `filter()`, `select()` and `data.frame()`. If you start here, run the setup chunk at the top of the notebook first, so that `dplyr` is loaded and `acids` is read.<br>
**Authors:** Andrés Méndez Hurtado, Adrián Martín Segura, Alba Pérez Cuervo, Laura Judith Marcos Zambrano, Matti Ruuskanen.
</div>

Before you summarise anything, check that the experiment repeats.

**On this page**

* TOC
{:toc}

---

## 4. Do the two replicates agree?

Each condition was fermented twice, separately (R1 and R2). Before summarising anything, you have to check whether the two give the same answer. If they do, the experiment is reproducible.

### Biological replicates and technical repetitions

Both are "repeating the measurement". They do not mean the same thing, and they do not count the same.

| | Biological replicate | Technical repetition |
|---|---|---|
| In our data | R1 and R2: the `replicate` column | 1, 2, 3: the `experiment` column of the insulin table |
| What it is | Two separate fermentation vessels, inoculated separately and left to ferment on their own: two real repetitions of the experiment | The same fermented liquid put on the cells three times, with a different batch of cells each time |
| What it measures | How repeatable the **biology** is | How repeatable the **assay** is: how good your technique is |

Technical repetitions get averaged into a single value; they never increase your sample size. **Treating them as independent samples inflates n and hands you significance that is not real.**

### Comparing the two replicates, acid by acid

We will compare ALL SIX acids, not just one. If we only looked at one, the answer would depend on which one we happened to pick.

**Step 1:** split the two replicates into two tables.

```r
r1 <- filter(acids, replicate == 1)     # Control_R1, A_R1, B_R1
r2 <- filter(acids, replicate == 2)     # Control_R2, A_R2, B_R2

r1
r2
```

```
      sample product replicate   formic   acetic propionic   lactic  butyric  citric
1 Control_R1 Control         1 34660.87 2485.879   305.614 4585.646  398.641 640.505
2       A_R1       A         1 29186.73 3587.407   407.037 3509.921 1635.671 711.395
3       B_R1       B         1 22333.59 3562.859  2431.971  358.035 2383.543 672.428
      sample product replicate   formic   acetic propionic   lactic  butyric  citric
1 Control_R2 Control         2 35620.77 2367.957   567.952 4379.197  584.875 659.495
2       A_R2       A         2 29401.88 3189.795   633.849 2614.445  839.359 991.664
3       B_R2       B         2 26137.49 3331.052  1889.117  371.603 2255.989 580.663
```

If you did the exercises of Part 4, the `scfa` column also appears, under each table. It does not matter here: the next step keeps only the six acids.

Same products, same columns, same order in both. That is what makes the rest work.

**Step 2:** keep only the six acid columns of each table.

```r
values_r1 <- select(r1, formic:citric)
values_r2 <- select(r2, formic:citric)

values_r1
values_r2
```

```
    formic   acetic propionic   lactic  butyric  citric
1 34660.87 2485.879   305.614 4585.646  398.641 640.505
2 29186.73 3587.407   407.037 3509.921 1635.671 711.395
3 22333.59 3562.859  2431.971  358.035 2383.543 672.428
    formic   acetic propionic   lactic  butyric  citric
1 35620.77 2367.957   567.952 4379.197  584.875 659.495
2 29401.88 3189.795   633.849 2614.445  839.359 991.664
3 26137.49 3331.052  1889.117  371.603 2255.989 580.663
```

**Step 3:** compare them, one idea per line.

When you subtract two tables of the same size, R subtracts them cell by cell: the formic acid of Control_R1 minus the formic acid of Control_R2, and so on for all 18 numbers.

```r
gap <- abs(values_r1 - values_r2)     # HOW FAR APART the two are
gap
```

```
    formic  acetic propionic  lactic butyric  citric
1  959.903 117.922   262.338 206.449 186.234  18.990
2  215.158 397.612   226.812 895.476 796.312 280.269
3 3803.899 231.807   542.854  13.568 127.554  91.765
```

Why `abs()`? Without it, we would get negative numbers whenever R2 is the bigger one. The minus sign only tells us WHICH one was bigger, and we do not care about that: we want to know how far apart they are. `abs()` (absolute value) drops the sign and keeps the size. Try it without `abs()` and compare.

A gap on its own is hard to judge: 960 µg/ml is small for formic acid (around 35,000) but would be huge for citric acid (around 650). So we compare each gap with the typical value of that acid in that condition:

```r
average <- (values_r1 + values_r2) / 2   # the TYPICAL VALUE there
average
```

```
    formic   acetic propionic   lactic  butyric   citric
1 35140.82 2426.918   436.783 4482.422  491.758 650.0000
2 29294.31 3388.601   520.443 3062.183 1237.515 851.5295
3 24235.54 3446.956  2160.544  364.819 2319.766 626.5455
```

**Careful:** this is not the mean of an acid across samples. It is the midpoint between the two replicates of that one acid in that one condition.

```r
diff_pct <- round(100 * gap / average, 1)   # the gap, as a % of the typical value
diff_pct
```

```
  formic acetic propionic lactic butyric citric
1    2.7    4.9      60.1    4.6    37.9    2.9
2    0.7   11.7      43.6   29.2    64.3   32.9
3   15.7    6.7      25.1    3.7     5.5   14.6
```

And glue the product names back on, to read it easily:

```r
comparison <- data.frame(product = r1$product, diff_pct)
comparison
```

```
  product formic acetic propionic lactic butyric citric
1 Control    2.7    4.9      60.1    4.6    37.9    2.9
2       A    0.7   11.7      43.6   29.2    64.3   32.9
3       B   15.7    6.7      25.1    3.7     5.5   14.6
```

### Reading the table (discussion, no code)

Each number means: "the two replicates differ by this percentage of the typical value". Small means they agree. Read the table two ways: along a row, is this condition reproducible? Down a column, is this acid reliable?

- a) Look at the three rows. Which product has the most reproducible replicates, that is, the smallest numbers?
- b) Now look at the six columns. Which acids give the biggest numbers?

<details markdown="1">
<summary>Discussion</summary>

**a)** No product wins on every acid, which is exactly why it is worth looking at all six.

- **B** has no value above 25.1%: its replicates never disagree badly.
- **Control** agrees within 5% for four acids, but propionic (60.1%) and butyric (37.9%) differ a lot.
- **A** has the largest numbers overall: four acids above 10%, and butyric at 64.3%.

**b)** Propionic acid gives big numbers in all three products (60.1, 43.6 and 25.1%). Butyric acid gives big numbers in the Control (37.9%) and in A (64.3%), but in B its two replicates agree within 5.5%. In A, lactic (29.2%) and citric (32.9%) also differ noticeably.

</details>

---

## 5. The same thing as a picture

This is how the original paper presented these data: a bar for the mean, an error bar for the standard deviation, and the individual replicates as dots.

<figure>
<img src="{{ site.baseurl }}/modules/r/images/replicates-per-acid.svg" alt="Six panels, one per acid, each with a bar for Control, A and B. Bars show the mean of the two replicates, error bars the standard deviation, and dots the two replicates. The two dots are close together for formic, acetic and lactic acid in most conditions, and far apart for propionic acid in all conditions and for butyric acid in the Control and A.">
<figcaption>The six acids in the three conditions, drawn from <code>acids.csv</code>. Each panel has its own scale.</figcaption>
</figure>

Look at the distance between the two dots of each bar: it is the same information as the table above. Where the dots sit on top of each other, the replicates agree; where they are far apart, the error bar is long.

**Conclusions.** Formic and acetic acid repeat closely in every condition, and lactic acid too, except in product A. Propionic acid is the noisy one in every condition, and butyric acid in the Control and A. The authors of the original paper report all these acids descriptively, without statistics: with two wells, there is not much else you can honestly do.

The lesson: reproducibility is not the same in every condition or for every measurement, and you have to LOOK at it rather than assume it.

---

## What to take away

- A biological replicate repeats the experiment; a technical repetition repeats the measurement. Only biological replicates increase your sample size.
- Check the replicates BEFORE summarising them.
- Look at all the measurements, not just one: the answer can change from one acid, or one condition, to the next.
- Subtracting two tables of the same size works cell by cell, which makes it easy to compare replicates for every measurement at once.

**Next:** [Part 6: Microbiome data in R]({{ site.baseurl }}/modules/r/part-6-microbiome-data-in-r/).
