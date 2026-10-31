---
title: "Part 2: First steps in R"
---

# Part 2: First steps in R
{: .no_toc}

<div class="panel" markdown="1">
**Estimated time:** 45 to 60 minutes.<br>
**You need:** R and RStudio (no extra packages), and the session 1 notebook and data: [download r-session1.zip]({{ site.baseurl }}/modules/r/data/r-session1.zip). The same download is used in [Part 3]({{ site.baseurl }}/modules/r/part-3-our-own-data/).<br>
**Before you start:** [Part 1]({{ site.baseurl }}/modules/r/part-1-getting-to-know-r-and-rstudio/) explains the RStudio window and how to work with a notebook.<br>
**Authors:** Andrés Méndez Hurtado, Adrián Martín Segura, Alba Pérez Cuervo, Laura Judith Marcos Zambrano, Matti Ruuskanen.
</div>

**On this page**

* TOC
{:toc}

---

## How to use this page

1. [Download r-session1.zip]({{ site.baseurl }}/modules/r/data/r-session1.zip) and unzip it. You get a folder called `r-session1` containing the notebook `R_course_session1_STUDENT.Rmd`, an `input` folder with four data files, and an empty `output` folder.
2. Open `R_course_session1_STUDENT.Rmd` in RStudio (**File > Open File**).
3. Work through the notebook and this page side by side. This page covers sections 0 to 9 of the notebook; [Part 3]({{ site.baseurl }}/modules/r/part-3-our-own-data/) covers the rest.

Grey boxes with code are what you run. The box just below shows what R answers. Anything after a `#` is a comment: R ignores it, it is a note for humans.

Nothing is learned by watching: **type and run every line**. Where the notebook says `# > your answer:`, the code is yours to write. Try each exercise before opening the answer.

---

## 0. The working directory

R is always "standing" in some folder. When you ask it for a file, it looks there. This is the number one problem for beginners.

```r
getwd()          # which folder am I in?
list.files()     # what is in this folder?
```

When you run these chunks in the notebook, R answers with the path to your `r-session1` folder, and lists what is in it:

```
[1] "R_course_session1_STUDENT.Rmd" "input"                        
[3] "output"                       
```

In a notebook, RStudio runs the chunks from the folder where the notebook is saved. That is why the notebook must stay next to its `input` and `output` folders.

When you work with a plain script instead, set the folder yourself, once, at the start: save the script in your working folder, then use the menu **Session > Set Working Directory > To Source File Location**. You can also change folder by hand, replacing the path with one on your computer:

```r
setwd("C:/Users/your_user/.../r_course")
```

**Careful on Windows:** copying a path gives you backslashes, which R does not accept as they are.

| Result | Path |
|---|---|
| Wrong | `"C:\Users\alba\r_course"` |
| Right | `"C:/Users/alba/r_course"` |
| Right | `"C:\\Users\\alba\\r_course"` |

### How the folder is organised

One working folder. The notebook (or script) sits loose in it, next to two subfolders:

| Item | Contains |
|---|---|
| `R_course_session1_STUDENT.Rmd` | The recipe of the analysis |
| `input/` | The raw data as it arrived. Read-only: never modified |
| `output/` | Everything R produces: tables, figures, results |

If something goes wrong, you can always start again from the original data.

---

## 1. R is a calculator

```r
2 + 2
10 / 4
3 ^ 2          # power
sqrt(16)       # square root
```

```
[1] 4
[1] 2.5
[1] 9
[1] 4
```

The `[1]` at the start of each answer is not part of the result: it means "this line starts with the first value". It becomes useful when R prints long series of values.

| Operator or function | Meaning |
|---|---|
| `+` `-` `*` `/` | Addition, subtraction, multiplication, division |
| `^` | Power |
| `%%` | Remainder of a division |
| `sqrt()` | Square root |
| `round()` | Rounding |

R follows the usual order of operations. Use brackets whenever you are not sure.

---

## 2. Storing things in variables

The arrow `<-` means "store this here". Keyboard shortcut: **Alt + minus** (**Option + minus** on Mac).

```r
weight <- 70
height <- 1.75
weight          # typing the name shows its content
```

```
[1] 70
```

Storing a value prints nothing. The variables appear in the **Environment** pane, and you can use their names anywhere a number would go: that is the whole point.

