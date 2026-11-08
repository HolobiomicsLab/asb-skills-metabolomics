---
name: chromatographic-eic-extraction
description: Use when raw LC–MS mzML files must yield EICs for specific target compounds
  — e.g., extracting chromatograms for the five system suitability solution standards
  (leucine enkephaline, lysophosphatidylcholine (0:0/18:0), L-phenylalanyl-L-phenylalanine,
  L-tryptophan, alogliptin) from QC injections.
when_to_use_negative:
- Input is already an extracted feature table or data matrix — EIC extraction requires
  raw mzML chromatographic data
- The goal is untargeted feature annotation across all detected features — TidyMS
  lacks feature annotation capability and this skill targets known compounds only
- Data are non-chromatographic (e.g., direct-infusion MS), where consecutive-scan
  chromatogram building and CWT peak picking are not applicable
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: TidyMS
  role: Loads mzML files, builds chromatograms from consecutive MS scans, performs
    CWT-based peak picking, and extracts EICs for the target compounds
  repo: https://github.com/griquelme/tidyms
- name: Python
  role: Programming language in which the TidyMS EIC extraction and plotting pipeline
    is implemented
  repo: https://github.com/griquelme/tidyms-notebooks
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo10100416
    title: A Python-Based Pipeline for Preprocessing LC–MS Data for Untargeted Metabolomics
      Workflows
  source_content_hash: sha256:569763b3c7d8454afe36030ed9b2fdf195bf9abbb62492ab8b787186ead2cf7c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/griquelme/tidyms
  tools:
  - Python
  - TidyMS
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# chromatographic-eic-extraction

## Summary

Targeted extraction of extracted ion chromatograms (EICs) for known compounds from raw LC–MS mzML data using TidyMS, building chromatograms from consecutive MS scans and detecting peaks with the CWT-based algorithm. It is the core step for system suitability checks where marker compounds (e.g., the five SSS standards) must be located and plotted across QC injections.

## When to use

Use when raw LC–MS mzML files must yield EICs for specific target compounds — e.g., extracting chromatograms for the five system suitability solution standards (leucine enkephaline, lysophosphatidylcholine (0:0/18:0), L-phenylalanyl-L-phenylalanine, L-tryptophan, alogliptin) from QC injections.

## When NOT to use

- Input is already an extracted feature table or data matrix — EIC extraction requires raw mzML chromatographic data
- The goal is untargeted feature annotation across all detected features — TidyMS lacks feature annotation capability and this skill targets known compounds only
- Data are non-chromatographic (e.g., direct-infusion MS), where consecutive-scan chromatogram building and CWT peak picking are not applicable

## Inputs

- Raw LC–MS mzML data files (31 files from candidate RM 8231 / SRM 1950 plasma batches, MetaboLights MTBLS1919)
- List of target compounds with known retention time and m/z values (the five SSS standards)
- Sample metadata identifying system suitability QC injections and pooled/intrastudy QC injections

## Outputs

- EIC plots for each SSS standard compound across QC injections (content matching Supplementary Figure S1)
- Suitability plot of retention-time and m/z stability of the markers in pooled QC injections (matching Supplementary Figure S2)
- Combined report image (system_suitability_report.png)

## How to apply

Load each mzML raw file with TidyMS and build chromatograms from consecutive MS scans, then detect peaks using the CWT-based peak-picking algorithm with parameters optimized for chromatographic data. Extract EICs for each target compound's exact retention time, m/z pair — in the NIST RM 8231 / SRM 1950 application, the five SSS standard compounds leucine enkephaline, lysophosphatidylcholine (0:0/18:0), L-phenylalanyl-L-phenylalanine, L-tryptophan, and alogliptin — from the system suitability QC injections analyzed before platform conditioning and at the end of the batch. Render the EIC plots per compound and per injection (reproducing Supplementary Figure S1) and, for pooled/intrastudy QC injections, plot retention-time and m/z stability of the suitability markers (Supplementary Figure S2). Save the combined figures as a single report (e.g., system_suitability_report.png) so all five compounds are visually confirmed present across the run.

## Related tools

- **TidyMS** (Loads mzML files, builds chromatograms from consecutive MS scans, performs CWT-based peak picking, and extracts EICs for the target compounds) — https://github.com/griquelme/tidyms
- **Python** (Programming language in which the TidyMS EIC extraction and plotting pipeline is implemented) — https://github.com/griquelme/tidyms-notebooks

## Evaluation signals

- The rendered report contains EICs for all five SSS compounds (leucine enkephaline, lysophosphatidylcholine (0:0/18:0), L-phenylalanyl-L-phenylalanine, L-tryptophan, alogliptin), matching the content of Figures S1–S2
- Detected peaks for each marker appear at stable retention times and m/z values across the run (before platform conditioning and at end of batch, and in pooled QC injections)
- Feature detection completes in about 2 seconds per sample, consistent with the article's reported per-sample runtime
- Processing of all 31 raw data files completes on the order of the reported ~66 s runtime
- Chromatograms are built from consecutive MS scans and peaks are picked with the CWT-based algorithm, so each detected peak has an integrated chromatographic profile rather than a single-scan intensity

## Limitations

- Feature annotation is not provided by TidyMS, so extracted EICs must be matched to targets using externally supplied retention time / m/z pairs
- No software version numbers are stated for TidyMS or Python, so reproducing exact peak-picking behavior may require pinning versions manually
- Reported runtimes (~2 s per sample for feature detection; 66 s for 31 files) are hardware-dependent and serve only as order-of-magnitude checks

## Evidence

- [abstract] The present work introduces TidyMS, a package for the Python programming language for preprocessing LC–MS data for quality control (QC) procedures in untargeted metabolomics workflows.: "The present work introduces TidyMS, a package for the Python programming language for preprocessing LC–MS data for quality control (QC) procedures in untargeted metabolomics workflows."
- [results] The running time needed to analyze the 31 raw data files used to generate the results shown in Application 1 was 66 s.: "The running time needed to analyze the 31 raw data files used to generate the results shown in Application 1 was 66 s."
- [methods] The procedure included retention time alignment, peak picking, deisotoping, integration, and grouping together adducts derived from the same compound.: "The procedure included retention time alignment, peak picking, deisotoping, integration, and grouping together adducts derived from the same compound."
- [abstract] The capabilities of the package are shown with pipelines for an LC–MS system suitability check, system conditioning, signal drift evaluation, and data curation.: "The capabilities of the package are shown with pipelines for an LC–MS system suitability check, system conditioning, signal drift evaluation, and data curation."
