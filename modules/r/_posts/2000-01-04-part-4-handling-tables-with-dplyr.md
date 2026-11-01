---
title: "Part 4: Handling tables with dplyr"
---

# Part 4: Handling tables with dplyr
{: .no_toc}

<div class="panel" markdown="1">
**Estimated time:** 60 minutes.<br>
**You need:** R and RStudio with the `dplyr` package (see [Setup]({{ site.baseurl }}/modules/about/setup/#install-the-r-packages)), and the session 2 notebook and data: [download r-session2.zip]({{ site.baseurl }}/modules/r/data/r-session2.zip). The same download is used in [Part 5]({{ site.baseurl }}/modules/r/part-5-do-the-two-replicates-agree/).<br>
**Before you start:** this page assumes you know variables, data frames, `$`, `==` and `read.csv()` from [Part 2]({{ site.baseurl }}/modules/r/part-2-first-steps-in-r/) and [Part 3]({{ site.baseurl }}/modules/r/part-3-our-own-data/). The data are described in Part 3, section 10.<br>
**Authors:** Andrés Méndez Hurtado, Adrián Martín Segura, Alba Pérez Cuervo, Laura Judith Marcos Zambrano, Matti Ruuskanen.
</div>

Same data as last time. Now we make it talk.

**On this page**

* TOC
{:toc}

---

## How to use this page

1. [Download r-session2.zip]({{ site.baseurl }}/modules/r/data/r-session2.zip) and unzip it. You get a folder called `r-session2` containing the notebook `R_course_session2_STUDENT.Rmd`, an `input` folder with `acids.csv`, and an empty `output` folder.
2. Open the notebook in RStudio (**File > Open File**).
3. This page covers sections 1 and 2 of the notebook. [Part 5]({{ site.baseurl }}/modules/r/part-5-do-the-two-replicates-agree/) covers the rest.

On this page, R's answers are shown as they appear in the console. In a notebook, tables appear instead as a small interactive table below the chunk, which you can page through: the content is the same.

Try each exercise in the notebook before opening the answer.

---

## Setup

```r
# install.packages("dplyr")    # only the first time

library(dplyr)

acids <- read.csv("input/acids.csv")

acids
dim(acids)     # 6 rows (the samples) and 9 columns
```

When you load `dplyr`, R prints a message:

```
Attaching package: 'dplyr'

The following objects are masked from 'package:stats':

    filter, lag

The following objects are masked from 'package:base':

    intersect, setdiff, setequal, union
```

This is not an error. It says that `dplyr` has its own functions called `filter()`, `intersect()` and so on, and that from now on these are the ones R will use: the "last one loaded wins" rule from [Part 1]({{ site.baseurl }}/modules/r/part-1-getting-to-know-r-and-rstudio/#packages-extending-what-r-can-do).

```
      sample product replicate   formic   acetic propionic   lactic  butyric  citric
1 Control_R1 Control         1 34660.87 2485.879   305.614 4585.646  398.641 640.505
2 Control_R2 Control         2 35620.77 2367.957   567.952 4379.197  584.875 659.495
3       A_R1       A         1 29186.73 3587.407   407.037 3509.921 1635.671 711.395
4       A_R2       A         2 29401.88 3189.795   633.849 2614.445  839.359 991.664
5       B_R1       B         1 22333.59 3562.859  2431.971  358.035 2383.543 672.428
6       B_R2       B         2 26137.49 3331.052  1889.117  371.603 2255.989 580.663
[1] 6 9
```

One row per fermentation vessel: the six organic acids measured once in each, in µg/ml.

If `library(dplyr)` fails with `there is no package called 'dplyr'`, the package is not installed on this computer: run `install.packages("dplyr")` once, then try again.

---

## 1. The six dplyr verbs

`dplyr` is a package for handling tables. It is useful for the things you do with data every day: keeping only the rows you want, choosing columns, sorting, creating new columns, and calculating summaries per group. You could already do most of this with base R in Part 2, but `dplyr` makes it simpler and easier to read.

| Verb | What it does |
|---|---|
| `filter()` | Picks ROWS: trims the table vertically |
| `select()` | Picks COLUMNS: trims the table horizontally |
| `arrange()` | SORTS the rows: same table, different order |
| `mutate()` | ADDS a column, computed from the others |
| `summarise()` | REDUCES to one number: many rows become one |
| `group_by()` | Does the above PER GROUP |

They are all written the same way, and the first argument is ALWAYS the table:

```r
verb(table, what_I_want)
```

Column names are written bare, without quotation marks.

### filter(): picks rows

You give it a condition and it keeps the rows that meet it. The columns are untouched.

```r
# "give me the rows whose product is B"
filter(acids, product == "B")
#   from 6 rows -> 2 remain

# "give me the rows with more than 1000 ug/ml of butyric acid"
filter(acids, butyric > 1000)
#   from 6 rows -> 3 remain
```

```
  sample product replicate   formic   acetic propionic  lactic  butyric  citric
1   B_R1       B         1 22333.59 3562.859  2431.971 358.035 2383.543 672.428
2   B_R2       B         2 26137.49 3331.052  1889.117 371.603 2255.989 580.663
  sample product replicate   formic   acetic propionic   lactic  butyric  citric
1   A_R1       A         1 29186.73 3587.407   407.037 3509.921 1635.671 711.395
2   B_R1       B         1 22333.59 3562.859  2431.971  358.035 2383.543 672.428
3   B_R2       B         2 26137.49 3331.052  1889.117  371.603 2255.989 580.663
```

Compare with Part 2: `acids[acids$product == "B", ]` gives the same rows. With `filter()` there is no `$`, no square brackets and no comma to forget.

`filter()` only shows the result. To keep it, you need `<-`, as always.

**Careful:** to compare you write a double equals sign `==`. A single `=` means something else entirely.

#### Exercise 1a

Keep only the Control rows.

<details markdown="1">
<summary>Answer</summary>

```r
filter(acids, product == "Control")
```

```
      sample product replicate   formic   acetic propionic   lactic butyric  citric
1 Control_R1 Control         1 34660.87 2485.879   305.614 4585.646 398.641 640.505
2 Control_R2 Control         2 35620.77 2367.957   567.952 4379.197 584.875 659.495
```

</details>

#### Exercise 1b

Now a filter with a number instead of a word: keep only the rows whose acetic acid is above 3000.

<details markdown="1">
<summary>Answer</summary>

```r
filter(acids, acetic > 3000)
```

```
  sample product replicate   formic   acetic propionic   lactic  butyric  citric
1   A_R1       A         1 29186.73 3587.407   407.037 3509.921 1635.671 711.395
2   A_R2       A         2 29401.88 3189.795   633.849 2614.445  839.359 991.664
3   B_R1       B         1 22333.59 3562.859  2431.971  358.035 2383.543 672.428
4   B_R2       B         2 26137.49 3331.052  1889.117  371.603 2255.989 580.663
```

Four rows: all the samples with product A or B. Both Control samples are below 2500.

</details>

### select(): picks columns

The opposite of `filter()`. All the rows are kept; you choose which columns to see.

```r
# "give me only these two columns"
select(acids, sample, butyric)
#   still 6 rows, but out of 9 columns only 2 remain
```

```
      sample  butyric
1 Control_R1  398.641
2 Control_R2  584.875
3       A_R1 1635.671
4       A_R2  839.359
5       B_R1 2383.543
6       B_R2 2255.989
```

A minus sign removes a column, and a colon gives a range of columns:

```r
select(acids, -formic)        # every column except formic
select(acids, formic:citric)  # from formic to citric: the six acids
```

```
      sample product replicate   acetic propionic   lactic  butyric  citric
1 Control_R1 Control         1 2485.879   305.614 4585.646  398.641 640.505
2 Control_R2 Control         2 2367.957   567.952 4379.197  584.875 659.495
3       A_R1       A         1 3587.407   407.037 3509.921 1635.671 711.395
4       A_R2       A         2 3189.795   633.849 2614.445  839.359 991.664
5       B_R1       B         1 3562.859  2431.971  358.035 2383.543 672.428
6       B_R2       B         2 3331.052  1889.117  371.603 2255.989 580.663
    formic   acetic propionic   lactic  butyric  citric
1 34660.87 2485.879   305.614 4585.646  398.641 640.505
2 35620.77 2367.957   567.952 4379.197  584.875 659.495
3 29186.73 3587.407   407.037 3509.921 1635.671 711.395
4 29401.88 3189.795   633.849 2614.445  839.359 991.664
5 22333.59 3562.859  2431.971  358.035 2383.543 672.428
6 26137.49 3331.052  1889.117  371.603 2255.989 580.663
```

The columns come out in the order you ask for them. `filter()` trims vertically, `select()` trims horizontally: between the two you can reach any part of a table.

#### Exercise 2a

Keep only the columns `sample`, `product` and `acetic`.

<details markdown="1">
<summary>Answer</summary>

```r
select(acids, sample, product, acetic)
```

```
      sample product   acetic
1 Control_R1 Control 2485.879
2 Control_R2 Control 2367.957
3       A_R1       A 3587.407
4       A_R2       A 3189.795
5       B_R1       B 3562.859
6       B_R2       B 3331.052
```

</details>

#### Exercise 2b

Now the other way round: show every column EXCEPT `propionic` (use the minus sign).

<details markdown="1">
<summary>Answer</summary>

```r
select(acids, -propionic)
```

```
      sample product replicate   formic   acetic   lactic  butyric  citric
1 Control_R1 Control         1 34660.87 2485.879 4585.646  398.641 640.505
2 Control_R2 Control         2 35620.77 2367.957 4379.197  584.875 659.495
3       A_R1       A         1 29186.73 3587.407 3509.921 1635.671 711.395
4       A_R2       A         2 29401.88 3189.795 2614.445  839.359 991.664
5       B_R1       B         1 22333.59 3562.859  358.035 2383.543 672.428
6       B_R2       B         2 26137.49 3331.052  371.603 2255.989 580.663
```

</details>

### arrange(): sorts the rows

It removes nothing and adds nothing: it only changes the order.

```r
# sort the samples by butyric acid
arrange(acids, butyric)          # from lowest to highest
arrange(acids, desc(butyric))    # desc = highest to lowest
```

```
      sample product replicate   formic   acetic propionic   lactic  butyric  citric
1 Control_R1 Control         1 34660.87 2485.879   305.614 4585.646  398.641 640.505
2 Control_R2 Control         2 35620.77 2367.957   567.952 4379.197  584.875 659.495
3       A_R2       A         2 29401.88 3189.795   633.849 2614.445  839.359 991.664
4       A_R1       A         1 29186.73 3587.407   407.037 3509.921 1635.671 711.395
5       B_R2       B         2 26137.49 3331.052  1889.117  371.603 2255.989 580.663
6       B_R1       B         1 22333.59 3562.859  2431.971  358.035 2383.543 672.428
      sample product replicate   formic   acetic propionic   lactic  butyric  citric
1       B_R1       B         1 22333.59 3562.859  2431.971  358.035 2383.543 672.428
2       B_R2       B         2 26137.49 3331.052  1889.117  371.603 2255.989 580.663
3       A_R1       A         1 29186.73 3587.407   407.037 3509.921 1635.671 711.395
4       A_R2       A         2 29401.88 3189.795   633.849 2614.445  839.359 991.664
5 Control_R2 Control         2 35620.77 2367.957   567.952 4379.197  584.875 659.495
6 Control_R1 Control         1 34660.87 2485.879   305.614 4585.646  398.641 640.505
```

Sorting is how you find the extremes without computing anything: in the first table, the first row has the lowest butyric acid and the last row the highest.

#### Exercise 3a

Sort the samples from highest to lowest propionic acid.

<details markdown="1">
<summary>Answer</summary>

```r
arrange(acids, desc(propionic))
```

```
      sample product replicate   formic   acetic propionic   lactic  butyric  citric
1       B_R1       B         1 22333.59 3562.859  2431.971  358.035 2383.543 672.428
2       B_R2       B         2 26137.49 3331.052  1889.117  371.603 2255.989 580.663
3       A_R2       A         2 29401.88 3189.795   633.849 2614.445  839.359 991.664
4 Control_R2 Control         2 35620.77 2367.957   567.952 4379.197  584.875 659.495
5       A_R1       A         1 29186.73 3587.407   407.037 3509.921 1635.671 711.395
6 Control_R1 Control         1 34660.87 2485.879   305.614 4585.646  398.641 640.505
```

If you have already added the `scfa` column (next section), it appears too, under the table.

</details>

#### Exercise 3b

Now without `desc()`: sort the samples from lowest to highest formic acid.

<details markdown="1">
<summary>Answer</summary>

```r
arrange(acids, formic)
```

```
      sample product replicate   formic   acetic propionic   lactic  butyric  citric
1       B_R1       B         1 22333.59 3562.859  2431.971  358.035 2383.543 672.428
2       B_R2       B         2 26137.49 3331.052  1889.117  371.603 2255.989 580.663
3       A_R1       A         1 29186.73 3587.407   407.037 3509.921 1635.671 711.395
4       A_R2       A         2 29401.88 3189.795   633.849 2614.445  839.359 991.664
5 Control_R1 Control         1 34660.87 2485.879   305.614 4585.646  398.641 640.505
6 Control_R2 Control         2 35620.77 2367.957   567.952 4379.197  584.875 659.495
```

</details>

### mutate(): adds a new column

It computes something from the existing columns and sticks it on at the end.

The "good" fatty acids, the two the hypothesis is about, are butyric and propionic:

```r
# add the two "good" acids into a single new column
mutate(acids, scfa = butyric + propionic)
#   an scfa column appears at the end: 9 columns -> 10
#   (scfa = short chain fatty acids)
```

```
      sample product replicate   formic   acetic propionic   lactic  butyric  citric
1 Control_R1 Control         1 34660.87 2485.879   305.614 4585.646  398.641 640.505
2 Control_R2 Control         2 35620.77 2367.957   567.952 4379.197  584.875 659.495
3       A_R1       A         1 29186.73 3587.407   407.037 3509.921 1635.671 711.395
4       A_R2       A         2 29401.88 3189.795   633.849 2614.445  839.359 991.664
5       B_R1       B         1 22333.59 3562.859  2431.971  358.035 2383.543 672.428
6       B_R2       B         2 26137.49 3331.052  1889.117  371.603 2255.989 580.663
      scfa
1  704.255
2 1152.827
3 2042.708
4 1473.208
5 4815.514
6 4145.106
```

The table is now too wide for the console, so R prints the last column underneath, with the row numbers repeated.

**Important:** `mutate()` only shows the result on screen. To keep the new column in the table, you have to store it with `<-`:

```r
acids <- mutate(acids, scfa = butyric + propionic)
acids
```

From now on, `acids` has 10 columns. The rest of the page assumes you ran this line.

#### Exercise 4a

Create a column `ratio` that is butyric divided by acetic.

<details markdown="1">
<summary>Answer</summary>

```r
mutate(acids, ratio = butyric / acetic)
```

```
      sample product replicate   formic   acetic propionic   lactic  butyric  citric
1 Control_R1 Control         1 34660.87 2485.879   305.614 4585.646  398.641 640.505
2 Control_R2 Control         2 35620.77 2367.957   567.952 4379.197  584.875 659.495
3       A_R1       A         1 29186.73 3587.407   407.037 3509.921 1635.671 711.395
4       A_R2       A         2 29401.88 3189.795   633.849 2614.445  839.359 991.664
5       B_R1       B         1 22333.59 3562.859  2431.971  358.035 2383.543 672.428
6       B_R2       B         2 26137.49 3331.052  1889.117  371.603 2255.989 580.663
      scfa     ratio
1  704.255 0.1603622
2 1152.827 0.2469956
3 2042.708 0.4559480
4 1473.208 0.2631389
5 4815.514 0.6689973
6 4145.106 0.6772602
```

To keep the column, store the result: `acids <- mutate(acids, ratio = butyric / acetic)`.

</details>

#### Exercise 4b

Now an easier one, just a sum: create a column `acetic_lactic` that adds acetic and lactic.

<details markdown="1">
<summary>Answer</summary>

```r
mutate(acids, acetic_lactic = acetic + lactic)
```

```
      sample product replicate   formic   acetic propionic   lactic  butyric  citric
1 Control_R1 Control         1 34660.87 2485.879   305.614 4585.646  398.641 640.505
2 Control_R2 Control         2 35620.77 2367.957   567.952 4379.197  584.875 659.495
3       A_R1       A         1 29186.73 3587.407   407.037 3509.921 1635.671 711.395
4       A_R2       A         2 29401.88 3189.795   633.849 2614.445  839.359 991.664
5       B_R1       B         1 22333.59 3562.859  2431.971  358.035 2383.543 672.428
6       B_R2       B         2 26137.49 3331.052  1889.117  371.603 2255.989 580.663
      scfa acetic_lactic
1  704.255      7071.525
2 1152.827      6747.154
3 2042.708      7097.328
4 1473.208      5804.240
5 4815.514      3920.894
6 4145.106      3702.655
```

</details>

### summarise(): reduces to a single number

It condenses all the rows into one. In Part 3 we would have written this:

```r
mean(acids$butyric)
```

```
[1] 1349.68
```

`summarise()` does exactly the same thing, but hands it back as a TABLE, with a column name that we choose:

```r
# the average butyric acid across all 6 samples, as a table
summarise(acids, mean_butyric = mean(butyric))
#   from 6 rows -> 1 row
```

```
  mean_butyric
1      1349.68
```

So why bother, if `mean()` already worked? Because of what comes next: `summarise()` can do this per group, and plain `mean()` cannot.

You can ask for several numbers at once:

```r
summarise(acids,
          mean_butyric   = mean(butyric),
          mean_propionic = mean(propionic))
```

```
  mean_butyric mean_propionic
1      1349.68       1039.257
```

#### Exercise 5

Calculate the mean acetic acid AND the highest acetic acid of the 6 samples, in one table.

<details markdown="1">
<summary>Answer</summary>

```r
summarise(acids,
          mean_acetic = mean(acetic),
          max_acetic  = max(acetic))
```

```
  mean_acetic max_acetic
1    3087.492   3587.407
```

</details>

### group_by(): doing it per group

Everything above gives ONE number for the whole table. But what we actually want is one number for the Control, one for A and one for B, so that we can compare them.

**First, how we would do it by hand.** Pick out one product, compute its mean. Then the next one.

```r
control_rows <- filter(acids, product == "Control")
mean(control_rows$butyric)

a_rows <- filter(acids, product == "A")
mean(a_rows$butyric)

b_rows <- filter(acids, product == "B")
mean(b_rows$butyric)
```

```
[1] 491.758
[1] 1237.515
[1] 2319.766
```

It works. But it took 6 lines for 3 products, the results come out loose instead of in a table, and with 20 products it would be unbearable.

**Now the same thing with `group_by()`:**

```r
# average butyric acid per product, in one table
by_product <- group_by(acids, product)
summarise(by_product, mean_butyric = mean(butyric))
```

```
# A tibble: 3 × 2
  product mean_butyric
  <chr>          <dbl>
1 A              1238.
2 B              2320.
3 Control         492.
```

Same three numbers, in one table, in two lines.

`group_by()` on its own displays nothing new: all it does is mark which rows belong together. The work is done by the `summarise()` that comes after it, which now runs once per group instead of once for the whole table.

**About the output.** The result is a **tibble**, the `dplyr` version of a data frame. Two differences you can see here:

- Under each column name, its type: `<chr>` for text (character), `<dbl>` for numbers with decimals.
- Numbers are shown with fewer digits, and the dot at the end of `1238.` means the decimals are hidden. The full value is still stored: 1237.515, as computed by hand above.

The groups come out in alphabetical order: A, B, Control.

#### Exercise 6a

Now the same thing, but with PROPIONIC acid: compute its mean per product. Two lines: first group, then summarise.

<details markdown="1">
<summary>Answer</summary>

```r
by_product <- group_by(acids, product)
summarise(by_product, mean_propionic = mean(propionic))
```

```
# A tibble: 3 × 2
  product mean_propionic
  <chr>            <dbl>
1 A                 520.
2 B                2161.
3 Control           437.
```

</details>

#### Exercise 6b

A different question: how MANY samples does each product have? Use `n()`, which counts the rows in each group.

<details markdown="1">
<summary>Answer</summary>

```r
by_product <- group_by(acids, product)
summarise(by_product, n_samples = n())
```

```
# A tibble: 3 × 2
  product n_samples
  <chr>       <int>
1 A               2
2 B               2
3 Control         2
```

`n()` takes nothing between its brackets: it simply counts the rows of each group. `<int>` means whole numbers (integers).

</details>

---

## 2. Chaining steps

One verb is almost never enough. To do several things in a row, you store each step in a variable with `<-` and pass it to the next one. It is like an assembly line.

```r
# step 1: keep only replicate 1's rows
r1_only <- filter(acids, replicate == 1)   # 6 rows -> 3
# step 2: from that result, keep three columns
small   <- select(r1_only, sample, formic, acetic)
small
```

```
      sample   formic   acetic
1 Control_R1 34660.87 2485.879
2       A_R1 29186.73 3587.407
3       B_R1 22333.59 3562.859
```

The result of step 1 is the input table of step 2. Give the variables names that say what they contain, not `step1` and `step2`.

### The pipe (optional, just so you recognise it)

On the internet you will constantly see the symbol `|>` (or the older `%>%`), the "pipe". It chains the same steps without inventing a name for each one: it takes the table on its left and hands it to the next verb as its first argument.

```r
# exactly the same as the two lines above, but in one line
acids |> filter(replicate == 1) |> select(sample, formic, acetic)
```

```
      sample   formic   acetic
1 Control_R1 34660.87 2485.879
2       A_R1 29186.73 3587.407
3       B_R1 22333.59 3562.859
```

You do NOT need the pipe for this course. It is shown once, actually used, so that it does not startle you when you meet it online.

### Exercise 8

Keep the rows of product B and sort them from highest to lowest butyric acid, storing each step in a variable.

<details markdown="1">
<summary>Answer</summary>

```r
b_only   <- filter(acids, product == "B")
b_sorted <- arrange(b_only, desc(butyric))
b_sorted
```

```
  sample product replicate   formic   acetic propionic  lactic  butyric  citric
1   B_R1       B         1 22333.59 3562.859  2431.971 358.035 2383.543 672.428
2   B_R2       B         2 26137.49 3331.052  1889.117 371.603 2255.989 580.663
      scfa
1 4815.514
2 4145.106
```

</details>

### Exercise 8b (optional: with the pipe)

Redo Exercise 8 in a single line using the pipe `|>`, without storing anything in between.

<details markdown="1">
<summary>Answer</summary>

```r
acids |> filter(product == "B") |> arrange(desc(butyric))
```

The result is the same table as in Exercise 8.

</details>

### Exercise 9

- a) Keep only the samples whose butyric acid is above the average butyric acid of all six. Do not type the number: put the calculation inside the condition.
- b) Sort them from highest to lowest butyric acid.

<details markdown="1">
<summary>Answer</summary>

```r
above_mean <- filter(acids, butyric > mean(butyric))
arrange(above_mean, desc(butyric))
```

```
  sample product replicate   formic   acetic propionic   lactic  butyric  citric
1   B_R1       B         1 22333.59 3562.859  2431.971  358.035 2383.543 672.428
2   B_R2       B         2 26137.49 3331.052  1889.117  371.603 2255.989 580.663
3   A_R1       A         1 29186.73 3587.407   407.037 3509.921 1635.671 711.395
      scfa
1 4815.514
2 4145.106
3 2042.708
```

Inside `filter()`, `mean(butyric)` is computed on the whole column first (1349.68, as in the `summarise()` example), then each row is compared with it. If the data change, the threshold changes with them, which would not happen with a typed number.

</details>

### Exercise 10

- a) Using the `scfa` column, keep only the samples whose `scfa` is above 1000.
- b) Count how many samples of each product survived.

