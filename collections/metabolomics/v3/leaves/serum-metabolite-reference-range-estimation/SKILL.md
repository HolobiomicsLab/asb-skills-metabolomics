---
name: serum-metabolite-reference-range-estimation
description: Use when large-scale serum MSI-CE-MS metabolomic data (e.g. >5,000 samples
  plus QC runs) have been preprocessed into peak-area tables and you need per-metabolite
  reference ranges for a nutritional survey cohort.
when_to_use_negative:
- Peak areas have not yet been generated — raw mzML files still need automated PeakMeister
  preprocessing, peak annotation, and integration before reference ranges can be computed.
- Only a single batch or small QC set is available (e.g. 468 NIST SRM-1950 plasma
  samples) without the large national survey cohort of ~5,000 serum samples needed
  for population reference interval estimation.
- The metabolite of interest lacks an available chemical standard or is a late-migrating
  compound without flanking/stable-isotope internal standards, so peak annotation
  fidelity is not assured.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0218
- http://edamontology.org/topic_3570
tools:
- name: PeakMeister
  role: 'R-based automated preprocessing of targeted full-scan MSI-CE-MS mzML files:
    computes migration indexes from 11 internal standards to predict migration times,
    annotates and integrates metabolite peaks, and emits peak-area tables used to
    derive the reference ranges.'
  repo: https://github.com/PBM-Group/PeakMeister
- name: R
  role: Statistical environment in which PeakMeister runs and in which the per-metabolite
    2.5th/97.5th percentile and median reference ranges are computed from the peak-area
    distributions.
- name: MSI-CE-MS
  role: High-throughput separation platform (multisegment injection-capillary electrophoresis-mass
    spectrometry, <4 min/sample) that acquired the serum and QC data across multiple
    batches from which peak areas are extracted.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c03513
    title: A Software Tool for Rapid and Automated Preprocessing of Large-Scale Serum
      Metabolomic Data by Multisegment Injection-Capillary Electrophoresis-Mass Spectrometry
  source_content_hash: sha256:88aa34ab59e9698962f7d5be57810e0248398dc2cec4ed76252768f8223f19e2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PBM-Group/PeakMeister
  repo_url: https://github.com/PBM-Group/PeakMeister
  tools:
  - MSI-CE-MS
  - PeakMeister
  - R
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0218
  - http://edamontology.org/topic_3570
  provenance_tier: literature
  techniques:
  - CE-MS
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# serum-metabolite-reference-range-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimate population reference intervals (2.5th–97.5th percentile limits plus median) for serum metabolites quantified by MSI-CE-MS across a large national nutritional survey cohort, using automated PeakMeister preprocessing of targeted full-scan mzML data.

## When to use

Use when large-scale serum MSI-CE-MS metabolomic data (e.g. >5,000 samples plus QC runs) have been preprocessed into peak-area tables and you need per-metabolite reference ranges for a nutritional survey cohort.

## When NOT to use

- Peak areas have not yet been generated — raw mzML files still need automated PeakMeister preprocessing, peak annotation, and integration before reference ranges can be computed.
- Only a single batch or small QC set is available (e.g. 468 NIST SRM-1950 plasma samples) without the large national survey cohort of ~5,000 serum samples needed for population reference interval estimation.
- The metabolite of interest lacks an available chemical standard or is a late-migrating compound without flanking/stable-isotope internal standards, so peak annotation fidelity is not assured.

## Inputs

- PeakMeister output tables: peak areas per sample per metabolite, migration indexes, and saved processing parameters (R environment)
- Raw mzML files from targeted full-scan MSI-CE-MS runs of serum samples and QC samples
- PeakMeister 'Mass List and Parameters.xlsx' template (Sheet 1: Mass List with extraction.window.ppm, snr.threshold, migration.window.seconds; Sheet 2: Reference Mass List with min.mt.min/max.mt.min; Sheet 3: Parameters with number.of.injections and lock-mass settings)
- Cohort metadata for survey stratification (age, region of Brazil; e.g. ENANI-2019 participant records)

## Outputs

- reference_ranges_table_s2.csv — per-metabolite table of 2.5th percentile, median, and 97.5th percentile for the 40 serum metabolites with available chemical standards
- Verification report comparing computed reference ranges to the article's Table S2 (output_matches_reference / expert_review)
- Record of excluded outlier data file(s) with misannotated peaks

## How to apply

First pre-process all mzML files with PeakMeister (R) using a targeted mass list with ~30–35 ppm extraction.window.ppm and migration.window.seconds windows, a migration time index built from 11 internal standards, and snr.threshold so signals below the limit are recorded as '<LOD'; verify per-run annotation by inspecting the extracted ion electropherogram plots and remove any outlier data file where annotation fails (e.g. 73 peaks misannotated owing to a low ornithine-15N internal standard signal). Then load the PeakMeister output tables (peak areas, migration indexes, and saved processing parameters), subset peak areas to the target age stratum (children 6 months to 5 years) and stratify by survey region of Brazil to respect the ENANI-2019 national survey design, and compute the 2.5th and 97.5th percentile limits and the median of the peak-area distribution for each of the 40 metabolites with available chemical standards. Save the resulting 40-metabolite table as reference_ranges_table_s2.csv and check it against the article's Table S2 for reproducibility.

## Related tools

- **PeakMeister** (R-based automated preprocessing of targeted full-scan MSI-CE-MS mzML files: computes migration indexes from 11 internal standards to predict migration times, annotates and integrates metabolite peaks, and emits peak-area tables used to derive the reference ranges.) — https://github.com/PBM-Group/PeakMeister
- **R** (Statistical environment in which PeakMeister runs and in which the per-metabolite 2.5th/97.5th percentile and median reference ranges are computed from the peak-area distributions.)
- **MSI-CE-MS** (High-throughput separation platform (multisegment injection-capillary electrophoresis-mass spectrometry, <4 min/sample) that acquired the serum and QC data across multiple batches from which peak areas are extracted.)

## Examples

```
Rscript PeakMeister.R  # after filling 'Mass List and Parameters.xlsx' and placing mzML files in 'mzML Files'; then compute percentiles in R from the output peak-area table to produce reference_ranges_table_s2.csv
```

## Evaluation signals

- Computed reference ranges must match the values reported in the article's Table S2 (output_matches_reference / expert_review).
- Peak annotation fidelity must be consistent with the reported median accuracy >99.9% for peak annotation.
- After removing the single outlier data file, annotation quality per run should be verified by inspecting extracted ion electropherogram plots for proper peak annotation and integration.
- Reference intervals must be computed for exactly the 40 serum metabolites with available chemical standards from the ~5,000 ENANI-2019 participants, not a different subset.
- Signals falling below the snr.threshold must be recorded as '<LOD' rather than as quantifiable areas.

## Limitations

- Late-migrating serum metabolites lacking two flanking or a stable-isotope internal standard were prone to peak misannotation, which would bias the derived reference ranges.
- Low abundance metabolites with signals near the method detection limit were also prone to peak misannotation.
- A single outlier data file can produce widespread misannotation (73 peaks) when an internal standard signal is low (e.g. ornithine-15N), and must be identified and excluded.
- PeakMeister has not yet been validated for sample matrices other than serum (e.g. urine, saliva, stool).
- Real-time data preprocessing output for review was not yet implemented at publication time; preprocessing is performed retrospectively.

## Evidence

- [methods] Preprocessing parameters and outlier exclusion from the task workflow: [evidence span withheld — anchor status: not_found]
