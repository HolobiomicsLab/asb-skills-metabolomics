---
name: ukbiobank-nmr-qc-flag-aggregation
description: Use when post-QC NMR biomarker tables have had the 61 composites and
  81 ratios re-derived from the 107 non-derived biomarkers and derived measurements
  need QC flags propagated from their component biomarkers.
when_to_use_negative:
- When working with the raw UK Biobank field exports before remove_technical_variation()
  has been run — derived biomarkers in the raw release are Nightingale's originals,
  not recomputed, so flag aggregation would double-count
- When only the 107 non-derived biomarkers are analyzed — these retain their directly
  extracted measurement QC flags and need no aggregation
- When no sensitivity analysis to QC-flagged samples or measurements is planned and
  flags will not inform inclusion/exclusion decisions
edam_topics: []
tools:
- name: ukbnmr
  role: R package supplying recompute_derived_biomarkers(), recompute_derived_biomarker_qc_flags(),
    compute_extended_ratio_qc_flags(), and compute_lipoprotein_fatty_acids_flags()
    used to re-derive biomarkers and aggregate their QC flags
  repo: https://github.com/sritchie73/ukbnmr
- name: data.table
  role: Used with fread()/fwrite() to read exported UK Biobank field data and write
    the final recomputed biomarker and QC flag CSV files
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41597-023-01949-y
    title: Quality control and removal of technical variation of NMR metabolic biomarker
      data in ~120,000 UK Biobank participants
  source_content_hash: sha256:3248f7674b06555d5469afae63028d02aa169fa8de02824e12147f7c4a7c704e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sritchie73/ukbnmr
  tools:
  - R
  - data.table
  - ukbnmr
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# ukbiobank-nmr-qc-flag-aggregation

## Summary

Aggregates the UK Biobank NMR biomarker measurement QC flags of underlying non-derived biomarkers into QC flags for each recomputed composite biomarker, biomarker ratio, and extended ratio after removal of technical variation.

## When to use

Use when post-QC NMR biomarker tables have had the 61 composites and 81 ratios re-derived from the 107 non-derived biomarkers and derived measurements need QC flags propagated from their component biomarkers.

## When NOT to use

- When working with the raw UK Biobank field exports before remove_technical_variation() has been run — derived biomarkers in the raw release are Nightingale's originals, not recomputed, so flag aggregation would double-count
- When only the 107 non-derived biomarkers are analyzed — these retain their directly extracted measurement QC flags and need no aggregation
- When no sensitivity analysis to QC-flagged samples or measurements is planned and flags will not inform inclusion/exclusion decisions

## Inputs

- post-QC biomarker QC flags data.frame from remove_technical_variation() (one column per NMR biomarker, keyed by eid and visit_index)
- post-QC adjusted biomarker concentrations data.frame (107 non-derived biomarkers plus eid and visit_index)
- nmr_info data.frame bundled with the ukbnmr package (biomarker metadata including derived/non-derived Type)

## Outputs

- aggregated QC flag data.frame covering all 325 biomarkers (61 composites, 81 original ratios, 76 extended ratios, plus the 107 non-derived)
- final biomarker table written to nmr_biomarkers_recomputed.csv with 325 biomarker columns plus eid and visit_index

## How to apply

After running remove_technical_variation() from the ukbnmr R package, load the adjusted biomarker and biomarker_qc_flags data.frames. Filter to the 107 biomarkers that cannot be derived (Type == "Non-derived" in nmr_info), recompute the 61 composite biomarkers and 81 original ratios with recompute_derived_biomarkers(), and compute the 76 extended ratios (lipoprotein fatty acid fractions, cholesterol fractions, omega-3/omega-6 percentages) with compute_extended_ratios() and compute_lipoprotein_fatty_acids(). Because adjusting component biomarkers separately can introduce new unwanted variation into ratios and composites, the derived measurements' QC flags must be aggregated from the QC flags of their underlying biomarkers using recompute_derived_biomarker_qc_flags() plus compute_extended_ratio_qc_flags() and compute_lipoprotein_fatty_acids_flags(). This yields a QC flag table aligned to the final 325-biomarker table (107 non-derived + 61 composites + 81 original ratios + 76 extended ratios) keyed by eid and visit_index.

## Related tools

- **ukbnmr** (R package supplying recompute_derived_biomarkers(), recompute_derived_biomarker_qc_flags(), compute_extended_ratio_qc_flags(), and compute_lipoprotein_fatty_acids_flags() used to re-derive biomarkers and aggregate their QC flags) — https://github.com/sritchie73/ukbnmr
- **data.table** (Used with fread()/fwrite() to read exported UK Biobank field data and write the final recomputed biomarker and QC flag CSV files)

## Examples

```
library(ukbnmr); flags <- recompute_derived_biomarker_qc_flags(processed$biomarker_qc_flags); fwrite(flags, file="nmr_derived_qc_flags.csv")
```

## Evaluation signals

- Output CSV exists, is a valid CSV readable with data.table::fread(), and contains the eid and visit_index identifier columns
- Table contains exactly 325 biomarker columns: 107 non-derived + 61 composite biomarkers + 81 original ratios + 76 extended ratios
- Derived biomarker QC flags match the aggregated flags of their underlying non-derived biomarkers (e.g. a ratio is flagged if any component measurement is flagged)
- Concentrations are non-negative and each ratio column equals its defining quotient (A/B) of the adjusted component biomarkers
- Every row is uniquely identifiable by its combination of eid and visit_index

## Limitations

- No filtering or treatment of outlier samples is performed by the ukbnmr R package when removing technical variation; sensitivity analysis to inclusion/exclusion of QC-flagged samples and measurements is recommended
- The QC procedure relies on regressing out technical covariates rather than calibration to quality control samples, and may remove biological variation of interest; flag aggregation inherits these caveats
- Only a small fraction of samples (2.6%) could be assessed for reproducibility via blind duplicates, so QC-flag accuracy has limited empirical validation

## Evidence

- [methods] Recompute derived biomarkers then aggregate QC flags: "We provide an additional function, `recompute_derived_biomarkers()` to recompute all composite biomarkers and ratios from 107 non-derived biomarkers"
- [discussion] Rationale for re-deriving after adjustment: "biomarker adjustment can introduce new unwanted variation into biomarkers which are ratios, percentages, or otherwise composed of multiple other biomarkers"
- [discussion] Filter to non-derived biomarkers: "we recommend filtering to the 107 non-derived biomarkers and re-deriving the rest post-adjustment"
- [readme] Recomputation counts from algorithm: "The 61 composite biomarkers and 81 biomarker ratios are recomputed from their adjusted parts."
- [readme] Extended ratios count: "An additional 76 biomarker ratios of potential biological significance are computed."
- [discussion] No outlier filtering caveat: "no filtering or treatment of outlier samples is performed by the ukbnmr R package when removing technical variation."
