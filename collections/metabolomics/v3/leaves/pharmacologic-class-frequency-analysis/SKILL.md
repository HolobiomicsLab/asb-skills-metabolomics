---
name: pharmacologic-class-frequency-analysis
description: Use when a cohort-level feature table has been annotated with GNPS Drug
  Library drug IDs and pharmacologic classes, and the goal is exposure readouts such
  as class frequency tallies and demographic (age/sex) association testing.
when_to_use_negative:
- The goal is immediate clinical decision-making — the article explicitly states the
  GNPS Drug Library is 'not intended for immediate clinical decision-making' and only
  lays groundwork for precision medicine
- Reliable self-reported medication records already exist and no empirical cross-check
  or exposure stratification of unannotated metabolites (e.g., N-acyl lipids) is needed
- Input is still raw LC-MS/MS spectra with no MZmine feature extraction or GNPS Drug
  Library annotation performed — run feature detection and spectral matching first
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: GNPS (Global Natural Product Social Molecular Networking)
  role: MS/MS spectral matching source that annotates features with drug identities,
    metabolites/analogs, and the standardized machine-readable pharmacologic class
    metadata that this frequency analysis tallies
  repo: ninahaoqizhao/Manuscript_GNPS_Drug_Library
- name: MZmine
  role: Upstream feature detection, deconvolution, alignment, and MS/MS spectral extraction
    producing the consensus feature table across individuals before class-frequency
    aggregation
  repo: https://github.com/mzmine/mzmine
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-65993-5
    title: A resource to empirically establish drug exposure records directly from
      untargeted metabolomics data
  source_content_hash: sha256:36be1e587b52f07326bd5eca2f7c6b35702b0515d9aa4a1fe0b68b59ba467c9c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
  tools:
  - GNPS (Global Natural Product Social Molecular Networking)
  - MZmine
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# pharmacologic-class-frequency-analysis

## Summary

Aggregate GNPS Drug Library spectral-matching drug annotations across individuals in an untargeted metabolomics cohort, tally pharmacologic class frequencies, and test demographic associations (e.g., age and sex) with chi-square tests to empirically establish drug exposure patterns.

## When to use

Use when a cohort-level feature table has been annotated with GNPS Drug Library drug IDs and pharmacologic classes, and the goal is exposure readouts such as class frequency tallies and demographic (age/sex) association testing.

## When NOT to use

- The goal is immediate clinical decision-making — the article explicitly states the GNPS Drug Library is 'not intended for immediate clinical decision-making' and only lays groundwork for precision medicine
- Reliable self-reported medication records already exist and no empirical cross-check or exposure stratification of unannotated metabolites (e.g., N-acyl lipids) is needed
- Input is still raw LC-MS/MS spectra with no MZmine feature extraction or GNPS Drug Library annotation performed — run feature detection and spectral matching first

## Inputs

- Annotated feature table with per-individual drug IDs and pharmacologic class metadata from GNPS Drug Library spectral matching of MZmine-extracted MS/MS spectra
- Cohort metadata with demographic variables (age, sex) per individual
- Raw LC-MS/MS files (e.g., MassIVE accession MSV000080673) if feature extraction must be redone

## Outputs

- Per-individual drug exposure pattern table
- Pharmacologic class frequency counts across the cohort
- Age/sex stratified chi-square association statistics (e.g., p = 8.5e-05 for NSAIDs in females)
- drug_exposure_results.csv combining annotations, class frequencies, and demographic associations

## How to apply

Start from the annotated feature table produced by MZmine feature detection/deconvolution/alignment plus GNPS Drug Library MS/MS spectral matching, where each feature carries a drug identity, metabolite/analog status, and pharmacologic class metadata. Aggregate annotation hits per individual (presence/absence per drug or class) across the cohort (e.g., 1993 American Gut Project participants from MSV000080673), then tally pharmacologic class frequencies cohort-wide and check that dominant classes correspond to common prescriptions (the reference study recovered 89 drugs whose classes matched). Stratify per-individual drug/class exposure counts by demographic variables (age, sex) and run chi-square tests on the contingency tables — e.g., analgesics/NSAIDs and paracetamol were significantly more frequent in females (p = 8.5e-05). Compile per-individual annotations, class frequency counts, and association statistics into drug_exposure_results.csv. Optionally use exposure-stratified groups as a downstream lens on other metabolites (e.g., microbiota-derived N-acyl lipids), since empirical drug readout can capture exposures missed by self-reported medication use.

## Related tools

- **GNPS (Global Natural Product Social Molecular Networking)** (MS/MS spectral matching source that annotates features with drug identities, metabolites/analogs, and the standardized machine-readable pharmacologic class metadata that this frequency analysis tallies) — ninahaoqizhao/Manuscript_GNPS_Drug_Library
- **MZmine** (Upstream feature detection, deconvolution, alignment, and MS/MS spectral extraction producing the consensus feature table across individuals before class-frequency aggregation) — https://github.com/mzmine/mzmine

## Examples

```
from scipy.stats import chi2_contingency; chi2, p, dof, exp = chi2_contingency(class_by_sex_counts); print(p)  # expect ~8.5e-05 for NSAIDs vs female sex
```

## Evaluation signals

- Detected drug count and class composition should be in the same range as the reference run (89 drugs whose pharmacologic classes matched common prescriptions)
- Reproduce the expected demographic association: analgesics/NSAIDs and paracetamol significantly more frequent in females, chi-square p = 8.5e-05
- Tallied pharmacologic class frequencies should be consistent with known common prescription patterns from the literature
- Output file drug_exposure_results.csv contains the three required components: annotated feature table, pharmacologic class frequency counts, and demographic association statistics
- Every tally per individual should be derivable from the GNPS Drug Library annotation hits — no drug or class without a corresponding spectral match

## Limitations

- The GNPS Drug Library is not intended for immediate clinical decision-making; it supports data-driven exploration where traditional records are limited or unreliable
- Exposure readouts depend on the coverage of the MS/MS reference library; metabolized versions of drugs must be included or exposures are missed if only the parent drug is considered
- Class-frequency associations are statistical (chi-square) readouts of co-occurrence, not validated clinical exposure records

## Evidence

- [methods] Demographic association testing: [evidence span withheld — anchor status: not_found]
- [results] Reference result to reproduce: [evidence span withheld — anchor status: not_found]
- [abstract] Resource scope caveat: "While not intended for immediate clinical decision-making, it supports data-driven exploration of drug exposures where traditional records are limited or unreliable."
- [abstract] Empirical readout beats self-report: "We uncover drug-associated alterations in microbiota-derived N-acyl lipids that are not captured when stratifying by self-reported medication use."
