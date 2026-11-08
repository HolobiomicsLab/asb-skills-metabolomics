---
name: cholesterol-fraction-computation
description: Use when you have a post-QC NMR biomarker table (output of remove_technical_variation())
  filtered to the 107 non-derived biomarkers and need to derive the 76 additional
  ratios of biological significance, including cholesterol fractions.
when_to_use_negative:
- The input is the raw UK Biobank exported field data before technical variation removal
  — the README states no prefiltering of samples or columns should be performed prior
  to remove_technical_variation()
- You only need the 61 composite biomarkers and 81 original Nightingale ratios — use
  recompute_derived_biomarkers() instead of the extended-ratio functions
- The biomarker table already contains the extended ratios computed from directly-adjusted
  concentrations — these would not match ratios re-derived from adjusted component
  biomarkers
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: ukbnmr
  role: R package supplying compute_extended_ratios(), compute_lipoprotein_fatty_acids(),
    and the associated QC-flag aggregation functions used to derive the extended cholesterol
    and fatty acid ratios
  repo: https://github.com/sritchie73/ukbnmr
- name: R
  role: Execution environment for the ukbnmr workflow
- name: data.table
  role: Provides fread()/fwrite() for fast reading of exported field data and writing
    the final recomputed biomarker CSV
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41597-023-01949-y
    title: Quality control and removal of technical variation of NMR metabolic biomarker
      data in ~120,000 UK Biobank participants
  source_content_hash: sha256:3912c3db5e0e05786040fd4f67f9cc1a921abbd38f8deba901abd04f5eb93514
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sritchie73/ukbnmr
  tools:
  - R
  - data.table
  - ukbnmr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# cholesterol-fraction-computation

## Summary

Computes extended biomarker ratios — including free/esterified cholesterol fractions and lipoprotein fatty acid fractions — from post-technical-variation UK Biobank NMR metabolomics data using the ukbnmr R package's compute_extended_ratios() and compute_lipoprotein_fatty_acids() functions.

## When to use

Use when you have a post-QC NMR biomarker table (output of remove_technical_variation()) filtered to the 107 non-derived biomarkers and need to derive the 76 additional ratios of biological significance, including cholesterol fractions.

## When NOT to use

- The input is the raw UK Biobank exported field data before technical variation removal — the README states no prefiltering of samples or columns should be performed prior to remove_technical_variation()
- You only need the 61 composite biomarkers and 81 original Nightingale ratios — use recompute_derived_biomarkers() instead of the extended-ratio functions
- The biomarker table already contains the extended ratios computed from directly-adjusted concentrations — these would not match ratios re-derived from adjusted component biomarkers

## Inputs

- data.table/data.frame of post-QC NMR biomarker concentrations from remove_technical_variation() with eid and visit_index identifier columns
- nmr_info data.frame bundled with ukbnmr (biomarker names, UK Biobank field mappings, derived/non-derived Type labels)
- data.frame of post-QC biomarker QC flags for the underlying non-derived biomarkers

## Outputs

- data.frame of 76 extended biomarker ratios (free/esterified cholesterol fractions, lipoprotein fatty acid fractions, omega-3/omega-6 percentages of polyunsaturated fatty acids)
- aggregated QC flag table for the extended ratios (compute_extended_ratio_qc_flags, compute_lipoprotein_fatty_acids_flags)
- final CSV of 325 biomarkers written via data.table::fwrite(), e.g. nmr_biomarkers_recomputed.csv

## How to apply

First run remove_technical_variation() to obtain adjusted biomarker concentrations, filter to the 107 biomarkers that cannot be derived (Type == "Non-derived" in the bundled nmr_info data.frame), and recompute the 61 composite biomarkers and 81 original ratios with recompute_derived_biomarkers(). Then call compute_extended_ratios() to derive the 76 additional ratios of potential biological significance — including serum free/esterified cholesterol fractions and omega-3/omega-6 percentages of polyunsaturated fatty acids — and compute_lipoprotein_fatty_acids() for the fatty acid fractions of the 3 lipoprotein classes and serum. Aggregate the QC flags of the underlying biomarkers for each derived ratio using compute_extended_ratio_qc_flags() and compute_lipoprotein_fatty_acids_flags(). Rationale: biomarker adjustment can introduce new unwanted variation into ratios and composites, so ratios must be recomputed from adjusted non-derived parts rather than adjusted directly. Finally write the combined table of 325 biomarkers (107 non-derived + 61 composites + 81 original ratios + 76 extended ratios) with data.table::fwrite().

## Related tools

- **ukbnmr** (R package supplying compute_extended_ratios(), compute_lipoprotein_fatty_acids(), and the associated QC-flag aggregation functions used to derive the extended cholesterol and fatty acid ratios) — https://github.com/sritchie73/ukbnmr
- **R** (Execution environment for the ukbnmr workflow)
- **data.table** (Provides fread()/fwrite() for fast reading of exported field data and writing the final recomputed biomarker CSV)

## Examples

```
library(ukbnmr); extended <- compute_extended_ratios(biomarkers); fa <- compute_lipoprotein_fatty_acids(biomarkers); flags <- compute_extended_ratio_qc_flags(biomarker_qc_flags); fwrite(extended, "nmr_extended_ratios.csv")
```

## Evaluation signals

- Output file nmr_biomarkers_recomputed.csv exists, is a valid CSV, and contains the eid and visit_index columns
- The table contains exactly 325 biomarker columns (107 non-derived + 61 composites + 81 original ratios + 76 extended ratios)
- All concentration values are non-negative
- Ratio columns match their defining quotients (A/B) recomputed from the component biomarker columns
- QC flags for each extended ratio correctly aggregate the flags of its underlying non-derived biomarkers

## Limitations

- Biomarker adjustment can introduce new unwanted variation into biomarkers which are ratios, percentages, or otherwise composed of multiple other biomarkers — extended ratios are only as good as the adjusted components
- Many biomarkers have extreme outliers and extremely non-normal distributions requiring careful treatment in downstream modelling
- The remove_technical_variation() step takes a little over an hour and requires at least 32 GB of RAM

## Evidence

- [readme] Extended ratios derived via ukbnmr functions: "An additional 76 biomarker ratios of potential biological significance are computed."
- [methods] Filtering to non-derived biomarkers: "First biomarker data is filtered to the 107 biomarkers that cannot be derived from any combination of other biomarkers."
- [methods] Recomputation rationale: "We provide an additional function, `recompute_derived_biomarkers()` to recompute all composite biomarkers and ratios from 107 non-derived biomarkers"
- [discussion] Adjustment can harm derived biomarkers: "biomarker adjustment can introduce new unwanted variation into biomarkers which are ratios, percentages, or otherwise composed of multiple other biomarkers"
