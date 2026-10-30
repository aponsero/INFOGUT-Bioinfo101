---
title: "Part 1: Sources of variation"
---

# Part 1: Sources of variation
{: .no_toc}

<div class="panel" markdown="1">
**Format:** lecture notes, no software needed.<br>
**Author:** Eliana Ibrahimi, University of Tirana, Albania.
</div>

**On this page**

* TOC
{:toc}

---

## 1. Design failures are invisible, until someone tries to replicate you

> *"Across biology, most irreproducible findings trace back to how the experiment was designed, not which statistical test was run afterward."*

Before you trust a p-value, trust the design. Every study goes through the same steps: study design, data collection, data analysis, and conclusions and reporting. A flaw in the first step cannot be fixed by any of the later ones.

- 1 to 3 vessels per condition is still common in published in vitro colon studies, often too few to detect anything but the largest effects.
- Small studies don't just miss real effects, they systematically inflate the size of the ones they do find.

*Reading: Schloss (2018), mBio, on reproducibility threats in microbiome research; Button et al. (2013), Nat Rev Neurosci, on how low power inflates observed effect sizes.*

---

## 2. A quick map of the systems

Every design choice on this page applies to any of these platforms. The details differ, the variance sources don't.

<figure>
  <img src="{{ site.baseurl }}/modules/experimental-design/images/platforms.png" alt="Three types of in vitro colon model: batch fermentation, continuous systems, and multi-compartment and mucosal systems">
  <figcaption>Batch fermentation: sealed vessel, single inoculum, hours-long run. Continuous systems (chemostats, SHIME, TIM-2, M-ARCOL): continuous feed and effluent removal, steady state over days to weeks. Multi-compartment and mucosal systems: simulated colon regions with luminal and mucosal phases (e.g. M-SHIME). Platforms compared in Payne et al. (2012), Trends in Biotechnology.</figcaption>
</figure>

---

## 3. Four sources of variation compete with your effect

Each of these is a real, estimable source of variance: something your design must isolate.

<figure>
  <img src="{{ site.baseurl }}/modules/experimental-design/images/sources-of-variation.png" alt="Diagram of four sources of variation (donor, vessel, batch or run, technical) surrounding the treatment effect">
</figure>

| Source | What it is |
|---|---|
| **Donor** | Whose microbiota founded the system: the largest single source |
| **Vessel** | Run-to-run drift, even from identical starting communities |
| **Batch / run** | Day, operator, media lot, equipment: easy to overlook |
| **Technical** | Repeated sampling, extraction, sequencing of one sample |

Your experiment must be designed to separate these four from the effect you actually care about. That separation is what the rest of this lecture builds.

---

## 4. Inter-individual variability is the largest source of variance

<figure>
  <img src="{{ site.baseurl }}/modules/experimental-design/images/donor-variability.png" alt="Bar chart of illustrative phylum composition for three donors">
  <figcaption>Illustrative values, not real data. The spread matters, not the exact numbers (Knight et al., 2018, Nat Rev Microbiol).</figcaption>
</figure>

- Composition, diversity, and resilience differ enormously from one donor to the next, often more than any treatment you'll apply.
- A single-donor pilot tells you about that donor's microbiota, not "the microbiome".
- Treat donor as a blocking or random-effect factor to be modelled, not a nuisance to average away.
- More donors improves generalizability; fewer donors buys more vessels per donor.

---

## 5. Identical inputs, divergent communities

Split one donor's inoculum into parallel vessels and, days later, each has settled into its own slightly different community.

<figure>
  <img src="{{ site.baseurl }}/modules/experimental-design/images/vessel-divergence.png" alt="One donor inoculum split into three vessels, each reaching a different state">
</figure>

Parallel vessels from the same donor still drift apart through stochastic colonization and selection. Vessel is a real source of variance, not just measurement noise, and it's why "one vessel per condition" is not a real experiment.

---

## 6. Know what your "n" actually counts

This single distinction causes more inflated significance in in vitro microbiome papers than any other design mistake.

<figure>
  <img src="{{ site.baseurl }}/modules/experimental-design/images/replicates.png" alt="Two vessels as biological replicates, each with three technical samples">
</figure>

- A **biological replicate** is an independent vessel or donor, with its own founding community.
- A **technical replicate** is a repeated sampling (for example, sequencing from the same vessel), and measures the same biological replicate.
- Treating three technical replicates as three biological ones triples your apparent n and shrinks your p-value. This is called **pseudoreplication**.

