---
title: "Part 2: Power and sample size"
---

# Part 2: Power and sample size
{: .no_toc}

<div class="panel" markdown="1">
**Format:** lecture notes, no software needed.<br>
**Author:** Eliana Ibrahimi, University of Tirana, Albania.<br>
**Before you start:** this page builds on [Part 1]({{ site.baseurl }}/modules/experimental-design/part-1-sources-of-variation/), in particular biological replicates and the pre-treatment baseline control.
</div>

**On this page**

* TOC
{:toc}

---

## 1. Turning an effect size into a number of vessels

Given the effect you're hoping to see, how many independent vessels does it take to detect it?

**Statistical power** is the probability of detecting a true effect, if it really exists, at your chosen significance threshold. It depends on three things you choose before running the experiment:

- the **effect size** you care about (d)
- the **significance level** (α, conventionally 0.05)
- the **power** (1 − β, conventionally 80%)

The effect size comes from pilot data or comparable published work, never from wishful thinking.

<div class="panel" markdown="1">
**Sample size for two independent groups, comparing means**

n per group ≈ 2 (z<sub>1−α/2</sub> + z<sub>1−β</sub>)² / d²

where d is Cohen's d, the standardized difference between the group means.

With α = 0.05 (two-sided): z<sub>1−α/2</sub> = 1.96<br>
With power = 80%: z<sub>1−β</sub> = 0.84<br>
Combined constant: 2 × (1.96 + 0.84)² = 15.7
</div>

---

## 2. Worked example: sizing a butyrate comparison

Pilot data: control butyrate 12.0 mM, prebiotic target 16.0 mM, pooled vessel-to-vessel SD 4.0 mM.

**Step 1. Standardize the effect**

d = (16.0 − 12.0) / 4.0 = 1.0

A difference of one standard deviation: a large effect by conventional benchmarks.

**Step 2. Plug into the formula**

n ≈ 15.7 / 1.0² ≈ 15.7, rounded up to **16 independent vessels per group**.

**Step 3. Total design size**

16 × 2 groups = **32 vessels**. The full unpaired, two-group comparison needs 32 independent biological units.

This is why an in vitro study run with 3 vessels per group only reliably detects very large effects (d ≥ ~1.6) [TO CHECK]. Anything subtler needs more vessels, or a smarter design (section 5).

---

## 3. The sample size and power trade-off

For a fixed α and effect size, power and n move together.

<figure>
  <img src="{{ site.baseurl }}/modules/experimental-design/images/power-curves.png" alt="Power as a function of sample size for effect sizes 0.2, 0.5 and 0.8, at alpha 0.005 and 0.05">
  <figcaption>Power as a function of sample size for three effect sizes (0.2, 0.5, 0.8) and two significance levels (α = 0.005, solid lines; α = 0.05, dashed lines). Source: [CITATION NEEDED]</figcaption>
</figure>

- Going from 80% to 90% power requires ~27% more participants [TO CHECK].
- Going from 80% to 95% requires ~51% more [TO CHECK].

---

## 4. From effect size to sample size

Two independent groups, α = 0.05 (two-sided), power = 80%.

| Cohen's d | Interpretation | n per group | Total n (2 groups) |
|---|---|---|---|
| 0.2 | small | 392 | 784 |
| 0.5 | medium | 63 | 126 |
| 0.8 | large | 25 | 50 |
| **1.0** | **butyrate example** | **16** | **32** |
| 1.2 | very large | 11 | 22 |
| 2.0 | huge | 4 | 8 |

---

## 5. A paired, baseline-corrected design cuts your n sharply

Recall the pre-treatment baseline control from [Part 1]({{ site.baseurl }}/modules/experimental-design/part-1-sources-of-variation/): using it turns an unpaired comparison into a paired one.

- This removes the vessel-to-vessel variance that both groups share, shrinking the effective standard deviation of the comparison.
- Effective SD of the difference: σ<sub>diff</sub> = σ × √(2(1 − ρ)), where ρ is the baseline-to-endpoint correlation within a vessel.
- The sample-size formula for a paired (one-sample) test drops the factor of 2: n ≈ (z<sub>1−α/2</sub> + z<sub>1−β</sub>)² / d<sub>paired</sub>²

<div class="panel" markdown="1">
**Butyrate example, with ρ = 0.6**

σ<sub>diff</sub> = 4.0 × √(2 × 0.4) = 3.58 mM

d<sub>paired</sub> = 4.0 / 3.58 = 1.12

n = 7.84 / 1.12² ≈ 6.3, rounded up to **7 vessels**
</div>

ρ still has to come from real pilot data, not a guess: an optimistic ρ understates the n you actually need.

---

## 6. Web-based calculation tools

[PowerAndSampleSize.com](http://powerandsamplesize.com):

- 120+ calculators backed by peer-reviewed R packages.
- Generates shareable URLs and produces output suitable for manuscripts and grant applications.
- Integrates naturally with the `pwr` R package.

---

## 7. A pre-registration checklist

Before day one of the experiment, check that you have:

1. **Defined the independent units:** donors and vessels, with technical replicates labelled separately.
2. **Specified the controls:** blank, vehicle, positive, baseline.
3. **Written the randomization and blocking plan.**
4. **Logged the potential confounders:** run, position, operator, batch.
5. **Fixed the sampling schedule** and defined the stabilization period.
6. **Stated the effect size, α, and power explicitly**, and calculated n from them rather than assuming it.

---

## 8. Take home

- Identify every source of variation (donor, vessel, batch, technical) before you run a single vessel.
- Your statistical unit is the independent biological replicate; protect that definition.
- Randomize and block what you can't eliminate.
- Calculate n from a real effect size and your chosen α and power, and reach for a paired design before adding vessels.

Next: choosing the right test, correcting for multiple comparisons, and reading effect sizes are covered in the [Statistics module]({{ site.baseurl }}/modules/statistics/programme/).

---

## 9. Exercise

<div class="panel" markdown="1">
How would you design a **crossover** and a **longitudinal** study in a chemostat system, and how would that change the power calculation?
</div>

---

## References

References as cited in the lecture slides:

- Button et al. (2013), Nat Rev Neurosci
- Knight et al. (2018), Nat Rev Microbiol
- Payne et al. (2012), Trends in Biotechnology
- Schloss (2018), mBio
- Sinha et al. (2017), Nat Biotechnol

[TO BE ADDED: full references]
