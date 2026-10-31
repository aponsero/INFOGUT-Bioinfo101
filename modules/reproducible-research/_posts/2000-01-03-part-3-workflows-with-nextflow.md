---
title: "Part 3: Workflows with Nextflow"
---

# Part 3: Workflows with Nextflow
{: .no_toc}

<div class="panel" markdown="1">
**Format:** lecture notes, nothing to install. The outputs shown on this page come from a real run.<br>
**Estimated time:** 20 minutes.<br>
**Author:** Petra Polakovičová, Institute for Clinical and Experimental Medicine, Czech Republic.<br>
**Before you start:** this page builds on [Part 2]({{ site.baseurl }}/modules/reproducible-research/part-2-software-environments-with-conda/) (environment files) and on loops and scripts from the [Command line module]({{ site.baseurl }}/modules/command-line/part-3-making-the-computer-repeat-itself/).
</div>

**On this page**

* TOC
{:toc}

---

## 1. Dataflow: an assembly line, not a script

A Bash script is a list of instructions run from top to bottom. If you want to process 200 samples, you write a loop, and you decide yourself what runs when.

Nextflow works differently. It treats your analysis as an **assembly line**: input data (for example FASTQ files) enter at one end, pass through a series of stations, and the output comes out at the other end.

<figure>
  <img src="{{ site.baseurl }}/modules/reproducible-research/images/assembly-line.png" alt="Input data (fastq files) on a conveyor belt passing through process 1, process 2 and process 3 to the output">
  <figcaption>A Nextflow workflow as an assembly line: input data pass through a series of processes.</figcaption>
</figure>

You **declare the processes** (the stations), and **connect them with channels** (the conveyor belts). You do not write the order in which things run: Nextflow works it out from how the belts are connected.

<figure>
  <img src="{{ site.baseurl }}/modules/reproducible-research/images/process-runs.png" alt="Items arrive on the input of a process, which runs and sends results to its output">
  <figcaption>A process runs (in parallel) whenever data arrives on its input.</figcaption>
</figure>

<div style="display: flex; gap: 24px; align-items: center; flex-wrap: wrap;">
  <figure style="flex: 0 0 220px; margin: 0;">
    <img src="{{ site.baseurl }}/modules/reproducible-research/images/parallel-stations.png" alt="Three copies of a process, each working on a different item at the same time">
  </figure>
  <ul style="flex: 1 1 300px;">
    <li>You don't tell each station when to work.</li>
    <li>A station works whenever an item arrives on its belt.</li>
    <li>Many identical items can be worked on at different stations simultaneously.</li>
  </ul>
</div>

**The line runs itself:**

- You wire the belts, and the line schedules itself: no manual ordering.
- Each task is isolated: every time a process runs on one item, it works in its own folder.
- Parallelism comes for free: 200 items on a channel means up to 200 runs at once, with no loops to write.

---

## 2. The four building blocks

| Block | What it is |
|---|---|
| **process** | A step |
| **channel** | A data stream |
| **operator** | Transforms a stream |
| **workflow** | The wiring |

### Process: a step

A process wraps one tool call. It has three main sections:

```groovy
process TRIM {
    input:
    tuple val(sample_id), path(reads)

    output:
    tuple val(sample_id), path("*.trimmed.fastq.gz")

    script:
    """
    cutadapt ${reads}
    """
}
```

| Section | What it does |
|---|---|
| `input:` | What the process consumes from a channel. Here, a sample name (`val`) together with its read files (`path`). |
| `output:` | What it emits. This must match the files the script actually creates. |
| `script:` | The real shell command: the actual tool call, written as you would type it in the terminal. |

### Channel: the stream that feeds steps

A channel carries items from one process to the next. Channels are usually created from files:

```groovy
Channel.fromPath('data/*.fastq.gz')            // one item per file
Channel.fromFilePairs('*_{R1,R2}.fastq.gz')    // paired reads -> tuples
```

The `*` and `{R1,R2}` are the wildcards you met in the [Command line module]({{ site.baseurl }}/modules/command-line/part-2-asking-questions-of-your-files/). `fromFilePairs` groups each sample's R1 and R2 files into one item.

### Operator: reshape the stream

Operators transform the items in a channel before they reach the next process:

