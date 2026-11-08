---
name: istd-normalization-rsd-calculation
description: Use when peak-area tables from GC-MS amino acid analysis must be ISTD-normalized
  one-to-one and precision summarized as average RSD to compare derivatization agents
  (PCF, MTBSTFA, MSTFA) across plasma sample sets.
when_to_use_negative:
- Normalizing large panels of unannotated metabolites lacking cognate isotope-labeled
  standards — the article reports that internal-standard normalization did not yield
  median precision below 30% across all 563 metabolite annotations, so machine-learning
  normalization (SERDA/SERRF) is the appropriate choice there
- Input consists of pooled QC samples intended for SERDA/SERRF model training rather
  than a small-set derivatization-precision comparison
- Peak areas have already been normalized (e.g., to a total ion signal or batch model),
  since re-normalization to cognate ISTDs requires raw peak areas
tools:
- name: R
  role: Scripting environment used to divide each amino acid's raw peak area by its
    cognate internal-standard peak area and compute per-trial RSDs; the paper runs
    R 3.6.3 on Windows 10
- name: gas chromatography-mass spectrometry (GC-MS)
  role: Instrument platform that generated the derivatized plasma peak-area data being
    normalized and precision-tested
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo13080944
    title: Denoising Autoencoder Normalization for Large-Scale Untargeted Metabolomics
      by Gas Chromatography–Mass Spectrometry
  source_content_hash: sha256:1272686ccfa0023d6fcb00458caa50204f04d492de39f275ed41d4a45cb7bb60
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - gas chromatography-mass spectrometry (GC-MS)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# istd-normalization-rsd-calculation

## Summary

Computes analytical precision of GC-MS derivatization reagents by one-to-one normalization of amino acid peak areas to their cognate stable-isotope-labeled internal standards in R, then averaging per-trial relative standard deviations (RSD) across replicate trials to benchmark PCF (2.7%) vs MTBSTFA (8.9%) vs MSTFA (9.6%).

## When to use

Use when peak-area tables from GC-MS amino acid analysis must be ISTD-normalized one-to-one and precision summarized as average RSD to compare derivatization agents (PCF, MTBSTFA, MSTFA) across plasma sample sets.

## When NOT to use

- Normalizing large panels of unannotated metabolites lacking cognate isotope-labeled standards — the article reports that internal-standard normalization did not yield median precision below 30% across all 563 metabolite annotations, so machine-learning normalization (SERDA/SERRF) is the appropriate choice there
- Input consists of pooled QC samples intended for SERDA/SERRF model training rather than a small-set derivatization-precision comparison
- Peak areas have already been normalized (e.g., to a total ion signal or batch model), since re-normalization to cognate ISTDs requires raw peak areas

## Inputs

- Supplementary Table S1 peak-area table: 16 amino acids with paired stable-isotope-labeled internal standards, 30 BioIVT plasma samples, three monthly replicate trials
- Derivatization reagent annotation per injection (PCF, MTBSTFA, MSTFA)
- Sample metadata identifying the BioIVT plasma samples and trial month

## Outputs

- istd_aminoacid_rsd.csv: summary table of average RSD (%) per derivatization reagent (PCF, MTBSTFA, MSTFA)
- Per-trial, per-amino-acid RSD values underlying the reagent-level averages

## How to apply

Load the peak-area table (Supplementary Table S1) containing the 16 detectable amino acids paired with their stable-isotope-labeled internal standards for the 30 BioIVT plasma samples acquired in three monthly replicate trials under PCF, MTBSTFA, and MSTFA derivatization. In R, apply one-to-one ISTD normalization by dividing each amino acid's raw peak area by its cognate internal-standard peak area for every injection — pairing is strictly analyte-to-analyte, not a global or sum-based scaling. Compute the RSD (%) of each ISTD-normalized amino acid across the 30 samples within each of the three monthly replicate trials, separately for each derivatization reagent. Average the per-trial amino-acid RSDs across the three monthly trials to obtain one summary RSD per reagent. Validate the result against the reference values reported in Figure 3 / Supplementary Table S1: PCF 2.7%, MTBSTFA 8.9%, MSTFA 9.6%, with PCF giving the lowest precision. Write the summary table to istd_aminoacid_rsd.csv.

## Related tools

- **R** (Scripting environment used to divide each amino acid's raw peak area by its cognate internal-standard peak area and compute per-trial RSDs; the paper runs R 3.6.3 on Windows 10)
- **gas chromatography-mass spectrometry (GC-MS)** (Instrument platform that generated the derivatized plasma peak-area data being normalized and precision-tested)

## Evaluation signals

- Reproduced average RSDs match the reference values: PCF 2.7%, MTBSTFA 8.9%, MSTFA 9.6%
- PCF ranks as the lowest-RSD reagent, with MTBSTFA and MSTFA clustered near 9% — ordering of the three reagents is an invariant of the result
- Normalization used exactly 16 one-to-one amino-acid/ISTD pairs (cognate pairing, not pooled or sum-based scaling)
- RSDs are computed within each of the three monthly replicate trials over 30 BioIVT plasma samples before averaging, not pooled across all injections at once
- Output file istd_aminoacid_rsd.csv contains one summary RSD per derivatization reagent

## Limitations

- Applies only to analytes with a stable-isotope-labeled cognate standard (here, 16 detectable amino acids); the article found neither internal standards nor sum-based normalizations achieved median precision below 30% across all 563 metabolite annotations, so this approach does not scale to the full untargeted panel
- Reference values (2.7%/8.9%/9.6%) are tied to the specific 30 BioIVT plasma samples and three monthly replicate trials of that comparison; other sample sets will yield different RSDs
- Precision here is intra-method reproducibility on a small set, distinct from the large-scale QC-based RSDs reported for SERRF (19% on pooled QC, 34% on external BioIVT pools) and SERDA (16%/19%)

## Evidence

- [other] Reference RSD values per reagent: [evidence span withheld — anchor status: not_found]
- [other] One-to-one ISTD normalization procedure: [evidence span withheld — anchor status: not_found]
- [abstract] Context of BioIVT RSD evaluation in the article: "external cross-validation with BioIVT plasma pools yielded a median 34% relative standard deviation (RSD)"
- [abstract] Scope limitation of ISTD-based normalization: "neither internal standards nor sum-based normalizations yielded median precision of less than 30% across all 563 metabolite annotations"
