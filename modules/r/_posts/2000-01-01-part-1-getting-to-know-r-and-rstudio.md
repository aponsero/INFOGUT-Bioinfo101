---
title: "Part 1: Getting to know R and RStudio"
---

# Part 1: Getting to know R and RStudio
{: .no_toc}

<div class="panel" markdown="1">
**Estimated time:** 20 minutes.<br>
**You need:** R and RStudio installed (see [Setup]({{ site.baseurl }}/modules/about/setup/)). There is no code to write on this page: it is a map of the place.<br>
**Authors:** Andrés Méndez Hurtado, Adrián Martín Segura, Alba Pérez Cuervo, Laura Judith Marcos Zambrano, Matti Ruuskanen.
</div>

Nothing here has to be memorised. You will meet all of it again, in practice, in the next parts. Come back to this page as a reference.

**On this page**

* TOC
{:toc}

---

## 1. What is R?

R is a programming language created in the 1990s by statisticians, for people working with data. The way it works is simple: you write an instruction, R runs it, and R gives you back a result: a number, a table or a figure.

<figure>
<img src="{{ site.baseurl }}/modules/r/images/r-workflow.png" alt="You write an instruction, R runs it, and you get back a number, a table or a plot.">
<figcaption>You write an instruction, R runs it, and you get a result back.</figcaption>
</figure>

Three things worth knowing:

- **It is free and open.** No licence, no subscription. It runs on Windows, Mac and Linux. The code is open, so anyone can check exactly what a function does to your data, which matters in research.
- **It is written, not clicked.** There are no menus and no buttons for the analysis. Everything you do is a line of text that you can save, re-read and re-run.
- **The community is enormous.** Over 20,000 extensions exist, and for almost any question you have, somebody has already asked it online.

---

## 2. Why write code instead of clicking?

Clicking through menus is faster the first time. That is true, and your first week in R will be slower than the same work in a spreadsheet. But look at what happens afterwards:

<figure>
<img src="{{ site.baseurl }}/modules/r/images/clicking-vs-script.png" alt="Total time spent against the number of times an analysis is repeated. Clicking costs the same time at every repeat; a script costs more the first time and almost nothing afterwards.">
<figcaption>Each repeat by clicking costs the full time again. A script costs more once, then almost nothing.</figcaption>
</figure>

| | Clicking (spreadsheet, menus) | Writing a script |
|---|---|---|
| First time | Faster | Slower |
| Repeating it | By hand, again | Instant |
| New data arrive | Do everything again | Change one line |
| "How did you do this?" | You have to remember | The script is the answer |
| Can a colleague check it? | No | Yes |

This is what people mean by **reproducible analysis**, and it is the single best reason to learn R.

---

## 3. R and RStudio: two different things

<figure>
<img src="{{ site.baseurl }}/modules/r/images/r-engine-rstudio-dashboard.png" alt="A car: R is the engine that does the actual work; RStudio is the dashboard, where you write, see and control.">
<figcaption>R is the engine; RStudio is the dashboard.</figcaption>
</figure>

| | What it is | If you install only this |
|---|---|---|
| **R** | The engine. The program that actually does the work. | Everything works, but in a bare, uncomfortable window. |
| **RStudio** | The dashboard. A workplace built around R: editor, console, objects, files, plots and help, all in one window. | Nothing works. There is no engine. |

RStudio does not analyse anything by itself. It sends your instructions to R and shows you what comes back. Two practical consequences:

- You install **R first, then RStudio**: RStudio looks for an R installation when it starts.
- After that, you **only ever open RStudio**. R runs quietly underneath.

---

## 4. The RStudio window: four panes

RStudio always has the same four panes, in the same four corners.

When you open RStudio for the first time, **you only see three**. That is normal.

<figure>
<img src="{{ site.baseurl }}/modules/r/images/rstudio-first-open.png" alt="RStudio opened for the first time: the console fills the left side; the environment and files panes are on the right.">
<figcaption>RStudio the first time you open it: the console on the left, two panes on the right.</figcaption>
</figure>

The fourth pane, **Source**, appears when you open or create a file, for example with **File > New File > R Script**.

<figure>
<img src="{{ site.baseurl }}/modules/r/images/rstudio-new-script.png" alt="The File menu of RStudio open at New File, R Script.">
<figcaption>Opening a new file makes the Source pane appear.</figcaption>
</figure>

