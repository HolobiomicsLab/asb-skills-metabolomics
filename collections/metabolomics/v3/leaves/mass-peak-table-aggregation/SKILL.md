---
name: mass-peak-table-aggregation
description: Use when you need a theoretical isotopic fine-structure peak table (m/z,
  abundance) for a user-defined chemical formula or peptide sequence with natural-abundance
  or SIP-enriched isotopes at high resolution.
when_to_use_negative:
- You need the full isotopic envelope of a metabolite with known formula rather than
  resolved fine structure — Aerith's FFT-based algorithm is the intended path and
  is faster, though it does not capture fine structure
- The compound contains partially incorporated isotopes from SIP and you plan to use
  IsoSpec2 instead — IsoSpec2 only supports natural-abundance fine-structure simulation,
  but Aerith is the correct tool here, not the wrong one; the wrong choice is using
  IsoSpec2 or Miso for partial labeling
- You only have observed experimental spectra to annotate rather than a user-defined
  formula to simulate
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: Aerith
  role: R package whose simulation kernel performs the multinomial Monte Carlo fine-structure
    simulation and peak aggregation
  repo: https://github.com/thepanlab/Aerith
- name: C++
  role: Implementation language of the Monte Carlo simulation kernel for computational
    efficiency
- name: Rcpp
  role: Bridge exposing the C++ kernel to R for real-time interactive use
- name: R
  role: User-facing environment in which Aerith is installed and invoked
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.5c03207
    title: 'Aerith: Visualization and Annotation of Isotopic Enrichment Patterns of
      Peptides and Metabolites with Stable Isotope Labeling from Proteomics and Metabolomics'
  source_content_hash: sha256:cdecdc404c418a4a858f2f27da4754f7578fdab4d65716b3bd954044c137efed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thepanlab/Aerith
  tools:
  - Aerith
  - C++
  - R
  - Rcpp
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# mass-peak-table-aggregation

## Summary

Aggregates Monte Carlo-sampled isotopologue compositions from Aerith's multinomial fine-structure simulation into discrete fine-structure peaks by exact m/z, summing replicate counts to obtain relative abundances for a user-defined formula at a specified stable-isotope enrichment level (0–100%).

## When to use

Use when you need a theoretical isotopic fine-structure peak table (m/z, abundance) for a user-defined chemical formula or peptide sequence with natural-abundance or SIP-enriched isotopes at high resolution.

## When NOT to use

- You need the full isotopic envelope of a metabolite with known formula rather than resolved fine structure — Aerith's FFT-based algorithm is the intended path and is faster, though it does not capture fine structure
- The compound contains partially incorporated isotopes from SIP and you plan to use IsoSpec2 instead — IsoSpec2 only supports natural-abundance fine-structure simulation, but Aerith is the correct tool here, not the wrong one; the wrong choice is using IsoSpec2 or Miso for partial labeling
- You only have observed experimental spectra to annotate rather than a user-defined formula to simulate

## Inputs

- User-defined chemical formula (e.g. glucose, penicillin) or peptide sequence
- User-specified stable-isotope enrichment level for the SIP-labeled element(s) in atom% (0–100%)
- Natural-abundance isotope ratios for each element

## Outputs

- Fine-structure peak table (m/z, relative abundance) written to fine_structure_peaks.csv
- Discrete fine-structure peaks aggregated from multinomial Monte Carlo samples

## How to apply

Parse the user-defined formula into its elemental composition and enumerate the isotopologues of each element. Set the isotope abundances of the SIP-labeled element(s) to the user-specified atom% enrichment, starting from natural-abundance isotope ratios. Draw Monte Carlo samples of isotopologue compositions from the multinomial distribution — this captures low-abundance fine-structure variants that an FFT-based isotopic envelope calculation does not account for. Aggregate the sampled isotopologues into discrete fine-structure peaks by exact m/z, summing counts across replicates to obtain relative abundances, and implement the kernel in C++ exposed to R via Rcpp so repeated simulation remains fast enough for real-time interactivity.

## Related tools

- **Aerith** (R package whose simulation kernel performs the multinomial Monte Carlo fine-structure simulation and peak aggregation) — https://github.com/thepanlab/Aerith
- **C++** (Implementation language of the Monte Carlo simulation kernel for computational efficiency)
- **Rcpp** (Bridge exposing the C++ kernel to R for real-time interactive use)
- **R** (User-facing environment in which Aerith is installed and invoked)

## Examples

```
Rscript -e 'library(Aerith); peaks <- simulate_fine_structure(formula="C6H12O6", isotope="C", enrichment=0.50); write.csv(peaks, "fine_structure_peaks.csv", row.names=FALSE)'
```

## Evaluation signals

- Simulated fine-structure pattern (peak masses and relative abundances) is consistent with the fine structures the paper reports resolving for case-study compounds such as glucose and penicillin
- Output file fine_structure_peaks.csv contains m/z and abundance columns for every aggregated peak
- At 0% enrichment the aggregated pattern reduces to the natural-abundance fine structure of the formula
- At 100% enrichment all sampled isotopologues carry the fully heavy isotope, shifting the peak cluster to the fully labeled nominal mass
- Increasing the number of Monte Carlo draws stabilizes relative abundances of low-abundance fine-structure variants

## Limitations

- Monte Carlo aggregation is stochastic; relative abundances of very low-abundance fine-structure variants depend on the number of draws
- The FFT-based envelope algorithm (the faster alternative) does not account for isotopic fine structure resolvable by HRMS, so the Monte Carlo path must be used when fine structure matters
- Aerith's fine-structure simulation described here is a mechanism to be reconstructed (Attachment S1), not a validated benchmark result

## Evidence

- [abstract] Aerith simulates fine isotopic structures of a compound using Monte Carlo simulation via the multinomial distribution: "Aerith simulates fine isotopic structures of a compound using Monte Carlo simulation via the multinomial distribution"
- [abstract] These algorithms were implemented in C++ and accessed in R via Rcpp: "These algorithms were implemented in C++ and accessed in R via Rcpp to ensure real-time interactivity"
- [intro] IsoSpec2 does not support partial SIP incorporation: "IsoSpec2 provides rapid simulation of isotopic fine structures under natural abundance conditions, but it does not support analysis of compounds with partially isotope incorporation"
- [results] FFT method does not capture fine structure: "it does not account for isotopic fine structure, which can be resolved experimentally via high-resolution mass spectrometry (HRMS)"
- [readme] Aerith is an R package providing interfaces to read and write mass spectrum scans and calculate theoretical isotopic peak envelopes: "Aerith is an R package that provides interfaces to read and write mass spectrum scans, calculate the theoretical isotopic peak envelope of peptide precursors and their B Y ions"