- Names are case sensitive: `Weight` and `weight` are different.
- Use short names, without spaces or accents, and do not start them with a number.
- Assigning a new value to an existing name simply overwrites the old content.

**Mini exercise.** Compute the body mass index (weight divided by height squared) and store it in a variable called `bmi`. You need the `^` operator and `<-`.

<details markdown="1">
<summary>Answer</summary>

```r
bmi <- weight / height ^ 2
bmi
```

```
[1] 22.85714
```

</details>

---

## 3. Vectors: several values in a single variable

A vector is an ordered series of values of the **same type**, built with `c()` (for "combine").

```r
numbers <- c(1, 2, 3, 4, 5)
numbers
```

```
[1] 1 2 3 4 5
```

```r
names_list <- c("Ana", "Luis", "Marta")
names_list
```

```
[1] "Ana"   "Luis"  "Marta"
```

A very handy shortcut: a colon generates a sequence.

```r
1:5
10:15
```

```
[1] 1 2 3 4 5
[1] 10 11 12 13 14 15
```

How many elements does a vector have?

```r
length(numbers)
```

```
[1] 5
```

Every column of a table is, underneath, a vector. Learn vectors and you have learned half of R.

---

## 4. Positions: pulling out specific elements

Square brackets `[ ]` mean "give me this part of the object". **In R, counting starts at 1, not at 0.**

```r
numbers <- 1:5
numbers[1]        # the first one
numbers[3]        # the third one
numbers[2:4]      # from the second to the fourth
numbers[c(1, 5)]  # the first and the fifth
numbers[-1]       # ALL BUT the first
```

```
[1] 1
[1] 3
[1] 2 3 4
[1] 1 5
[1] 2 3 4 5
```

A minus sign removes instead of selecting. You can also select by condition:

```r
numbers[numbers > 3]
```

```
[1] 4 5
```

Selecting by condition is what filtering a table really is. You will use the same logic later to keep only the rows you are interested in.

---

## 5. Operations on the whole vector at once

This is one of the great advantages of R. An operation applied to a vector is applied to every element: you almost never need to repeat it by hand.

```r
numbers
numbers + 5        # adds 5 to EACH value
numbers * 2
```

```
[1] 1 2 3 4 5
[1]  6  7  8  9 10
[1]  2  4  6  8 10
```

**Careful:** this changes nothing unless you store it. `numbers + 5` only prints the result; `numbers <- numbers + 5` replaces the content of `numbers`.

### Summary functions

```r
mean(numbers)      # mean
sd(numbers)        # standard deviation
min(numbers)
max(numbers)
sum(numbers)
round(mean(numbers), 1)   # round to 1 decimal
```

```
[1] 3
[1] 1.581139
[1] 1
[1] 5
[1] 15
[1] 3
```

In `round(mean(numbers), 1)`, one function sits inside another: the inside one runs first, and its result goes to the outside one.

```r
summary(numbers)   # the whole summary in one go
```

```
   Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
      1       2       3       3       4       5 
```

`summary()` is the fastest way to see whether a variable looks reasonable. Other useful ones: `median()`, and `length()`, which you have already met.

### Exercise 1

Create a vector called `marks` with the values 7, 8, 5, 9, 6. Then compute its mean rounded to 2 decimals, and its standard deviation.

You need: `c()`, `mean()`, `round()`, `sd()`.

<details markdown="1">
<summary>Answer</summary>

```r
marks <- c(7, 8, 5, 9, 6)
round(mean(marks), 2)
sd(marks)
```

```
[1] 7
[1] 1.581139
```

The mean is exactly 7, so rounding to 2 decimals shows `7`, not `7.00`.

</details>

---

## 6. Data types: the class() function

Knowing the type of a value, or of a column, prevents most of the errors you will meet.

| Type | What it holds | Examples |
|---|---|---|
| numeric | Numbers, with or without decimals | `70`, `1.75`, `-3` |
| character | Text, always in quotes | `"Ana"`, `"control"` |
| logical | Only two values, no quotes | `TRUE`, `FALSE` |

```r
class(numbers)
class(names_list)
class(TRUE)
```

```
[1] "integer"
[1] "character"
[1] "logical"
```

