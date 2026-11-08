---
name: sample-quality-control-outlier-detection
description: Use when a lipidomics experiment (e.g. a LipidomicsExperiment from Metabolomics
  Workbench or Skyline export) needs QC screening before downstream analysis and suspect
  samples must be identified and removed.
when_to_use_negative:
- The dataset is not a lipidomics experiment (lipidr's mva/PCA operates on lipid peak-area
  data, not general omics matrices).
- Outlier status must be determined from %CV of measured lipids rather than sample
  dispersion — lipidr supports filtering lipids by %CV, which is a different QC operation.
- You need lipid-level outlier detection rather than sample-level outlier detection;
  the workflow here identifies and removes whole samples.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: lipidr
  role: R/Bioconductor package whose mva(measure='Area', method='PCA') function computes
    the PCA and whose interactive plotting displays scores plots for sample QC and
    outlier identification
  repo: github.com/ahmohamed/lipidr
- name: R/Bioconductor
  role: Runtime environment in which the LipidomicsExperiment is loaded, PCA is run,
    and the cleaned dataset is saved as RDS
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00082
    title: 'lipidr: A Software Tool for Data Mining and Analysis of Lipidomics Datasets'
  source_content_hash: sha256:ea664860dfde0b97ac5729c2d8025100b97478ef78c48b48146944841eae6196
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - R/Bioconductor
  - lipidr
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# sample-quality-control-outlier-detection

## Summary

Detect and remove outlier samples in lipidomics datasets by running PCA via lipidr's multivariate analysis and inspecting the interactive scores plot for samples with significantly large dispersion, as demonstrated on Metabolomics Workbench study ST001111.

## When to use

Use when a lipidomics experiment (e.g. a LipidomicsExperiment from Metabolomics Workbench or Skyline export) needs QC screening before downstream analysis and suspect samples must be identified and removed.

## When NOT to use

- The dataset is not a lipidomics experiment (lipidr's mva/PCA operates on lipid peak-area data, not general omics matrices).
- Outlier status must be determined from %CV of measured lipids rather than sample dispersion — lipidr supports filtering lipids by %CV, which is a different QC operation.
- You need lipid-level outlier detection rather than sample-level outlier detection; the workflow here identifies and removes whole samples.

## Inputs

- LipidomicsExperiment object imported into R (e.g. from Metabolomics Workbench study ST001111, measure='Area', after sample name correction)
- Sample annotations (e.g. SampleType) added via add_sample_annotation() for coloring score plots

## Outputs

- PCA scores plot (interactive, colored by SampleType) identifying outlier samples
- PCA scores figure saved as figure/*.png
- Cleaned LipidomicsExperiment with outlier samples removed, saved as cleaned_st001111.rds

## How to apply

Load the imported, name-corrected LipidomicsExperiment (for ST001111, built from Metabolomics Workbench files with measure='Area') into R. Run lipidr::mva() with method='PCA' on the peak-area matrix to compute principal components. Display the PCA score plot with points colored by SampleType using lipidr's interactive plotting, and inspect the plot for points with significantly large dispersion from the main cluster. In the ST001111 breast-cancer tissue example, samples 42 and 18 were flagged this way and removed. Rationale: lipidr generates PCA score plots explicitly for 'quality control of samples and measured lipids', so outlier removal is done prior to univariate/multivariate analysis. Save the PCA scores figure (figure/*.png) and the cleaned dataset (cleaned_st001111.rds) after removal.

## Related tools

- **lipidr** (R/Bioconductor package whose mva(measure='Area', method='PCA') function computes the PCA and whose interactive plotting displays scores plots for sample QC and outlier identification) — github.com/ahmohamed/lipidr
- **R/Bioconductor** (Runtime environment in which the LipidomicsExperiment is loaded, PCA is run, and the cleaned dataset is saved as RDS)

## Examples

```
d <- lipidr::mva(cleaned_st001111, measure="Area", method="PCA")
```

## Evaluation signals

- The PCA score plot renders with points colored by SampleType, confirming sample annotations were attached to the LipidomicsExperiment.
- Outlier samples (in ST001111, samples 42 and 18) are visibly separated with significantly large dispersion from the main cluster in the scores plot.
- The cleaned experiment no longer contains the flagged samples, and persists as a valid .rds object (cleaned_st001111.rds).
- A PCA scores figure is written to figure/*.png.
- Downstream analyses are run on the post-removal dataset, matching the paper's workflow ordering of QC before univariate/multivariate analysis.

## Limitations

- Outlier identification relies on visual inspection of the interactive scores plot; the workflow does not specify a numeric dispersion threshold.
- Outlier samples must be removed before further analysis — the skill addresses sample removal only, not lipid-level QC such as %CV filtering or normalization.
- This workflow is specific to lipidomics data handled as LipidomicsExperiment objects (from Skyline exports, numerical matrices, or Metabolomics Workbench files).

## Evidence

- [results] two outlier samples with significantly large dispersion (samples `42` and `18`) were identified and removed: [evidence span withheld — anchor status: not_found]
- [results] Run lipidr::mva(measure="Area", method="PCA"): [evidence span withheld — anchor status: not_found]
- [readme] lipidr implements PCA, PCoA and OPLS(DA) to reveal patterns in data and discover variables related to an outcome of interest.: [evidence span withheld — anchor status: external_doc]