```groovy
reads_ch
  .map { f -> tuple(f.simpleName, f) }
  .filter { id, f -> id.startsWith('sample') }
  .set { tidy_reads }
```

| Operator | What it does |
|---|---|
| `map` | Transforms each item |
| `filter` | Keeps some items |
| `collect` | Gathers all items into one list |
| `groupTuple` | Groups items by a key |
| `join` | Merges two channels by key |
| `view` | Prints items (useful for debugging) |

### Workflow: wire everything into a pipeline

The `workflow` block connects processes and channels:

```groovy
workflow {
    reads = Channel.fromFilePairs(params.reads)
    QC(reads)
    TRIM(reads)
    TAX_ASSIGNMENT(TRIM.out)
}
```

Read it as wiring, not as a sequence of commands: both `QC` and `TRIM` receive the reads, so they can run at the same time, and `TAX_ASSIGNMENT` receives the output of `TRIM` (`TRIM.out`), so it starts on each sample as soon as that sample is trimmed.

---

## 3. -resume: never redo finished work

- Each task runs in its own folder, which Nextflow remembers.
- Re-run with `-resume` and Nextflow skips the tasks whose inputs and code have not changed, re-running only what is new, and what is downstream of it.

| Task | On re-run with `-resume` |
|---|---|
| COUNT_READS (sampleA) | Cached |
| COUNT_READS (sampleB) | Re-run |
| COUNT_READS (sampleC) | Cached |
| GC (sampleB) | Re-run |

In this example, something changed for sampleB, so its read count is re-run, and so is the next step that depends on it (GC). Samples A and C are taken from the cache. If a run crashes after ten hours, `-resume` restarts from where it stopped, not from the beginning.

---

## 4. Example: the same quality control, twice

To see what Nextflow changes in practice, here is the same small analysis written twice: as a Bash script, and as a Nextflow pipeline. It runs FastQC on every sample, then combines all the FastQC reports into one MultiQC report (see the quality control section of the [Microbiome data module]({{ site.baseurl }}/modules/microbiome/part-2-the-bioinformatics-pipeline/)).

### The inputs

Both versions read the list of samples from a **samplesheet**, a CSV file with one row per sample. A sample with only one FASTQ file (single-end sequencing) leaves `fastq_2` empty:

```
sample,fastq_1,fastq_2
S1,data/1_S103_L001_R1_001.fastq.gz,data/1_S103_L001_R2_001.fastq.gz
S2,data/2_S115_L001_R1_001.fastq.gz,data/2_S115_L001_R2_001.fastq.gz
S3,data/1a_S103_L001_R1_001.fastq.gz,
```

Both use the same Conda environment file (see [Part 2]({{ site.baseurl }}/modules/reproducible-research/part-2-software-environments-with-conda/)):

```yaml
name: qc
channels:
  - conda-forge
  - bioconda
dependencies:
  - python=3.12
  - fastqc=0.12.1
  - multiqc=1.35
```

### Version 1: a Bash script

```bash
#!/bin/bash
# ============================================================
#  Quality control: FastQC for every sample, then MultiQC
# ============================================================
#
#  How to run:
#     conda activate qc
#     bash bash_qc.sh
# ============================================================


# ---- 1. Settings (change these if you need to) -------------

SAMPLESHEET="samplesheet.csv"   # the file with your samples
OUTDIR="manual_results"         # where the results will be saved


# ---- 2. Create the output folders --------------------------

mkdir -p "$OUTDIR/fastqc"
mkdir -p "$OUTDIR/multiqc"


# ---- 3. Run FastQC on every sample -------------------------

# "tail -n +2"  -> read the samplesheet, but skip line 1 (the header)
# "tr -d '\r'"  -> remove hidden Windows line endings, just in case
for line in $(tail -n +2 "$SAMPLESHEET" | tr -d '\r')
do
    # Cut the line at the commas and take column 1, 2 and 3
    sample=$(echo "$line" | cut -d',' -f1)
    read1=$(echo "$line"  | cut -d',' -f2)
    read2=$(echo "$line"  | cut -d',' -f3)

    echo "Starting FastQC for sample: $sample"

    # The "&" at the end runs FastQC in the background,
    # so the loop can already start the next sample.
    # This is what makes all samples run at the same time (in parallel).
    if [ -n "$read2" ]
    then
        fastqc --outdir "$OUTDIR/fastqc" "$read1" "$read2" &   # paired-end
    else
        fastqc --outdir "$OUTDIR/fastqc" "$read1" &             # single-end
    fi
done


# ---- 4. Wait until all FastQC jobs are finished ------------

wait
echo "All FastQC jobs are finished."


# ---- 5. Run MultiQC to combine all reports into one --------

echo "Running MultiQC..."
multiqc --force --outdir "$OUTDIR/multiqc" "$OUTDIR/fastqc"

echo "Done! Open this file in your browser:"
echo "   $OUTDIR/multiqc/multiqc_report.html"
```

