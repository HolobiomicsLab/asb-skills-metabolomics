---
name: qc-sample-based-data-curation
description: Use when an extracted LC–MS feature table (e.g. from mzML via pyopenms)
  must be cleaned with pooled/intrastudy QC and blank samples before PCA or other
  statistical analysis.
when_to_use_negative:
- The raw spectra still need feature detection/correspondence (retention time alignment,
  peak picking, deisotoping) and no extracted feature table exists yet
- The study lacks pooled/intrastudy QC injections or blank samples, so the QC-presence,
  RSD, D-ratio, and blank filters cannot be computed
- The goal is metabolite feature annotation — TidyMS lacks feature annotation capability
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: TidyMS
  role: Python package providing the data container and QC curation filters (Rt, QC-presence,
    blank, RSD, prevalence, D-ratio)
  repo: https://github.com/griquelme/tidyms
- name: pyopenms
  role: Reads the raw mzML files from the MetaboLights deposit into the pipeline
- name: scikit-learn
  role: Builds PCA score plots used to validate that curation tightened pooled QC
    clustering
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo10100416
    title: A Python-Based Pipeline for Preprocessing LC–MS Data for Untargeted Metabolomics
      Workflows
  source_content_hash: sha256:ddeac674cb589004c13cbe4088e03a4980f365f6ab4f028a834977b20367562b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/griquelme/tidyms
  tools:
  - TidyMS
  - pyopenms
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# qc-sample-based-data-curation

## Summary

Sequential QC-driven curation of an extracted LC–MS feature matrix (from TidyMS/pyopenms on mzML data) using pooled/intrastudy QC samples and blanks to retain only analytically robust features. Yields a cleaned data matrix (e.g. 665 features for NIST candidate RM 8231 and SRM 1950) ready for statistical analysis.

## When to use

Use when an extracted LC–MS feature table (e.g. from mzML via pyopenms) must be cleaned with pooled/intrastudy QC and blank samples before PCA or other statistical analysis.

## When NOT to use

- The raw spectra still need feature detection/correspondence (retention time alignment, peak picking, deisotoping) and no extracted feature table exists yet
- The study lacks pooled/intrastudy QC injections or blank samples, so the QC-presence, RSD, D-ratio, and blank filters cannot be computed
- The goal is metabolite feature annotation — TidyMS lacks feature annotation capability

## Inputs

- Raw LC–MS data in mzML format (e.g. from MetaboLights MTBLS1919) read with pyopenms
- Extracted feature table (peak areas per sample, after retention time alignment, peak picking, deisotoping and adduct grouping)
- Sample-class metadata distinguishing study samples (NIST candidate RM 8231 and SRM 1950 plasma), pooled/intrastudy QC injections, and solvent/zero-volume-injection/process blanks

## Outputs

- Curated data matrix of surviving metabolic features (665 features for the NIST candidate RM 8231 / SRM 1950 case study)
- Per-filter statistics (feature counts and values removed at each curation step)
- PCA score plots built with scikit-learn to monitor the effect of curation on pooled QC samples

## How to apply

Load the raw mzML files and the extracted feature table (e.g. from the MetaboLights MTBLS1919 deposit) into the TidyMS Data Container built on Pandas, then apply the filters in order: (1) an Rt filter discarding features eluting below 90 s, since the system dead time was ~0.8 min; (2) removal of features absent from pooled/intrastudy QC samples, which is a prerequisite for LOESS batch correction; (3) a blank-based filter that sets a feature's peak area to zero in a plasma sample if it is 10-fold or less than the maximum peak area in the solvent, zero-volume-injection, and process blanks; (4) an RSD filter eliminating features with RSD > 20% across pooled/intrastudy QC injections; (5) a 100% intraclass prevalence filter with area threshold 5, zeroing values below it; and (6) a D-ratio filter removing features whose D-ratio (MAD of pooled/intrastudy QC relative to MAD of all study samples) is 10% or higher. The rationale is to keep only features robust enough for downstream analysis; monitor each step with filter statistics and PCA score plots (scikit-learn) to confirm pooled QC samples cluster tightly after curation.

## Related tools

- **TidyMS** (Python package providing the data container and QC curation filters (Rt, QC-presence, blank, RSD, prevalence, D-ratio)) — https://github.com/griquelme/tidyms
- **pyopenms** (Reads the raw mzML files from the MetaboLights deposit into the pipeline)
- **scikit-learn** (Builds PCA score plots used to validate that curation tightened pooled QC clustering)

## Evaluation signals

- Every retained feature passes all criteria: Rt >= 90 s, sample/blank peak area ratio > 10x, pooled/intrastudy QC RSD <= 20%, 100% intraclass prevalence at area threshold 5, and D-ratio < 10%
- The pipeline's filter statistics (features removed at each step) account exactly for the drop from the initial feature count to the final curated matrix (665 features in the case study)
- Post-curation PCA score plot shows pooled/intrastudy QC samples clustering tightly with reduced analytical drift
- All peak areas below the prevalence threshold value of 5 are set to zero in the output matrix
- Running time remains rapid/automatable (TidyMS analyzed 31 raw data files in 66 s in the companion application)

## Limitations

- TidyMS lacks feature annotation capability, so the curated matrix is feature-level (m/z, retention time) only
- Thresholds are study- and instrument-specific: the 90 s Rt cutoff derives from this system's ~0.8 min dead time and must be re-derived for other setups
- No software version numbers for TidyMS, Python, or scikit-learn are stated in the article, which may complicate exact reproduction
- The D-ratio is defined relative to the MAD of all study samples, so its value depends on the composition of the full sample set

## Evidence

- [results] The first step was implemented to discard features with retention time values lower than 90 s, as the system dead time was approximately 0.8 min.: "The first step was implemented to discard features with retention time values lower than 90 s, as the system dead time was approximately 0.8 min."
- [results] RSD filter threshold: "all features with an RSD larger than 20% in pooled /intrastudy QC sample injections were eliminated."
- [results] D-ratio definition: "estimated as the MAD of pooled /intrastudy QC samples relative to the MAD of all study samples."
- [methods] Data source: "The mass spectrometry data have been deposited to the Metabo Lights public repository [38] with the data set identifier MTBLS1919."
- [discussion] Downstream integration: "The implementation in Python allows easy integration of results into state-of-the-art tools for model generation, such as scikit-learn"
- [abstract] Scope of package: "The capabilities of the package are shown with pipelines for an LC–MS system suitability check, system conditioning, signal drift evaluation, and data curation."
