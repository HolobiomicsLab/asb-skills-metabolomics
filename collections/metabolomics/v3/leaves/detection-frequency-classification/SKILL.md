---
name: detection-frequency-classification
description: Use when a curated metabolite database (e.g., the 492-entry CMDN-annotated
  PMDB) has been screened against processed LC-HRMS feature data from a cohort (e.g.,
  438 BCP urine samples) and detected metabolites must be binned by detection frequency
  per sample and per subject.
when_to_use_negative:
- Input is raw LC-HRMS data that has not yet been processed into a feature table —
  peak detection/feature construction must happen first
- The goal is de novo annotation or discovery of unknown metabolites; this skill only
  classifies detections of curated PMDB entries with known m/z and retention time
- The metabolite set has no curated accurate-m/z and retention-time values to match
  against cohort features
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0091
tools:
- name: CMDN
  role: Produced the curated, CMDN-annotated PMDB (N = 492) that is screened and frequency-binned
    by this skill
  repo: LeaveMeNotTonight/CMDN
- name: TraceFinder
  role: Provides the detection criteria used to match PMDB entries against sample
    features and mark each metabolite detected or not detected per sample
- name: LC-HRMS
  role: Source of the processed feature table from the 438 BCP urine cohort samples
    against which the PMDB is screened
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.est.5c08558
    title: Reaction-Guided Metabolomics Accelerates High-Throughput Annotation of
      Xenobiotic Metabolites for Human Exposome
  source_content_hash: sha256:559cdb92c03da7c91517191e0d6992e77b2ac16eac44b58a1503b6ef75386b3a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/LeaveMeNotTonight/CMDN
  tools:
  - CMDN
  - LC-HRMS
  - TraceFinder
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# detection-frequency-classification

## Summary

Stratifies xenobiotic metabolites detected in cohort LC-HRMS data by how frequently each appears across samples (high >50%, moderate 10–50%, low <10%) and computes per-subject detection counts, converting a binary screen result into an exposure-landscape summary.

## When to use

Use when a curated metabolite database (e.g., the 492-entry CMDN-annotated PMDB) has been screened against processed LC-HRMS feature data from a cohort (e.g., 438 BCP urine samples) and detected metabolites must be binned by detection frequency per sample and per subject.

## When NOT to use

- Input is raw LC-HRMS data that has not yet been processed into a feature table — peak detection/feature construction must happen first
- The goal is de novo annotation or discovery of unknown metabolites; this skill only classifies detections of curated PMDB entries with known m/z and retention time
- The metabolite set has no curated accurate-m/z and retention-time values to match against cohort features

## Inputs

- Curated CMDN-annotated PMDB (Table S2, sheet 1; N = 492 pesticide metabolites with accurate m/z and retention time)
- Processed LC-HRMS feature table from the 438 BCP urine cohort samples
- nDATA/TraceFinder detection criteria / matching configuration

## Outputs

- List of detected metabolites with frequency-group labels (high >50%, moderate 10–50%, low <10%)
- Per-subject counts of distinct metabolite identifications and the cohort mean (~11 per subject)
- pmdb_bcp_screening_results.csv

## How to apply

Load the curated CMDN-annotated PMDB (Table S2, sheet 1; N = 492 manually curated pesticide metabolites defined by accurate m/z and retention time) and the processed LC-HRMS feature table from the cohort samples (438 BCP urine samples). Apply the nDATA/TraceFinder detection criteria to match each PMDB entry against sample features, marking every metabolite as detected or not detected in each sample — do not re-annotate, only match known m/z–RT pairs. For each detected metabolite, compute its detection frequency across all samples and assign it to a frequency group: high (>50% of samples), moderate (10–50%), or low (<10%). Separately, count distinct metabolite identifications per subject and average across the cohort (the reported BCP result is ~11 per subject). Write the detected metabolites with their frequency-group labels plus the per-subject summary to a results file (pmdb_bcp_screening_results.csv). Rationale: frequency stratification turns a presence/absence screen into an exposure landscape, distinguishing pervasive exposures from sporadic ones across the human cohort.

## Related tools

- **CMDN** (Produced the curated, CMDN-annotated PMDB (N = 492) that is screened and frequency-binned by this skill) — LeaveMeNotTonight/CMDN
- **TraceFinder** (Provides the detection criteria used to match PMDB entries against sample features and mark each metabolite detected or not detected per sample)
- **LC-HRMS** (Source of the processed feature table from the 438 BCP urine cohort samples against which the PMDB is screened)

## Examples

```
Rscript -e 'res <- read.csv("pmdb_bcp_screening_results.csv"); res$freq_group <- cut(res$detection_freq, c(0,0.10,0.50,1), labels=c("low","moderate","high"))'
```

## Evaluation signals

- The screen recovers exactly 92 detected metabolites from the 492-entry PMDB against the 438 BCP urine samples
- Frequency-group assignments partition cleanly into the >50%, 10–50%, and <10% bins with no unassigned metabolites
- Mean identifications per subject equals the reported value of approximately 11
- All 438 samples contribute to detection-frequency denominators and to per-subject counts
- Output CSV contains frequency-group labels only for metabolites marked detected, and no entries outside the 492 curated PMDB

## Limitations

- Detection calls rely on nDATA/TraceFinder matching criteria against curated m/z–RT entries rather than authentic analytical standards for all metabolites, so frequency labels inherit any matching error
- Frequency bins are cohort-specific (438 BCP urine samples); the same metabolite could shift groups in a different cohort or matrix
- Coverage is bounded by the manually curated PMDB (N = 492) — metabolites outside it will never be classified

## Evidence