Everything in this script uses commands from the [Command line module]({{ site.baseurl }}/modules/command-line/programme/): a variable, a `for` loop, `tail`, `cut`, a pipe. Two lines do the parallel work: the `&` sends each FastQC job to the background so the loop can start the next one, and `wait` holds the script until all of them have finished, before MultiQC runs.

It works well for three samples. But notice what you had to handle yourself: reading the samplesheet column by column, creating the folders, the paired-end and single-end cases, starting the jobs in parallel, and waiting for them. With 300 samples, the `&` would start 300 FastQC jobs at once on your computer. And if the script crashes at sample 250, it starts again from sample 1.

### Version 2: a Nextflow pipeline

```groovy
#!/usr/bin/env nextflow

// Usage: nextflow run main.nf --samplesheet samplesheet.csv -with-conda environment.yml

params.samplesheet = 'samplesheet.csv'
params.outdir      = 'results'

process FASTQC {
    tag "$sample"
    publishDir "${params.outdir}/fastqc", mode: 'copy'

    input:
    tuple val(sample), path(reads)

    output:
    path "*_fastqc.{zip,html}"

    script:
    """
    fastqc --threads ${task.cpus} ${reads}
    """
}

process MULTIQC {
    publishDir "${params.outdir}/multiqc", mode: 'copy'

    input:
    path fastqc_files

    output:
    path "multiqc_report.html"
    path "multiqc_data"

    script:
    """
    multiqc .
    """
}

workflow {
    reads_ch = Channel
        .fromPath(params.samplesheet)
        .splitCsv(header: true)
        .map { row ->
            def reads = row.fastq_2 ? [file(row.fastq_1), file(row.fastq_2)] : [file(row.fastq_1)]
            tuple(row.sample, reads)
        }

    fastqc_out = FASTQC(reads_ch)

    MULTIQC(fastqc_out.collect())
}
```

You can recognise the four building blocks:

| Block | In this pipeline |
|---|---|
| **params** | Settings that can be changed from the command line, like `--samplesheet samplesheet.csv`. They replace the settings section of the Bash script. |
| **process** | `FASTQC` runs FastQC on one sample. `MULTIQC` runs MultiQC on all the reports. Each `script:` block is just the shell command. `publishDir` copies the outputs to the results folder; `tag` labels each task with its sample name in the progress display. |
| **channel** | `reads_ch` reads the samplesheet (`fromPath`) and turns it into one item per sample. |
| **operators** | `splitCsv` reads the CSV rows by column name. `map` builds, for each row, a pair of the sample name and its read files, with one or two files depending on whether `fastq_2` is empty. `collect` waits for all FastQC outputs and gathers them into one list, so that MultiQC runs once, on everything. |
| **workflow** | The wiring: the samples go to `FASTQC`, all FastQC outputs go to `MULTIQC`. |

There is no loop, no `&` and no `wait`: Nextflow runs one FASTQC task per item in the channel, as many at once as the computer allows, and starts MULTIQC when `collect` has everything. The `-with-conda environment.yml` option tells Nextflow to build the environment from the file and run every task inside it.

---

## 5. What a run looks like

*The outputs below come from a real run of these files on three samples of public test data (see section 6). Folder paths have been shortened to `/home/user/...`, and long outputs are shortened where marked with `...`.*

### Building the Conda environment (for the Bash version)

