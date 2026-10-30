---
title: "Part 1: Data life cycle and FAIR"
---

# Part 1: Data life cycle and FAIR
{: .no_toc}

<div class="panel" markdown="1">
**Format:** lecture notes, no software needed.<br>
**Estimated time:** 20 minutes.<br>
**Author:** Alise Ponsero, Quadram Institute Bioscience, Norwich, UK.
</div>

**On this page**

* TOC
{:toc}

---

## 1. Three questions before we start

<div class="panel" markdown="1">
Take a minute with each of these. There is no need to write anything down.

**Could you re-run it?** Take the last figure you published or presented. Starting from the raw files, could you reproduce that exact number today?

**Where is it written?** A reviewer asks which samples you excluded, and why. Where is that recorded, and could you find it this afternoon?

**Could someone take over?** If you left tomorrow, how long would a colleague need to pick up your project and carry on with it?
</div>

---

## 2. This is not a talk about fraud

When people hear "reproducibility crisis", they often think of fabricated numbers, duplicated gel images and deliberate misconduct. That is real and investigated, but it is not what this module is about.

What irreproducibility usually looks like is much more ordinary:

- A threshold chosen once and never written down.
- An intermediate file overwritten by the next version of itself.
- A sample dropped for a perfectly good reason that nobody recorded.
- Knowledge that lives in one head, and leaves when that person does.

Which is good news: the fix is boring, and you can start it this week.

---

## 3. Same lab, same protocol, two years apart

*An invented scenario.*

**Run 1: a clean result.** A postdoc runs the colon model: one control, two treatments, technical replicates, community composition at endpoint. Treatment A produces a strong and convincing shift. The work is written up.

**Run 2, two years later: nothing happens.** A new PhD student repeats it as the starting point of her own project. Same lab. Same protocol document. Same inoculum source. Treatment A does almost nothing.

Nobody did anything wrong. So, what could have changed?

**Try it:** before opening the answer, list everything you can think of, at the bench and at the keyboard.

<details markdown="1">
<summary>Everything that could have changed</summary>

| At the bench | At the keyboard |
|---|---|
| Medium batch and supplier | Sequencing run and depth |
| Inoculum donor, storage and thawing | Pipeline and software versions |
| Vessel, working volume, pH control | Reference database release |
| Incubation and sampling times | Filtering and normalisation thresholds |
| Extraction kit and lot number | Which replicates were excluded |
| Primer set and library preparation | Defaults that changed with an update |

Every item on this list is something that could have been written down at the time of Run 1.
</details>

---

## 4. The data life cycle

The DataONE data life cycle describes the stages data go through.

**When you generate the data yourself:**

| Stage | What happens |
|---|---|
| **Plan** | How the data will be managed and kept usable over their lifetime |
| **Collect** | Observations recorded by hand or by instrument, and put into digital form |
| **Assure** | Quality established through checks and inspections |
| **Describe** | Documented using an appropriate metadata standard |
| **Preserve** | Deposited in an appropriate long-term archive |

Assure and Describe are stages of their own, not things you do afterwards if there is time.

**When you reuse data that already exist:**

| Stage | What happens |
|---|---|
| **Discover** | Useful data located and obtained, together with their metadata |
| **Integrate** | Data from different sources combined into one analysable set |
| **Analyze** | The stage most people think of as the whole job |

Each stage comes with its own tools: a data management plan (DMP) for Plan, a lab notebook for Collect and Assure, checklists and minimum information lists for Describe, and the FAIR principles for Describe and Preserve, which is what makes the Discover stage possible for the next person.

<figure>
  <img src="{{ site.baseurl }}/modules/fair-data/images/data-life-cycle.png" alt="The data life cycle: Plan, Collect, Assure, Describe, Preserve when you generate data, and Discover, Integrate, Analyze when you reuse data, with the DMP, lab notebook, checklists and FAIR principles attached to the stages they support">
  <figcaption>The data life cycle. Top row: when you generate the data yourself. Bottom row: when you reuse data that already exist. Source: DataONE Data Life Cycle.</figcaption>
</figure>

---

## 5. Plan: the data management plan

The first stage of the cycle has a name, a template and a deadline.

| What it is | Why funders ask | Why it helps you |
|---|---|---|
| A short document written at the start of a project. What data will be produced, in what formats, where they will live, who is responsible for them, and what will eventually be shared. | Public money buys a reusable asset, not only a paper. Managing data along FAIR principles and giving open access to them, as open as possible and as closed as necessary, is a mandatory practice for most funders. | It forces three decisions: where the data live, who owns them, and what gets shared. It is also the only way storage and repository costs end up in the budget, since unbudgeted costs cannot be charged later. |

Write one properly once, and the structure carries over to every project after it.