R says `"integer"` for `numbers`, not `"numeric"`: in section 4, `numbers` was created with `1:5`, and the colon creates whole numbers, which R stores as integers. Integers are a kind of number, and everything you do with numbers works the same on them. A vector created with `c(1, 2, 3)` would give `"numeric"`.

### Comparisons: questions with a yes/no answer

A comparison does not give you a number: it gives `TRUE` or `FALSE` for every element.

```r
numbers > 3         # gives TRUE / FALSE for each element
sum(numbers > 3)    # TRUE counts as 1: how many are above 3
```

```
[1] FALSE FALSE FALSE  TRUE  TRUE
[1] 2
```

| Operator | Meaning |
|---|---|
| `>` `<` | Greater than, smaller than |
| `>=` `<=` | Greater or equal, smaller or equal |
| `==` | Is equal to (double equals) |
| `!=` | Is different from |

The classic beginner error: `=` assigns a value, `==` compares. To ask a question, you always need two.

### A typical mistake: mixing text and numbers

If you mix text and numbers in one vector, **everything** becomes text:

```r
mixed <- c(1, 2, "three")
mixed
class(mixed)
```

```
[1] "1"     "2"     "three"
[1] "character"
```

The quotes around `"1"` and `"2"` show that they are no longer numbers. You cannot compute a mean on them any more.

---

## 7. Counting categories: the table() function

For text or group variables, a mean makes no sense. What you want is a count.

```r
sex   <- c("M", "F", "F", "M", "F")
group <- c("control", "treated", "control", "treated", "control")

table(sex)
table(group)
```

```
sex
F M 
3 2 
group
control treated 
      3       2 
```

`table()` also cross-tabulates two variables: it counts every combination.

```r
table(sex, group)
```

```
   group
sex control treated
  F       2       1
  M       1       1
```

You will use this constantly to check that the experimental design is what you think it is.

### Exercise 2

Create a vector `days` with `"monday"`, `"tuesday"`, `"monday"`, `"monday"`, `"tuesday"`, `"wednesday"`, and count how many times each day appears.

You need: `c()`, `table()`.

<details markdown="1">
<summary>Answer</summary>

```r
days <- c("monday", "tuesday", "monday", "monday", "tuesday", "wednesday")
table(days)
```

```
days
   monday   tuesday wednesday 
        3         2         1 
```

</details>

---

## 8. Data frames: the table of R

This is what you will use from now on. A data frame is a set of vectors placed side by side as columns. Each column has **one** type, and all of them must have the **same** length. Rows are the observations (your samples); columns are what you measured on them (your variables).

```r
patients <- data.frame(
  name  = c("Ana", "Luis", "Marta", "Jorge"),
  age   = c(34, 45, 29, 52),
  group = c("control", "treated", "control", "treated")
)
patients
```

```
   name age   group
1   Ana  34 control
2  Luis  45 treated
3 Marta  29 control
4 Jorge  52 treated
```

Every table you read from a CSV file arrives as a data frame.

### Exploring a data frame: always the first thing to do

```r
nrow(patients)     # number of ROWS    (= samples)
ncol(patients)     # number of COLUMNS (= variables)
dim(patients)      # rows and columns in one go
names(patients)    # the column names
```

```
[1] 4
[1] 3
[1] 4 3
[1] "name"  "age"   "group"
```

```r
str(patients)      # structure: the type of each column
```

```
'data.frame':	4 obs. of  3 variables:
 $ name : chr  "Ana" "Luis" "Marta" "Jorge"
 $ age  : num  34 45 29 52
 $ group: chr  "control" "treated" "control" "treated"
```

`str()` is the most useful function of the day: it tells you whether your numbers really arrived as numbers (`num`) or as text (`chr`).

```r
head(patients, 2)  # the first 2 rows
tail(patients, 2)  # the last 2 rows
```

```
  name age   group
1  Ana  34 control
2 Luis  45 treated
   name age   group
3 Marta  29 control
4 Jorge  52 treated
```

```r
summary(patients)  # a summary of each column
```

```
     name                age           group          
 Length:4           Min.   :29.00   Length:4          
 Class :character   1st Qu.:32.75   Class :character  
 Mode  :character   Median :39.50   Mode  :character  
                    Mean   :40.00                     
                    3rd Qu.:46.75                     
                    Max.   :52.00                     
```