```
$ conda env create -f environment.yml
Channels:
 - conda-forge
 - bioconda
Platform: linux-64
Collecting package metadata (repodata.json): ...working... done
Solving environment: ...working... done

## Package Plan ##

  environment location: /home/user/miniforge3/envs/qc

  added / updated specs:
    - fastqc=0.12.1
    - multiqc=1.35
    - python=3.12

The following packages will be downloaded:
...
                                           Total:       405.2 MB

The following NEW packages will be INSTALLED:
...
  fastqc             bioconda/noarch::fastqc-0.12.1-hdfd78af_0
...
  multiqc            bioconda/noarch::multiqc-1.35-pyhdfd78af_2
...
  openjdk            conda-forge/linux-64::openjdk-25.0.2-h1602c4f_26
...

Preparing transaction: ...working... done
Verifying transaction: ...working... done
Executing transaction: ...working... done
#
# To activate this environment, use
#
#     $ conda activate qc
#
# To deactivate an active environment, use
#
#     $ conda deactivate
```

Three tools were requested; Conda solved and installed 150 packages, including Java (`openjdk`), which FastQC needs. The specs come from the file, and the channels are used in the order given.

### Running the Bash script

```
$ conda activate qc
(qc) $ bash bash_qc.sh
Starting FastQC for sample: S1
Starting FastQC for sample: S2
Starting FastQC for sample: S3
...
Started analysis of 1a_S103_L001_R1_001.fastq.gz
application/gzip
Started analysis of 1_S103_L001_R1_001.fastq.gz
Approx 40% complete for 1a_S103_L001_R1_001.fastq.gz
Approx 35% complete for 1_S103_L001_R1_001.fastq.gz
Approx 80% complete for 1_S103_L001_R1_001.fastq.gz
Approx 80% complete for 1a_S103_L001_R1_001.fastq.gz
Analysis complete for 1_S103_L001_R1_001.fastq.gz
Analysis complete for 1a_S103_L001_R1_001.fastq.gz
...
All FastQC jobs are finished.
Running MultiQC...

/// MultiQC v1.35

       file_search | Search path: /home/user/qc_demo/manual_results/fastqc
            fastqc | Found 5 reports
     write_results | Data        : manual_results/multiqc/multiqc_data
     write_results | Report      : manual_results/multiqc/multiqc_report.html
           multiqc | MultiQC complete
Done! Open this file in your browser:
   manual_results/multiqc/multiqc_report.html
```

The script starts the three samples in quick succession, and their messages arrive mixed together: the jobs really are running at the same time. Five reports are found: two for each paired-end sample, one for the single-end sample.

### Running the Nextflow pipeline

```
$ nextflow run main.nf --samplesheet samplesheet.csv -with-conda environment.yml

 N E X T F L O W   ~  version 26.10.4

Launching `main.nf` [desperate_escher] revision: 7b7a60b2ac

executor >  local (4)
[7f/d9d3d5] FASTQC (S3) | 3 of 3 ✔
[3f/d7b7c9] MULTIQC     | 1 of 1 ✔
Creating env using conda: environment.yml [cache /home/user/qc_demo/work/conda/env-df0c054277ae7d23c01dd953577dd492]
```

This is the final state of Nextflow's progress display, which updates live while the pipeline runs:

- `desperate_escher` is a random name given to this run.
- `executor > local (4)`: four tasks ran on this computer (three FASTQC, one MULTIQC).
- Each process gets one line. `3 of 3 ✔` means the three FASTQC tasks all finished successfully; `(S3)` is the `tag` of the last task to run.
- `[7f/d9d3d5]` is the beginning of the folder where that task ran (see below).
- The Conda environment was built once, and stored in `work/conda` to be reused.

### Every task in its own folder

Each task runs in its own folder inside `work/`:

```
$ ls work
2c  3f  5b  63  7f  a8  conda

$ ls -a work/7f/d9d3d5f9ab4a9ec9877f43f2ee83e2
.               .command.err  .command.run  1a_S103_L001_R1_001.fastq.gz
..              .command.log  .command.sh   1a_S103_L001_R1_001_fastqc.html
.command.begin  .command.out  .exitcode     1a_S103_L001_R1_001_fastqc.zip

$ cat work/7f/d9d3d5f9ab4a9ec9877f43f2ee83e2/.command.sh
#!/bin/bash -ue
fastqc --threads 1 1a_S103_L001_R1_001.fastq.gz
```

This is the folder of the FASTQC task for sample S3. The hidden `.command` files record everything about the task: `.command.sh` is the exact command that was run (the `script:` block, with the variables filled in), `.command.out` and `.command.err` hold what the tool printed, and `.exitcode` holds its exit code (0 means success, as in the Command line module). The input file is a link to the original in `data/` (nothing is copied or modified), and the outputs sit next to it.