<figure>
<img src="{{ site.baseurl }}/modules/r/images/rstudio-four-panes.png" alt="RStudio with four numbered panes: 1 Source, top left; 2 Console, bottom left; 3 Environment, top right; 4 Files, Plots, Packages and Help, bottom right.">
<figcaption>The four panes: Source (1), Console (2), Environment (3), and Files, Plots, Packages and Help (4).</figcaption>
</figure>

You can resize the panes, maximise one or close one. If your screen looks different from someone else's, a pane is probably just closed, not broken.

### Source (top left): where you write

<figure>
<img src="{{ site.baseurl }}/modules/r/images/pane-source.png" alt="The Source pane with a short script.">
</figure>

- This is where your file is: a script, or the notebooks used in this course. It is saved on your computer, and it is the thing you keep at the end.
- Writing a line here does nothing by itself. Pressing Enter only starts a new line, as in a word processor.
- To run a line, press **Ctrl + Enter** (**Cmd + Enter** on Mac). The line is sent to the console and R runs it.
- You will spend about 90% of your time in this pane.

### Console (bottom left): where R answers

<figure>
<img src="{{ site.baseurl }}/modules/r/images/pane-console.png" alt="The Console pane showing a few commands and R's answers.">
</figure>

- This is R itself. Instructions arrive here and results appear here.
- `>` at the start of the line means R is ready and waiting.
- `+` instead means R thinks your instruction is unfinished, usually because of a missing bracket. Press **Esc** to get out.
- You can type directly into the console, but nothing you type here is saved. Rule of thumb: try things in the console, keep the good ones in your file.
- Red text is normal. Some of it is errors, some of it is just warnings or messages. Read it before panicking.

### Environment (top right): what R is holding

<figure>
<img src="{{ site.baseurl }}/modules/r/images/pane-environment.png" style="max-width:520px" alt="The Environment pane listing four values.">
</figure>

- The list of everything you have created in this session: values, vectors, tables.
- It lives in memory, not on disk. Closing R empties it.
- When R says an object does not exist, look here first. Usually it has not been created yet, or the name is spelled differently. R is case sensitive: `weight` and `Weight` are two different things.
- Clicking a table here opens it in a spreadsheet-like viewer.
- The broom icon clears everything: a good way to check that your code really works from zero.

### Files, Plots, Packages, Help (bottom right)

<figure>
<img src="{{ site.baseurl }}/modules/r/images/pane-files.png" style="max-width:600px" alt="The Files tab showing a course folder with input and output folders and several R scripts.">
</figure>

| Tab | What it is for |
|---|---|
| Files | The folder you are working in, like your file explorer. Useful early on: it shows where R actually is. |
| Plots | Every figure you draw appears here, with arrows to go back and a button to export. |
| Packages | The extensions installed on this computer, and which ones are loaded. |
| Help | The manual page of any function: the official documentation. |
| Viewer | Notebooks and interactive output. |

---

## 5. Scripts and notebooks

There are two common ways of writing R code in RStudio.

<figure>
<img src="{{ site.baseurl }}/modules/r/images/script-vs-notebook.png" alt="Left: an R script, with only code and comments, and results in the console. Right: an R Markdown notebook, with text, code and results together, which can be exported as a report.">
<figcaption>An R script (left) and an R Markdown notebook (right).</figcaption>
</figure>

| | R script (`.R`) | R notebook (`.Rmd`) |
|---|---|---|
| Contains | Only code, plus comments after `#` | Written text and code, in blocks called **chunks** |
| Where results appear | Console and Plots pane | Immediately below each chunk |
| Exports to | Nothing: the file is the code itself | HTML, Word or PDF |
| Best for | A pipeline you run from top to bottom | Explaining results to somebody else |

It is the same R underneath. The only difference is how code and results are laid out on the page. You may also see files ending in `.qmd`: that is Quarto, a newer version of the same idea.

### How to work with the notebooks of this course

Each hands-on part of this module comes with a notebook: the explanations, the example code and empty chunks for your answers.

