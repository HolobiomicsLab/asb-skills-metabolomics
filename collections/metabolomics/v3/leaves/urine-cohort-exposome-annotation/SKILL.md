---
name: urine-cohort-exposome-annotation
description: Use when a curated metabolite database with accurate m/z and retention
  time must be screened against a large LC-HRMS urine/cohort feature table to quantify
  which xenobiotic metabolites are present and at what detection frequencies.
when_to_use_negative:
- Your metabolites of interest already have authentic standards and reference MS2
  spectra, so a standard targeted/library-search workflow applies instead
- The feature table is unprocessed raw LC-HRMS data — this skill expects an already-processed
  feature table matched by accurate m/z and retention time
- The cohort is small (<~10 samples), making >50% / 10-50% / <10% frequency stratification
  statistically uninformative
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: CMDN
  role: Produces the annotated, manually curated PMDB (N = 492 pesticide metabolites)
    that serves as the screening library; R-based framework with tidyverse, CluMSID,
    OrgMassSpecR, MSMSsim, msentropy, readxl dependencies
  repo: github.com/LeaveMeNotTonight/CMDN
- name: LC-HRMS
  role: Generates the raw urine-cohort data from which the processed feature table
    (accurate m/z, retention time per sample) is derived for PMDB matching
- name: TraceFinder
  role: Provides the detection criteria (nDATA workflow) used to mark each PMDB metabolite
    as detected or not detected in each of the 438 samples
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.est.5c08558
    title: Reaction-Guided Metabolomics Accelerates High-Throughput Annotation of
      Xenobiotic Metabolites for Human Exposome
  source_content_hash: sha256:5ee6ac172f6f5a12fe2eec3caf5096092f47df555e1cb4550e6b9af07848edb4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CMDN
  - LC-HRMS
  - TraceFinder
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# urine-cohort-exposome-annotation

## Summary

Screens a curated, CMDN-annotated pesticide metabolite database (PMDB, N = 492) against processed LC-HRMS feature tables from a 438-sample human urine cohort (BCP) using nDATA/TraceFinder detection criteria, then stratifies detected metabolites by sample detection frequency to map the internal exposome.

## When to use

Use when a curated metabolite database with accurate m/z and retention time must be screened against a large LC-HRMS urine/cohort feature table to quantify which xenobiotic metabolites are present and at what detection frequencies.

## When NOT to use

- Your metabolites of interest already have authentic standards and reference MS2 spectra, so a standard targeted/library-search workflow applies instead
- The feature table is unprocessed raw LC-HRMS data — this skill expects an already-processed feature table matched by accurate m/z and retention time
- The cohort is small (<~10 samples), making >50% / 10-50% / <10% frequency stratification statistically uninformative

## Inputs

- CMDN-annotated and manually curated PMDB (Table S2, sheet 1; N = 492 pesticide metabolites with accurate m/z and retention time)
- Processed LC-HRMS feature table from the 438 BCP urine cohort samples
- Per-sample metadata to compute per-subject distinct metabolite counts

## Outputs

- pmdb_bcp_screening_results.csv listing the 92 detected pesticide metabolites with high/moderate/low frequency-group labels (>50%, 10-50%, <10%)
- Per-subject summary of distinct metabolite identifications with cohort average (~11 identifications per subject)

## How to apply

Load the curated CMDN-annotated PMDB (Table S2, sheet 1; N = 492 manually curated pesticide metabolites with accurate m/z and retention time) and the processed LC-HRMS feature table from the 438 BCP urine cohort samples. Apply the nDATA/TraceFinder detection criteria to match PMDB entries against sample features, marking each metabolite as detected or not detected in every sample. Classify detected metabolites into frequency groups: high (>50% of samples), moderate (10-50%), and low (<10%); count distinct metabolite identifications per subject and compute the cohort average (~11 per subject). Rationale: a curated PMDB enables targeted screening of biotransformation-derived metabolites that traditional 'bottom-up' library-searching workflows miss, because they lack reference standards and spectra entries. Write the 92 detected metabolites with frequency-group labels and the per-subject summary to pmdb_bcp_screening_results.csv.

## Related tools

- **CMDN** (Produces the annotated, manually curated PMDB (N = 492 pesticide metabolites) that serves as the screening library; R-based framework with tidyverse, CluMSID, OrgMassSpecR, MSMSsim, msentropy, readxl dependencies) — github.com/LeaveMeNotTonight/CMDN
- **LC-HRMS** (Generates the raw urine-cohort data from which the processed feature table (accurate m/z, retention time per sample) is derived for PMDB matching)
- **TraceFinder** (Provides the detection criteria (nDATA workflow) used to mark each PMDB metabolite as detected or not detected in each of the 438 samples)

## Examples

```
write.csv(pmdb_bcp_screening_results, "pmdb_bcp_screening_results.csv")
```

## Evaluation signals

- The screen recovers exactly 92 detected metabolites from the 438-sample BCP cohort
- Frequency-group assignments partition cleanly into >50%, 10-50%, and <10% bins with no unassigned metabolites
- Mean distinct identifications per subject equals the reported value of approximately 11
- Every detected metabolite traces back to an entry in the manually curated PMDB (N = 492, Table S2 sheet 1) with valid accurate m/z and retention time
- Output CSV contains frequency-group labels and per-subject summary columns as specified

## Limitations

- Detection frequency stratification does not distinguish intensity-based exposure differences within a frequency group; MS intensities vary across the 438 samples
- Metabolites lacking curated accurate m/z and RT entries in the PMDB cannot be screened — annotation coverage depends on the enzymatic biotransformation assays (1,021 pesticides, 2,886 derivatives annotated)
- CMDN requires specific R packages (tidyverse, CluMSID, CluMSIDdata, OrgMassSpecR, pheatmap, reshape2, MSMSsim, msentropy, readxl) and compatible tools (MSDial ver. 4.80, Biotransformer 3.0)

## Evidence