When something goes wrong, this is where you look: go to the task's folder and read `.command.sh` and `.command.err`.

The final outputs are copied (by `publishDir`) into the `results/` folder:

```
$ find results -maxdepth 2
results
results/fastqc
results/fastqc/1_S103_L001_R1_001_fastqc.html
results/fastqc/1_S103_L001_R1_001_fastqc.zip
...
results/multiqc
results/multiqc/multiqc_data
results/multiqc/multiqc_report.html
```

### Running it again, with -resume

Running the same command again with `-resume`, without changing anything:

```
$ nextflow run main.nf --samplesheet samplesheet.csv -with-conda environment.yml -resume

 N E X T F L O W   ~  version 26.10.4

Launching `main.nf` [deadly_pasteur] revision: 7b7a60b2ac

[7f/d9d3d5] FASTQC (S3) | 3 of 3, cached: 3 ✔
[3f/d7b7c9] MULTIQC     | 1 of 1, cached: 1 ✔
```

Nothing was run: all four tasks were taken from the cache, with the same task folders as before.

Now, a fourth sample (S4) is added at the end of the samplesheet, and the pipeline is run again with `-resume`:

```
$ nextflow run main.nf --samplesheet samplesheet.csv -with-conda environment.yml -resume

 N E X T F L O W   ~  version 26.10.4

Launching `main.nf` [trusting_boltzmann] revision: 7b7a60b2ac

executor >  local (2)
[63/4f416d] FASTQC (S4) | 4 of 4, cached: 3 ✔
[5b/d73227] MULTIQC     | 1 of 1 ✔
```

Only two tasks ran: FastQC on the new sample, and MultiQC, because its input (the list of all FastQC reports) changed. The three existing samples were taken from the cache. On a real dataset, where each step can take hours, this is what saves your week.

### The result: a MultiQC report

Both versions produce the same MultiQC report: one HTML page combining the FastQC results of all samples.

<figure>
  <img src="{{ site.baseurl }}/modules/reproducible-research/images/mqc_general_stats.png" alt="MultiQC general statistics table with one row per FASTQ file, showing duplication percentage, GC content and number of sequences">
  <figcaption>The General Statistics table of the MultiQC report, with one row per FASTQ file (after the run with four samples).</figcaption>
</figure>

<figure>
  <img src="{{ site.baseurl }}/modules/reproducible-research/images/mqc_per_base_quality.png" alt="MultiQC plot of mean quality scores along the reads for the seven FASTQ files">
  <figcaption>Mean quality score along the reads, for all seven FASTQ files on one plot. The green, orange and red zones are the ones described in the Microbiome data module.</figcaption>
</figure>

---

## 6. Want to try it yourself? (optional)

You don't need to run anything to follow this module. If you are curious, everything used on this page is available:

- [Download the example files (zip)]({{ site.baseurl }}/modules/reproducible-research/data/nextflow-qc-example.zip): `environment.yml`, `bash_qc.sh`, `main.nf`, `samplesheet.csv` and the test FASTQ files, with a README explaining how to run them. You need Conda ([Miniforge](https://github.com/conda-forge/miniforge) recommended) and Nextflow installed on a Linux or macOS computer.
- The official Nextflow training, which runs in the browser through GitHub Codespaces, with nothing to install: [training.nextflow.io/latest/envsetup](https://training.nextflow.io/latest/envsetup/).

---

## 7. Take home messages

- **Conda = the right software, kept separate.** Each project gets its own environment (just a folder), so tools and versions never clash.
- **Nextflow = your analysis as an automatic assembly line.** Steps run in parallel on any number of samples, anywhere, and `-resume` never redoes finished work.
- **Together = reproducible (FAIR) science.** Share your `environment.yml` and your pipeline, and anyone can get the same results.

Many ready-made Nextflow pipelines for microbiome data already exist, maintained by the nf-core community: for example nf-core/ampliseq for 16S data and nf-core/taxprofiler for shotgun data (see the [Microbiome data module]({{ site.baseurl }}/modules/microbiome/part-2-the-bioinformatics-pipeline/)). Most of the time, you will run one of these rather than write your own.