1. Download the zip file from the page and unzip it. Keep the folder as it is: the notebook sits next to an `input` folder (the data) and an `output` folder (for your results).
2. In RStudio, open the notebook with **File > Open File**.
3. Code chunks are the grey blocks starting with ```` ```{r} ````. To run a whole chunk, click the green triangle at its top right. To run one line, put the cursor on it and press **Ctrl + Enter** (**Cmd + Enter** on Mac).
4. The result appears just below the chunk.
5. Where a chunk is empty, write your own answer in it and run it.
6. Save the notebook (**Ctrl + S**, **Cmd + S** on Mac): it becomes your notes.

If RStudio shows a message offering to install packages it needs for notebooks (such as `rmarkdown`), accept it.

---

## 6. Packages: extending what R can do

R comes with a large set of functions built in: `mean()`, `plot()`, `read.csv()` and hundreds more. That is called **base R**.

Everything else lives in **packages**: collections of functions written by other people and shared freely. There are more than 20,000, including `dplyr` for handling tables, `ggplot2` for figures, and hundreds of specialised ones for microbiome and metabolomics data.

Working with a package has two separate steps:

| | When | What it does |
|---|---|---|
| Install | Once per computer | Downloads the package onto your machine. Never repeated, except to update. |
| Load | Once per session | Tells R which of the installed packages you want available right now. |

Installing is buying the book and putting it on your shelf. Loading is taking it down and opening it.

```r
install.packages("ggplot2")   # ONCE per computer. In quotes.
library(ggplot2)              # EVERY session. No quotes.
```

At installation time the package is just a name, a piece of text, because R does not know it yet: hence the quotes. Once installed, R knows it as a thing, so no quotes are needed.

<figure>
<img src="{{ site.baseurl }}/modules/r/images/cran-bioconductor.png" alt="Two repositories send packages to your computer: CRAN, the official R repository for general purpose packages, and Bioconductor, the repository for biology and omics data.">
<figcaption>Packages come from two main repositories: CRAN and Bioconductor.</figcaption>
</figure>

Three practical notes:

- **Repositories.** CRAN is the official one. Bioconductor is the repository for biology and omics data, and it has its own installer (`BiocManager`).
- **Loading order matters.** Two packages can contain a function with the same name. The one loaded last wins: a classic source of code that mysteriously stops working.
- **Good habit.** Put all your `library()` calls together at the top of your file, so anyone opening it can see immediately what is needed.

The error `could not find function` almost always means: the package is installed, but you forgot to load it.

---

## 7. What a working session looks like

<figure>
<img src="{{ site.baseurl }}/modules/r/images/working-session.png" alt="Six steps: open RStudio, open or create your file, load your packages, read your data, run line by line, save.">
<figcaption>A working session, in six steps.</figcaption>
</figure>

1. Open RStudio.
2. Open your file, or create a new one, and save it straight away with a sensible name.
3. Run the `library()` lines at the top.
4. Read your data, and watch it appear in the Environment pane.
5. Work line by line or chunk by chunk, looking at the results as you go.
6. At the end, save the file.

**Save the code, not the results.** Results are cheap: you can always regenerate them by running the code again. Code you never wrote down is gone forever.

---

## 8. Quick reference

### Shortcuts

| Action | Windows / Linux | Mac |
|---|---|---|
| Run the current line | Ctrl + Enter | Cmd + Enter |
| Run the current chunk (notebooks) | Ctrl + Shift + Enter | Cmd + Shift + Enter |
| Interrupt, or get out of `+` | Esc | Esc |
| New R script | Ctrl + Shift + N | Cmd + Shift + N |
| Save the file | Ctrl + S | Cmd + S |
| Type the arrow `<-` | Alt + minus | Option + minus |

### Console symbols

| Symbol | Meaning |
|---|---|
| `>` | R is ready and waiting for you |
| `+` | Your instruction is unfinished, usually a missing bracket. Press Esc |

### Common first-week errors

| Message | What it usually means |
|---|---|
| `object 'x' not found` | Not created yet, or the name is spelled differently. Check the Environment. R is case sensitive. |
| `could not find function "..."` | The package is installed but not loaded. Run `library(...)`. |
| `unexpected symbol`, `unexpected ')'` | A typo: a missing comma, quote or bracket. |
| Red text that is not an error | Warnings and messages are also red. Read before panicking. |

### Glossary

| Term | Meaning |
|---|---|
| R | The programming language: the engine that does the work |
| RStudio | The interface you open to work with R |
| Script | A plain text file (`.R`) containing your instructions |
| Notebook | A file (`.Rmd`) mixing text and code chunks, with results shown below each chunk |
| Chunk | A block of code inside a notebook |
| Console | The pane where R receives instructions and returns results |
| Environment | The list of objects R is holding in memory right now |
| Object, variable | Something you created and gave a name to |
| Function | An instruction that does something, for example `mean()` or `plot()` |
| Package | A collection of functions written by others and shared |
| Base R | The functions that come with R, without any package |
| CRAN | The official package repository |
| Bioconductor | The package repository for biology and omics data |
| Reproducible | Your analysis can be re-run and re-checked by others, exactly |

**Next:** [Part 2: First steps in R]({{ site.baseurl }}/modules/r/part-2-first-steps-in-r/), where you open RStudio and start writing.
