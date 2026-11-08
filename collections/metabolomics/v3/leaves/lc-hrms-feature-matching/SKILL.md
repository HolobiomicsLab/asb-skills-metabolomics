---
name: lc-hrms-feature-matching
description: Use when you hold a processed LC-HRMS feature table (accurate m/z + RT
  per feature, per sample) and a curated annotation database, and need to score which
  database entries are detected across samples and stratify them by detection frequency.
when_to_use_negative:
- The compound list is unannotated or lacks accurate-m/z and RT values — feature matching
  requires curated accurate m/z and retention time per database entry
- The goal is de novo discovery of unknown xenobiotic metabolites from raw MS data
  — use the reaction-guided CMDN annotation workflow instead, which was designed to
  overcome reference-library-dependent 'bottom-up' search limits
- Samples lack a processed feature table (only raw vendor/mzML files are available)
  — feature detection and alignment must be performed first, e.g., with MS-Dial
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: CMDN
  role: Source of the curated, reaction-guided PMDB entries (accurate m/z + RT) that
    are screened against the feature table
  repo: https://github.com/LeaveMeNotTonight/CMDN
- name: LC-HRMS
  role: Instrument platform producing the cohort feature table (accurate m/z, RT,
    MS2) used as the matching target
- name: TraceFinder
  role: Provides the established detection criteria applied when matching database
    entries to sample features
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.est.5c08558
    title: Reaction-Guided Metabolomics Accelerates High-Throughput Annotation of
      Xenobiotic Metabolites for Human Exposome
  source_content_hash: sha256:d6a4c2d97b4ca6a697c8a45426431f0c4b88b406cd11cf5fdb41f90fb51698e9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/LeaveMeNotTonight/CMDN
  tools:
  - CMDN
  - LC-HRMS
  - TraceFinder
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# LC-HRMS feature matching against a curated database (nDATA/TraceFinder screening)

## Summary

Screens a curated, accurate-m/z and retention-time annotated compound database (e.g., the CMDN-annotated PMDB, N = 492 pesticide metabolites) against a processed LC-HRMS feature table from a sample cohort to determine which compounds are present, at what detection frequency, and with what MS intensities.

## When to use

Use when you hold a processed LC-HRMS feature table (accurate m/z + RT per feature, per sample) and a curated annotation database, and need to score which database entries are detected across samples and stratify them by detection frequency.

## When NOT to use

- The compound list is unannotated or lacks accurate-m/z and RT values — feature matching requires curated accurate m/z and retention time per database entry
- The goal is de novo discovery of unknown xenobiotic metabolites from raw MS data — use the reaction-guided CMDN annotation workflow instead, which was designed to overcome reference-library-dependent 'bottom-up' search limits
- Samples lack a processed feature table (only raw vendor/mzML files are available) — feature detection and alignment must be performed first, e.g., with MS-Dial

## Inputs

- Curated compound database with accurate m/z and retention time per entry (e.g., CMDN-annotated PMDB, Table S2 sheet 1, N = 492 pesticide metabolites)
- Processed LC-HRMS feature table from cohort samples (m/z, RT, MS intensity per feature per sample; e.g., 438 BCP urine samples)
- Detection-criteria configuration from nDATA/TraceFinder defining per-sample detected/not-detected calls

## Outputs

- Per-metabolite detection matrix (detected/not detected in each sample) with MS intensities
- Detection-frequency classification per metabolite: high (>50%), moderate (10-50%), low (<10%)
- Per-subject counts of distinct identified metabolites and cohort mean (e.g., ~11 identifications per subject)
- CSV summary of detected metabolites with frequency-group labels (e.g., pmdb_bcp_screening_results.csv)

## How to apply

Load the curated database (here: Table S2 sheet 1, the CMDN-annotated PMDB with N = 492 manually curated pesticide metabolites carrying accurate m/z and retention time) and the processed LC-HRMS feature table from the cohort (here: 438 BCP urine samples). Apply the nDATA/TraceFinder detection criteria to match each database entry against sample features on accurate m/z and RT, marking each metabolite as detected or not detected per sample. Classify detected metabolites into frequency bins: high (>50% of samples), moderate (10-50%), and low (<10%). Count distinct metabolite identifications per subject (cohort mean ~11 for the BCP urine example) and export the detected entries with frequency-group labels and per-subject summaries to a CSV (e.g., pmdb_bcp_screening_results.csv). Rationale: the database was built by reaction-guided annotation (CMDN over enzymatic S9 biotransformation assays of 1,021 pesticides) and manually curated, so matching is restricted to plausible biotransformation products rather than an open spectral-library search.

## Related tools

- **CMDN** (Source of the curated, reaction-guided PMDB entries (accurate m/z + RT) that are screened against the feature table) — https://github.com/LeaveMeNotTonight/CMDN
- **LC-HRMS** (Instrument platform producing the cohort feature table (accurate m/z, RT, MS2) used as the matching target)
- **TraceFinder** (Provides the established detection criteria applied when matching database entries to sample features)

## Examples

```
results <- match_features(pmdb_table_s2, feature_table_bcp438, criteria = nDATA_tracefinder); write.csv(results$detected, "pmdb_bcp_screening_results.csv")
```

## Evaluation signals

- Screen recovers exactly the expected number of detected metabolites (92 in the BCP urine reference run)
- Frequency-group assignments partition detected metabolites into exactly three bins: >50%, 10-50%, and <10% of samples
- Mean distinct metabolite identifications per subject equals the reported cohort value (~11 per subject for the 438-sample BCP cohort)
- Every detected call is backed by a matched feature with an MS intensity value in the source feature table
- Output CSV contains one row per detected metabolite with a valid frequency-group label and the per-subject summary is internally consistent with the detection matrix

## Limitations

- Screening only finds metabolites present in the curated database — compounds outside the PMDB (e.g., biotransformations of pesticides not among the 1,021 assayed) will be missed
- Detection is bounded by the quality of the upstream feature table; poorly detected low-intensity metabolites may fall in the low-frequency (<10%) bin or be missed
- Matches without authentic-standard verification remain annotations; the paper validates only via multichannel evidence (standards, RT prediction, murine studies, time-course logistic modeling)

## Evidence

- [results] Screening the PMDB against the BCP urine cohort identified a total of 92 pesticide metabolites, exhibiting varying detection frequencies and MS intensities across the 438 samples.: [evidence span withheld — anchor status: not_found]