---

## 7. How many vessels, how many donors?

<figure>
  <img src="{{ site.baseurl }}/modules/experimental-design/images/donors-vs-vessels.png" alt="Within-donor replication (one donor, several vessels) compared with across-donor replication (one vessel per donor, several donors)">
</figure>

**Within-donor replication:** one donor, several parallel vessels. Controls for donor variability by holding it constant, but conclusions only generalize to that one donor's microbiota.

**Across-donor replication:** one vessel per donor, several donors. Costs more vessels for the same total n, but the result generalizes across the population you sampled donors from.

**Best practice where feasible:** donors × vessels per donor, a nested design analysed with donor as a random effect. Typical published studies use 3 to 6 donors and 2 to 3 vessels per condition: a floor, not a target.

---

## 8. Every treatment needs a matched comparator

<figure>
  <img src="{{ site.baseurl }}/modules/experimental-design/images/controls.png" alt="Four types of control: blank or untreated, vehicle, positive, and pre-treatment baseline">
</figure>

| Control | Purpose |
|---|---|
| **Blank / untreated** | Tracks natural drift over the incubation, independent of any treatment |
| **Vehicle control** | Isolates the compound's effect from its solvent or delivery matrix |
| **Positive control** | A compound with a known effect: confirms the system responds at all |
| **Pre-treatment baseline** | Same vessel, sampled before dosing, so it serves as its own reference |

Keep the baseline control in mind: it becomes the key to a much cheaper sample-size calculation in [Part 2]({{ site.baseurl }}/modules/experimental-design/part-2-power-and-sample-size/).

---

## 9. The variables you didn't mean to test

- Media batch, incubator drift, operator technique: anything that changes between sessions.
- Reactor position or feeding order within a run: details that can still track your grouping.
- Time of harvest and processing delay before extraction, which shift community composition on their own.
- Sequencing batch or lane effects downstream of the wet lab, invisible until you check for them.

<figure>
  <img src="{{ site.baseurl }}/modules/experimental-design/images/confounded.png" alt="Treatment assignment where operator A handles all controls and operator B all treated vessels">
  <figcaption>Confounded design: operator changes exactly track treatment.</figcaption>
</figure>

Confounding happens when a hidden variable tracks your treatment groups instead of cutting across them (Sinha et al., 2017, Nat Biotechnol).

---

## 10. Randomize, block, and run in parallel

<figure>
  <img src="{{ site.baseurl }}/modules/experimental-design/images/balanced.png" alt="Treatment assignment where operators A and B alternate across control and treated vessels">
  <figcaption>Balanced design: operator no longer tracks treatment.</figcaption>
</figure>

- **Randomize** vessel position, feeding order, and processing order so hidden variables scatter evenly across groups.
- **Block** by donor and by run. Compare within blocks, not across them, so the block itself absorbs that variance.
- **Run every condition within a batch in parallel:** same day, same reagent lot, same operator rotation.
- **Record run metadata** even if you don't plan to model it yet: you may need it later to explain an outlier.

---

## 11. Repeated samples from one vessel aren't independent

<figure>
  <img src="{{ site.baseurl }}/modules/experimental-design/images/sampling-timeline.png" alt="Timeline with a stabilization period of about one to two weeks, a baseline, and samples at days 3, 7, 10 and 14 of the treatment period">
</figure>

- Samples from the same vessel over time are autocorrelated, so four timepoints are not four independent observations.
- Define "baseline" per vessel, not pooled across vessels, because vessels stabilize to different states even under identical protocols.
- This is exactly what repeated-measures and mixed-effects models exist to handle, by modelling vessel as a random effect (see the [Statistics module]({{ site.baseurl }}/modules/statistics/programme/)).

---

## 12. A quick gut check

<div class="panel" markdown="1">
Take a minute to think about these. There is no need to write anything down.

- Which of your last three experiments had a hidden confound you didn't check for?
- Right now, could you say whether your reported "n" counts vessels, or samples from one vessel?
- If a reviewer asked for your power calculation today, would you have one to show them?
</div>

The last question is the subject of [Part 2: Power and sample size]({{ site.baseurl }}/modules/experimental-design/part-2-power-and-sample-size/).