*Source: European Commission, open science in Horizon Europe.*

---

## 6. Describe: the checklists that already exist

Minimum information standards are not bureaucracy. They are a prompt for the things you would otherwise forget to record.

| Checklist | What it covers |
|---|---|
| **MIxS** (Genomic Standards Consortium) | The metadata backbone. Separate checklists for genomes, metagenomes, marker gene sequences, metagenome-assembled genomes and uncultivated viral genomes, plus environmental packages describing the habitat a sample came from. The foundation for standardised microbiome metadata since 2011. |
| **STORMS** (Nature Medicine, 2021) | Seventeen items across the six sections of a paper, built on MIxS together with the STROBE and STREGA epidemiology guidelines. Scoped to human microbiome research, so not your study design, but worth knowing it exists. |
| **STREAMS** (Nature Microbiology, 2025) | Expands STORMS to 67 items covering environmental, synthetic and non-human host-associated studies, explicitly including engineered systems. Machine-actionable, aligned with existing metadata standards, and built from the input of 248 researchers across 28 countries. |

*Sources: Yilmaz et al. 2011; Mirzayi et al. 2021; Kelliher et al. 2025.*

### What a colon model still needs

A checklist can only be as specific as the community that wrote it. Whatever the checklist, record these anyway:

- Vessel type and working volume
- Medium composition, batch and supplier
- Inoculum provenance, storage and thawing
- pH control, temperature, retention time
- Feeding and sampling schedule
- How anaerobic conditions were maintained

If the fields your experiment needs do not exist in any checklist yet, that is not permission to skip them. Put them in your own metadata file, with clear names, explicit units and stated allowed values.

A well-documented supplementary file is what a standard looks like before it is a standard. The fields you and your colleagues converge on are the evidence the next checklist will be built from.

**You are not waiting for this standard to arrive. You are the people it will come from.**

---

## 7. Describe: FAIR is four questions, not an acronym

Ask them about your dataset, on behalf of somebody you will never meet.

| | Question | What it means |
|---|---|---|
| **F: Findable** | Can they find it? | It exists somewhere public, it carries a persistent identifier, and it is described well enough to turn up in a search that was not looking for it by name. |
| **A: Accessible** | Can they get it? | Retrievable through that identifier by a standard protocol. Open where possible. Where it cannot be open, the conditions for access are stated, not implied. |
| **I: Interoperable** | Can their tools read it? | Open formats, standard vocabularies, explicit units. A plain CSV, not a colour-coded workbook that only means something on your screen. |
| **R: Reusable** | May they use it, and do they know enough to? | Clear provenance, metadata rich enough to judge whether it fits their question, and a licence saying what is allowed. |

**FAIR is not a synonym for open.** A restricted dataset can be FAIR, provided the conditions are stated.

### Readable by you is not readable by a machine

Three of the four FAIR questions are about software, not about people.

**What you can read:**

<table>
  <thead><tr><th>Sample</th><th>OD</th></tr></thead>
  <tbody>
    <tr><td>Control R1</td><td>0.42</td></tr>
    <tr style="background-color: #f8d7da;"><td>A R1</td><td>0.38</td></tr>
    <tr><td>A R2</td><td>n/a</td></tr>
  </tbody>
</table>

The pink row means contaminated. That fact exists only in your head and in this one screenshot. No script can filter on a colour, and "n/a" is not a number.

**What a machine can read:**

| sample_id | od600 | qc_flag |
|---|---|---|
| Control_R1 | 0.42 | pass |
| A_R1 | 0.38 | contaminated |
| A_R2 | NA | missing |

The same information, in a column a script can filter on. The unit is in the column name. Missing values use one agreed code, every time.

---

## 8. Preserve: where it goes, and what you get back

Deposit is not the end of the project. It is the thing that makes the rest of it citable.

| Repository | What goes there |
|---|---|
| **ENA** | Nucleotide sequence data: raw reads, assemblies and their sample metadata. Hosted at EMBL-EBI. |
| **SRA** | The counterpart at NCBI for raw sequencing reads, widely used by North American collaborators and journals. |
| **MetaboLights** | Metabolomics experiments, raw and derived data, together with the study metadata that go with them. |
| **Zenodo** | Everything that has no domain repository: tables, documents, figures, supplementary data. Issues a DOI. |

Every deposit hands you back an **accession number or a DOI**. That string belongs in your paper, in your README and in your metadata file. It is the persistent identifier that makes the F in FAIR true, and it is the only permanent address your data will ever have.

---

## What's next

In [Part 2: Data horror-show and hands-on]({{ site.baseurl }}/modules/fair-data/part-2-data-horror-show-and-hands-on/), you look at six examples of bad practice, then clean up a project folder of your own.
