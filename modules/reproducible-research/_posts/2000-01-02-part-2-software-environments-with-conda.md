---
title: "Part 2: Software environments with Conda"
---

# Part 2: Software environments with Conda
{: .no_toc}

<div class="panel" markdown="1">
**Format:** lecture notes, nothing to install.<br>
**Estimated time:** 20 minutes.<br>
**Author:** Petra Polakovičová, Institute for Clinical and Experimental Medicine, Czech Republic.<br>
**Before you start:** this page uses the command line vocabulary from the [Command line module]({{ site.baseurl }}/modules/command-line/programme/).
</div>

**On this page**

* TOC
{:toc}

---

## 1. The big picture: two tools, two halves of one problem

Analysing sequencing data means running many tools, one after the other, on many samples. Two kinds of problems come up again and again, and each has its own tool:

| | Conda | Nextflow |
|---|---|---|
| **Manages** | Environments | Workflows |
| **Question it answers** | "How do I get the exact right software installed, without it fighting other software?" | "How do I chain many steps into an automated pipeline that runs anywhere, at any scale?" |
| **What it gives you** | Isolated collections of tools and versions; solves "dependency hell"; fixes "works on my machine" | Orchestrates the steps and the data flow between them; parallelism and crash-safe restart; portable from laptop to HPC to cloud |

This page covers Conda. [Part 3]({{ site.baseurl }}/modules/reproducible-research/part-3-workflows-with-nextflow/) covers Nextflow.

### The three hells of a bioinformatician

**1. Dependency hell.** Tool A needs version 1.19 of a Python library called numpy. Tool B needs version 1.26. Install both on the same computer and one of them breaks. Now imagine this multiplied across the dozens of tools in a typical analysis, each with its own list of libraries it depends on.

**2. "Works on my machine" hell.** The analysis runs on your laptop, then fails on a colleague's computer or on the cluster, because they have different versions of the tools installed, or are missing one. Results that can't be reproduced aren't science.

**3. Scaling hell.** A real analysis is a chain of steps (download, QC, trimming, denoising, taxonomic assignment, summary), run on hundreds of samples. The process crashes halfway through the night. A Bash script written for five samples doesn't cut it.

**Conda fixes hells 1 and 2. Nextflow fixes hell 3** (and, combined with Conda or containers, hell 2 as well).

---

## 2. What Conda actually is

Conda is two tools in one:

1. **A package manager.** It installs, removes and updates software written in any language: Python, R, Java, C libraries, command-line bioinformatics tools, GPU libraries (CUDA)...
2. **An environment manager.** It creates separate, isolated collections of that software, so that the tools of one project don't collide with those of another.

### The solver

When you ask Conda for a package, it doesn't just download it. It solves a puzzle: find a version of every package (the one you asked for, its dependencies, their own dependencies, and so on) such that all of them are compatible with each other and with what is already installed.

This is a genuinely hard problem (a satisfiability, or "SAT", problem), which is why creating a complex environment can take a while.

