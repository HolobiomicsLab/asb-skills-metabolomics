---
name: metabolomics-raw-data-reprocessing
description: Use when raw LC–MS mzML files (e.g. from MetaboLights MTBLS1919) must
  be converted into a curated, QC-controlled feature matrix for untargeted metabolomics,
  rather than analyzed with manual or GUI tools.
when_to_use_negative:
- The input is already a preprocessed feature table — TidyMS in this paper starts
  from raw mzML data, and re-running detection/alignment on a matrix is out of scope
- Feature annotation is required — the comparative table marks feature annotation
  as missing in TidyMS
- The analysis targets MS/MS-based structural identification rather than QC-oriented
  preprocessing of MS1 feature tables
tools:
- name: TidyMS
  role: Core Python package performing chromatogram building, CWT peak picking, retention
    time alignment, deisotoping, adduct grouping, QC filtering, and LOESS batch correction
  repo: https://github.com/griquelme/tidyms
- name: Python
  role: Language runtime in which TidyMS pipelines are executed and scripted
- name: scikit-learn
  role: Downstream framework for state-of-the-art model generation on the TidyMS-processed
    feature matrix
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo10100416
    title: A Python-Based Pipeline for Preprocessing LC–MS Data for Untargeted Metabolomics
      Workflows
  source_content_hash: sha256:0ff32d5b596d42486dc8642d76058338404e547e0a4d9e49d536a19237b14e68
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/griquelme/tidyms
  tools:
  - Python
  - TidyMS
  - scikit-learn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# metabolomics-raw-data-reprocessing

## Summary

Reprocess raw LC–MS metabolomics data (mzML) with the TidyMS Python package, covering the full QC chain from chromatogram building and CWT-based peak detection to feature filtering, signal drift evaluation, and data curation on NIST plasma reference materials.

## When to use

Use when raw LC–MS mzML files (e.g. from MetaboLights MTBLS1919) must be converted into a curated, QC-controlled feature matrix for untargeted metabolomics, rather than analyzed with manual or GUI tools.

## When NOT to use

- The input is already a preprocessed feature table — TidyMS in this paper starts from raw mzML data, and re-running detection/alignment on a matrix is out of scope
- Feature annotation is required — the comparative table marks feature annotation as missing in TidyMS
- The analysis targets MS/MS-based structural identification rather than QC-oriented preprocessing of MS1 feature tables

## Inputs

- Raw LC–MS mzML data files (e.g. 31 raw files for Application 1 from MetaboLights MTBLS1919)
- Metadata distinguishing pooled/intrastudy QC injections, study samples (candidate RM 8231 and SRM 1950 plasma), and blanks (solvent, zero volume injection, process blanks)

## Outputs

- Extracted data matrix of features (retention time, m/z pairs) with peak areas after filtering
- Curation/QC report figures (e.g. extracted ion chromatograms for the five SSS compounds and suitability plots for pooled QC injections, system_suitability_report.png)
- Python objects compatible with scikit-learn for downstream statistical analysis

## How to apply

Download the raw mzML files (e.g. NIST candidate RM 8231 and SRM 1950 plasma batches from MetaboLights MTBLS1919) and use TidyMS to load each file, build chromatograms from consecutive MS scans, and detect peaks with the continuous wavelet transform (CWT) peak-picking algorithm parameterized for chromatographic data. Run the TidyMS preprocessing chain of retention time alignment, peak picking, deisotoping, integration, and grouping of adducts derived from the same compound. Curate the extracted data matrix with the paper's filter chain: an Rt filter discarding features with retention time below 90 s (system dead time ~0.8 min), a pooled/intrastudy QC presence filter enabling LOESS batch correction, a blank filter setting peak area to zero when it is 10-fold or less than the maximum blank area, an RSD filter removing features with RSD above 20% in pooled QC, a 100% intraclass prevalence filter with area threshold 5, and a D-ratio filter (MAD of QC relative to MAD of study samples) removing features with D-ratio ≥10%. The rationale is to retain only analytically robust (retention time, m/z) features while providing a rapid, reproducible, open Python-based workflow that integrates with scikit-learn for downstream modeling.

## Related tools

- **TidyMS** (Core Python package performing chromatogram building, CWT peak picking, retention time alignment, deisotoping, adduct grouping, QC filtering, and LOESS batch correction) — https://github.com/griquelme/tidyms
- **Python** (Language runtime in which TidyMS pipelines are executed and scripted)
- **scikit-learn** (Downstream framework for state-of-the-art model generation on the TidyMS-processed feature matrix)

## Evaluation signals

- Processing of the 31 raw data files for Application 1 completes on the order of the reported 66 s, with feature detection about 2 s per sample
- The output report contains EICs for all five SSS compounds (leucine enkephalin, lysophosphatidylcholine (0:0/18:0), L-phenylalanyl-L-phenylalanine, L-tryptophan, alogliptin) and a suitability plot for pooled QC injections matching Supplementary Figures S1–S2
- No features with retention time below 90 s (system dead time ~0.8 min) survive the Rt filter
- Remaining features show RSD ≤ 20% in pooled/intrastudy QC injections and D-ratio < 10%
- Features absent from pooled/intrastudy QC samples are discarded before LOESS batch correction is applied

## Limitations

- TidyMS lacks feature annotation capability (marked ❌ in the comparative tool table)
- No software version numbers are stated for TidyMS, Python, or scikit-learn in the paper
- No accession IDs are provided for NIST candidate RM 8231 or SRM 1950 beyond the MetaboLights dataset identifier MTBLS1919
- Filter thresholds (90 s Rt cutoff, 10-fold blank rule, 20% RSD, area threshold 5, 10% D-ratio) were optimized for this chromatographic setup and plasma dataset and may need re-tuning for other platforms

## Evidence

- [abstract] Abstract: "The capabilities of the package are shown with pipelines for an LC–MS system suitability check, system conditioning, signal drift evaluation, and data curation."
- [methods] Methods: "The procedure included retention time alignment, peak picking, deisotoping, integration, and grouping together adducts derived from the same compound."
- [results] Results: "The running time needed to analyze the 31 raw data files used to generate the results shown in Application 1 was 66 s."
