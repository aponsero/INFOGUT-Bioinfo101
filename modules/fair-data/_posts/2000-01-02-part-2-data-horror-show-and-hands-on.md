---
title: "Part 2: Data horror-show and hands-on"
---

# Part 2: Data horror-show and hands-on
{: .no_toc}

<div class="panel" markdown="1">
**Estimated time:** 20 minutes.<br>
**You need:** a project folder to clean up: the folder you worked in during the [R and RStudio module]({{ site.baseurl }}/modules/r/programme/), or any small project folder of your own.<br>
**Before you start:** this page builds on [Part 1]({{ site.baseurl }}/modules/fair-data/part-1-data-life-cycle-and-fair/), in particular the four FAIR questions.<br>
**Author:** Alise Ponsero, Quadram Institute Bioscience, Norwich, UK.
</div>

**On this page**

* TOC
{:toc}

---

## 1. What is wrong here?

Six exhibits, all of them common, all of them made with good intentions. For each one, decide what the problem is before opening the answer.

### Exhibit 1: `scripts/` folder contents

```
analysis_final.R
analysis_final_v2.R
analysis_final_USE_THIS.R
analyse2_old.R
analysis_final_v2 (1).R
```

<details markdown="1">
<summary>What is wrong</summary>

Nobody, including the author six months later, can tell which script produced the published results. "final" is never final, `USE_THIS` only works until the next version, and `(1)` is a copy made by a download or a sync tool, which may or may not differ from the original.

**Better:** one script per analysis step, with a name that says what it does (for example `01_import_data.R`, `02_diversity.R`). Keep the history of changes with version control rather than in the filename (see the [Reproducible research module]({{ site.baseurl }}/modules/reproducible-research/programme/)), and delete or archive the versions that are no longer used.
</details>

### Exhibit 2: `project/` folder contents

```
Control_R1.csv
script.R
plot1.png
notes.txt
data2.csv
final_table.csv
... 34 more files
```

<details markdown="1">
<summary>What is wrong</summary>

Raw data, scripts, figures, notes and derived tables are all mixed in one flat folder. You cannot tell which files are original measurements and which were produced by a script, so you cannot tell what is safe to delete or regenerate. Names such as `data2.csv`, `plot1.png` and `script.R` describe nothing.

**Better:** a folder layout that separates raw data, processed data, scripts and results (section 3), and names that say what each file contains.
</details>

### Exhibit 3: `metadata.xlsx` spreadsheet

<table>
  <thead><tr><th></th><th>A</th><th>B</th><th>C</th></tr></thead>
  <tbody>
    <tr><td>1</td><td>Microbial Ecology Lab</td><td></td><td></td></tr>
    <tr><td>2</td><td></td><td></td><td></td></tr>
    <tr><td>3</td><td>Experiment 4 FINAL</td><td></td><td></td></tr>
    <tr><td>4</td><td></td><td></td><td></td></tr>
    <tr><td>5</td><td>Run by JD, Mar 2024</td><td></td><td></td></tr>
    <tr><td>6</td><td></td><td></td><td></td></tr>
    <tr><td>7</td><td>Sample</td><td>OD</td><td>pH</td></tr>
    <tr><td>8</td><td>Control R1</td><td>0.42</td><td>6.8</td></tr>
  </tbody>
</table>

<details markdown="1">
<summary>What is wrong</summary>

The spreadsheet is laid out like a document, not a table. The data only start at row 7, so any software reading the file will take "Microbial Ecology Lab" as the column header and the title lines as data. Useful information (which experiment, who ran it, when) sits in free-text cells that no script can use. "FINAL" again says nothing. The sample ID contains a space, "OD" does not say at which wavelength, and the `.xlsx` format needs specific software to open.

**Better:** a plain CSV where row 1 is the column names and every following row is one sample. Experiment-level information goes in the README, or in columns if it varies between samples (for example `operator`, `run_date`). Use explicit column names such as `od600`.
</details>

### Exhibit 4: `README.md` file contents

```
# Project

scripts for the paper
```

<details markdown="1">
<summary>What is wrong</summary>

A README is the first file a stranger opens, and this one answers none of their questions. Which project? Which paper? Where do the data come from? What does each folder contain? How do you run the analysis? Who do you ask?

**Better:** five headings, a few lines each: what this is, where the data came from, what each folder holds, how to run it, and who to ask (section 4).
</details>

### Exhibit 5: `metadata.csv` table

| sample_id | od600 |
|---|---|
| S1 | 0.42 |
| S2 | 0.51 |
| tube 4 (new) | 0.38 |
| tube 4 (new) redo | 0.39 |
| CTRL old | 0.44 |
| ctrl_2 | 0.45 |

<details markdown="1">
<summary>What is wrong</summary>

The sample IDs follow at least four different conventions, and some of them carry notes ("new", "redo", "old") that belong somewhere else. Was the redo meant to replace the first measurement, or are both valid? Is `CTRL old` the same kind of sample as `ctrl_2`? IDs like these cannot be matched reliably to sequencing files or other tables, and cannot be filtered by a script. Spaces and brackets also make them awkward to use at the command line.

**Better:** one ID convention for every sample, decided before the experiment (for example `Control_R1`, `A_R1`). IDs are unique and never change. Anything else (a redo, a replacement, a problem with the tube) goes in its own column, such as `qc_flag` or `notes`.
</details>