To see how much hides behind a short request: in the example used in [Part 3]({{ site.baseurl }}/modules/reproducible-research/part-3-workflows-with-nextflow/#what-a-run-looks-like), an environment file listing just three tools (Python, FastQC and MultiQC) made Conda install **150 packages**, about 400 MB, including Java and Perl, because FastQC needs them to run. Conda found and installed all of them, in compatible versions, without being asked.

---

## 3. An environment is just a folder

- A **Conda environment** is a self-contained folder holding a specific version of Python (or R, or other software) plus a specific set of packages at specific versions.
- **Activating** an environment just tells your computer: use the tools and versions stored in this folder.
- It is not a virtual machine. It is not a container. It is not magic.

When you activate an environment, its name appears at the start of your command prompt, so you always know which one you are using:

```
(base) user:~$ conda activate wmgs
(wmgs) user:~$
```

`base` is the environment Conda starts with. Keep it for Conda itself, and create a separate environment for each project or analysis.

---

## 4. How to install Conda

| Installer | What it is | Note |
|---|---|---|
| Anaconda Distribution | Huge bundle: Conda + 250+ packages + a graphical interface | Large; uses the commercial "defaults" channel |
| Miniconda | Minimal: Conda + Python essentials | From Anaconda Inc.; still points at "defaults" |
| **Miniforge** | Minimal: Conda + mamba, with conda-forge preset | **Recommended**: BSD/MIT licensed, independent |

<div class="panel" markdown="1">
**Licensing wrinkle.** Anaconda Inc.'s terms of service can require a paid licence for larger organisations using their installers or their "defaults" channel. Many universities and companies have moved to Miniforge with conda-forge, which is free and community-run.
</div>

Installation guides:

- [docs.conda.io: installing Conda](https://docs.conda.io/projects/conda/en/latest/user-guide/install/index.html)
- [docs.conda.io: getting started](https://docs.conda.io/projects/conda/en/latest/user-guide/getting-started.html)

---

## 5. Channels: where packages come from

A **channel** is a repository of Conda packages. Conda downloads packages from the channels you tell it to use.

| Channel | What it contains |
|---|---|
| **conda-forge** | A big community channel; the de facto standard |
| **bioconda** | Bioinformatics tools; it depends on conda-forge |
| **defaults** | Anaconda Inc.'s channel (see the licensing wrinkle above) |

Because bioconda packages rely on packages from conda-forge, the two are used together, with conda-forge listed first. More about channels: [docs.conda.io: managing channels](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-channels.html).

---

## 6. The core command vocabulary

```bash
# create an environment with specific packages
conda create -n microbiome python=3.11 kraken2 bracken bowtie2 samtools

# activate = use this environment's software
conda activate microbiome
conda deactivate

# add / remove packages in an environment
conda install -n microbiome multiqc metaphlan humann
conda remove -n microbiome multiqc

# inspect
conda env list    # all environments
conda list        # packages in the active environment

# update
conda update kraken2
conda update --all

# delete an environment (it's just a folder)
conda env remove -n microbiome
```

Notice how these commands follow the anatomy you met in the Command line module: the command (`conda`), an action (`create`, `install`...), options (`-n microbiome`), and what to act on.

---

## 7. environment.yml: the reproducible way

Typing `conda create` by hand isn't reproducible: in three months, you won't remember exactly what you typed. The proper artifact is an **environment file**, usually called `environment.yml`:

```yaml
# environment.yml
name: microbiome
channels:
  - conda-forge
  - bioconda
dependencies:
  - python=3.11
  - kraken2=2.1.3
  - bracken=2.9
  - metaphlan=4.1.1
  - bowtie2=2.5.4
  - samtools=1.19
  - pip:
      - some-pypi-only-pkg==2.0.1
```

It records the environment's name, the channels to use (in order), and every tool with its version. Anyone with this file can rebuild the same environment:

```bash
# build the exact same environment from the file
conda env create -f environment.yml

# capture everything installed in the active environment into a file
conda env export > environment.yml

# capture only what you explicitly asked for
conda env export --from-history > environment.yml
```

The two `export` options make different files. The first lists every package, including the 150 or so dependencies, with their exact builds: the most precise record, but tied to your operating system. The second lists only the packages you asked for, which is shorter and easier to rebuild on another system.

**The environment file belongs with your code.** Put it in your project folder, next to your scripts and README (see the [FAIR data module]({{ site.baseurl }}/modules/fair-data/part-2-data-horror-show-and-hands-on/#raw-data-is-read-only)): it is part of the record of how your results were made.

---

## 8. Conda vs pip

<figure>
  <img src="{{ site.baseurl }}/modules/reproducible-research/images/conda-vs-pip.png" alt="A Conda snake and a pip cube pulling on the same bioinformatics package">
  <figcaption>Conda and pip can end up fighting over the same packages.</figcaption>
</figure>

| Conda | pip |
|---|---|
| Installs packages of any language, from Conda channels | Installs Python packages only, from PyPI |
| Solves compatibility across all of its packages | Does not manage non-Python dependencies |

They can coexist in the same environment, but **order matters**:

1. Install everything you can with Conda first.
2. Then use `pip install` for the remainder, inside the activated Conda environment.

<div class="panel" markdown="1">
**Possible issue:** installing pip packages first, or mixing Conda and pip repeatedly, can lead to Conda "not knowing" what pip changed, producing broken environments. In an `environment.yml`, the pip packages go in their own `pip:` section at the end, as in the example above.
</div>

---

## What's next

[Part 3: Workflows with Nextflow]({{ site.baseurl }}/modules/reproducible-research/part-3-workflows-with-nextflow/) shows how to chain tools into a pipeline, and uses a Conda environment file to provide the software.