With a big table, never type its name alone: printing hundreds of rows floods the console. Use `head()`, `dim()` and `str()` instead.

### Accessing a column: the $ symbol

```r
patients$age
mean(patients$age)
table(patients$group)
```

```
[1] 34 45 29 52
[1] 40

control treated 
      2       2 
```

A column pulled out with `$` is just a vector, so everything you learned above works on it.

### Accessing by position: [row, column]

```r
patients[1, ]        # the whole row 1
patients[, 2]        # the whole column 2
patients[1, 2]       # row 1, column 2
patients[1:2, ]      # the first two rows
```

```
  name age   group
1  Ana  34 control
[1] 34 45 29 52
[1] 34
  name age   group
1  Ana  34 control
2 Luis  45 treated
```

Leaving one side of the comma empty means "all of them": all the columns in `patients[1, ]`, all the rows in `patients[, 2]`.

### Filtering rows by a condition

```r
patients[patients$age > 40, ]
patients[patients$group == "control", ]
```

```
   name age   group
2  Luis  45 treated
4 Jorge  52 treated
   name age   group
1   Ana  34 control
3 Marta  29 control
```

R keeps the rows where the condition is `TRUE`. Note the comma before the closing bracket: the condition chooses rows, and the empty space after the comma keeps all the columns.

**Careful:** to compare you use a double equals sign `==`. A single `=` means something else.

### Creating a new column

```r
patients$age_months <- patients$age * 12
patients
```

```
   name age   group age_months
1   Ana  34 control        408
2  Luis  45 treated        540
3 Marta  29 control        348
4 Jorge  52 treated        624
```

### Exercise 3

Using the data frame `patients`:

- a) how many rows and columns does it have?
- b) what is the mean age?
- c) keep only the rows of the `"treated"` group.

You need: `dim()`, `mean()`, `$`, `[ , ]`, `==`.

<details markdown="1">
<summary>Answer</summary>

```r
dim(patients)
mean(patients$age)
patients[patients$group == "treated", ]
```

```
[1] 4 4
[1] 40
   name age   group age_months
2  Luis  45 treated        540
4 Jorge  52 treated        624
```

`patients` now has 4 columns, because you added `age_months` just before. If you run the exercise before creating that column, you get `4 3`.

</details>

### Exercise 4 (recap)

Create a data frame `plants` with:

| pot | water | height |
|---|---|---|
| 1 | low | 12 |
| 2 | low | 15 |
| 3 | low | 11 |
| 4 | high | 22 |
| 5 | high | 25 |
| 6 | high | 20 |

Then:

- a) look at its structure;
- b) count how many pots there are of each watering level;
- c) compute the mean height of the pots with `"high"` watering.

You need: `data.frame()`, `str()`, `table()`, `mean()`, `$`, `[ ]`.

<details markdown="1">
<summary>Answer</summary>

```r
plants <- data.frame(
  pot    = c(1, 2, 3, 4, 5, 6),
  water  = c("low", "low", "low", "high", "high", "high"),
  height = c(12, 15, 11, 22, 25, 20)
)

str(plants)
table(plants$water)

high_pots <- plants[plants$water == "high", ]
mean(high_pots$height)
```

```
'data.frame':	6 obs. of  3 variables:
 $ pot   : num  1 2 3 4 5 6
 $ water : chr  "low" "low" "low" "high" ...
 $ height: num  12 15 11 22 25 20

high  low 
   3    3 
[1] 22.33333
```

For c), the first line keeps the rows with high watering and stores them; the second takes the mean of their `height` column.

</details>

---

## 9. Asking for help

```r
?mean            # opens the help page of a function
help(round)      # exactly the same thing, written as a function
```

The help page opens in the **Help** pane. It tells you what the function does, which arguments it takes, and gives examples at the bottom.

If you do not even know the name of the function, search the internet for "how to ... in R". That is what everybody does, always.

### Reading an error message is a skill

R tells you which object or function failed.

| Message | What it usually means |
|---|---|
| `object 'x' not found` | A typo, or something you never created |
| `could not find function "..."` | A package is missing or not loaded |

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

**Next:** [Part 3: Our own data]({{ site.baseurl }}/modules/r/part-3-our-own-data/), with the second half of the same notebook.