### Exhibit 6: `metadata.csv` table

| sample_id | date_collected |
|---|---|
| Control_R1 | 01/02/2024 |
| Control_R2 | 02/01/2024 |
| A_R1 | Feb 1 2024 |
| A_R2 | 1 Feb 24 |
| B_R1 | 2024-02-01 |
| B_R2 | 44958 |

<details markdown="1">
<summary>What is wrong</summary>

The same kind of information is written in six different formats in one column. `01/02/2024` and `02/01/2024` could each be 1 February or 2 January depending on who typed them. `44958` is how Excel stores a date internally (a number of days), which appears when a date cell is converted to a number; converted back, it gives 1 February 2023, so the year may be wrong too. A script cannot sort, filter or compare these values without guessing.

**Better:** one format for every date: the ISO 8601 format `YYYY-MM-DD` (for example `2024-02-01`), which is unambiguous and sorts correctly. Check that your spreadsheet software has not converted it when you save.
</details>

### The same problem, six times

Every one of those exhibits is information that exists in exactly one person's head.

**Nobody meant to.** Each of those was the fastest thing to do on the day it happened. None of them was a mistake at the time, which is exactly why they are so common.

**You cannot see it from inside.** While you still remember, every one of those files is perfectly clear. The problem only appears when the memory fades, or when somebody else opens the folder.

**Fix it while you remember.** Not at submission, not when a reviewer asks, not when you hand over. Now, while you can still say what every file is and why it is there.

---

## 2. Why a metadata file, when the filename already says it?

`Control_R1` already tells you it is a control and a first replicate. So why write it down again?

**Two different readers.** A filename is readable by a human who was there. A metadata file is readable by a script and by a stranger. Filtering on a column works in any language. Parsing a filename works until somebody adds `Control_R1_repeat`.

And no filename will ever tell a reader what treatments A and B actually were.

An example `data/metadata.csv`, with six rows:

```
sample_id,condition,replicate,replicate_type,timepoint,file
Control_R1,control,R1,technical,endpoint,...
A_R1,A,R1,technical,endpoint,...
...
```

**Try it:** what other columns would this file need if these were your own experiments? (Part 1, section 6 lists what a colon model needs.)

---

## 3. Raw data is read-only

One habit, one folder layout, and everything downstream stays recoverable.

Find a logical layout that works, for example:

```
project/
    README.md
    data/
        raw/
        processed/
    scripts/
    results/
```

**Three rules:**

1. Nothing in `data/raw` is ever edited. You read from it, you never write to it. If it needs fixing, the fix is a script that writes somewhere else.
2. Everything in `results/` can be deleted and regenerated by running what is in `scripts/`. If that is not true, something is only in your head.
3. If you cannot tell which files are raw and which are derived, then all of them are effectively raw and none of them are reproducible.

---

## 4. Your turn: clean up your project folder

Use the folder you worked in during the [R and RStudio module]({{ site.baseurl }}/modules/r/programme/), or any small project folder of your own.

**Before you start, score your folder** against the four FAIR questions from Part 1: Yes, Partly or No for each. Nobody has to see your first score.

| Question | Before | After |
|---|---|---|
| **Findable:** could someone find it, and does it say what it is? | | |
| **Accessible:** could someone get it, and are the conditions stated? | | |
| **Interoperable:** could their tools read it (open formats, explicit units)? | | |
| **Reusable:** do they know enough to reuse it, and are they allowed to? | | |

Then work through these four steps:

| Step | What to do |
|---|---|
| **Restructure** | Move what you have into `data/raw`, `data/processed`, `scripts` and `results`. Nothing in `data/raw` gets edited from here on. |
| **Rename** | One convention, applied to every file. No spaces, no slashes, no accents. |
| **Write `data/metadata.csv`** | One row per sample, one column per variable. Explicit units, one agreed code for missing values, no colour and no free text in a numeric column. |
| **Write `README.md`** | What this is. Where the data came from. What each folder holds. How to run it. Who to ask. Five headings, a few lines each. |

Moving files will break the paths in your scripts: update them as you go, then re-run the scripts to check that everything in `results/` can still be regenerated.

**When you finish, score your folder again.**

Perfect is not the target. Better than it was an hour ago is the target.

---

## 5. Two things to take away

**FAIR is a direction, not a pass mark.** A folder that is better than it was this morning is a success.

**The only test that matters:** could a competent stranger, holding only your files, carry on? Everything in this module was a way of answering yes.

You now have a folder worth sharing: structured, named, described and documented. The [Reproducible research module]({{ site.baseurl }}/modules/reproducible-research/programme/) shows how to put it on GitHub.

---

## 6. Where to look next

| Resource | What it is |
|---|---|
| [GO FAIR](https://go-fair.org/fair-principles) | The fifteen principles sitting behind the four questions, each one explained in a paragraph. |
| [FAIRsharing](https://fairsharing.org) | A curated registry of standards, databases and funder or journal data policies. Search it before you invent a format of your own. |
| [MIxS](https://gensc.org/mixs) | The Genomic Standards Consortium checklists and environmental packages for sequence metadata. |
| [STREAMS](https://doi.org/10.1038/s41564-025-02186-2) | The 67-item reporting checklist for environmental, synthetic and non-human host-associated microbiome studies. |

And the research data team at your own institution, or attached to your university library.