<details markdown="1">
<summary>Answer</summary>

```r
high_scfa <- filter(acids, scfa > 1000)
high_scfa_by_product <- group_by(high_scfa, product)
summarise(high_scfa_by_product, n_samples = n())
```

```
# A tibble: 3 × 2
  product n_samples
  <chr>       <int>
1 A               2
2 B               2
3 Control         1
```

One Control sample (Control_R1, with 704) is below 1000. If R says `object 'scfa' not found`, you have not stored the `scfa` column yet: run `acids <- mutate(acids, scfa = butyric + propionic)` first.

</details>

### Exercise 11

Chaining two verbs, storing each step:

- a) keep the rows of replicate 2;
- b) sort them from highest to lowest propionic acid;
- c) save the result in `output/replicate2_propionic.csv`.

<details markdown="1">
<summary>Answer</summary>

```r
r2_only   <- filter(acids, replicate == 2)
r2_sorted <- arrange(r2_only, desc(propionic))
write.csv(r2_sorted, "output/replicate2_propionic.csv", row.names = FALSE)
r2_sorted
```

```
      sample product replicate   formic   acetic propionic   lactic  butyric  citric
1       B_R2       B         2 26137.49 3331.052  1889.117  371.603 2255.989 580.663
2       A_R2       A         2 29401.88 3189.795   633.849 2614.445  839.359 991.664
3 Control_R2 Control         2 35620.77 2367.957   567.952 4379.197  584.875 659.495
      scfa
1 4145.106
2 1473.208
3 1152.827
```

`write.csv()` is the function from [Part 3]({{ site.baseurl }}/modules/r/part-3-our-own-data/#saving-results-the-output-folder): it works the same on a table made with `dplyr`.

</details>

### Exercise 12

- a) Add a column `butyric_pct`: butyric as a percentage of `scfa`, that is `100 * butyric / scfa`.
- b) Group by product and calculate the mean `butyric_pct` of each one.
- c) Sort the result from highest to lowest.
- d) Save it in `output/butyric_pct_per_product.csv`.

<details markdown="1">
<summary>Answer</summary>

```r
acids_pct      <- mutate(acids, butyric_pct = 100 * butyric / scfa)
pct_by_product <- group_by(acids_pct, product)
pct_summary    <- summarise(pct_by_product, mean_butyric_pct = mean(butyric_pct))
pct_sorted     <- arrange(pct_summary, desc(mean_butyric_pct))
write.csv(pct_sorted, "output/butyric_pct_per_product.csv", row.names = FALSE)
pct_sorted
```

```
# A tibble: 3 × 2
  product mean_butyric_pct
  <chr>              <dbl>
1 A                   68.5
2 Control             53.7
3 B                   52.0
```

Four verbs, four steps, each stored under a name that says what it holds. In product A, butyric acid makes up about two thirds of the two "good" acids; in the Control and in B, about half. Check the `output` folder with `list.files("output")`: both files from Exercises 11 and 12 should be there.

</details>

---

## What to take away

| Verb | What it does |
|---|---|
| `filter()` | Picks ROWS (trims vertically) |
| `select()` | Picks COLUMNS (trims horizontally) |
| `arrange()` | Sorts |
| `mutate()` | Adds a column |
| `summarise()` | Reduces |
| `group_by()` | Makes `summarise()` work per group |

Chain steps by storing each result in a variable. You have also met the pipe `|>` once: you do not need it for this course, but now it will not startle you when you see it online.

**Next:** [Part 5: Do the two replicates agree?]({{ site.baseurl }}/modules/r/part-5-do-the-two-replicates-agree/), the second half of the same notebook.
