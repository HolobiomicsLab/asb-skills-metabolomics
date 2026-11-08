---
name: cwt-based-peak-detection
description: Use when raw LC–MS mzML files must be turned into detected chromatographic
  peaks (TIC/EIC peaks) before feature correspondence, system suitability checks,
  or signal-drift evaluation in untargeted metabolomics QC.
when_to_use_negative:
- Input is already an extracted feature table — peak detection is a raw-scan-level
  step and re-running it on curated matrices is redundant.
- The goal is feature annotation or metabolite identification — the article notes
  TidyMS lacks feature annotation capability.
- Data are not chromatographic LC–MS scans (e.g., direct-infusion spectra), since
  the CWT parameters are optimized for chromatographic data and chromatogram building
  assumes consecutive MS scans.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: TidyMS
  role: Provides the CWT-based peak-picking algorithm, chromatogram building, and
    the DataContainer/Pipeline objects that hold detected features
  repo: https://github.com/griquelme/tidyms
- name: pyopenms
  role: Reads raw mzML files into TidyMS MSData objects prior to chromatogram building
    and peak detection
- name: Python
  role: Language runtime in which the peak-picking pipeline and parameterization are
    executed
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.3390/metabo10100416
    title: A Python-Based Pipeline for Preprocessing LC–MS Data for Untargeted Metabolomics
      Workflows
  source_content_hash: sha256:8ab930f77bfdadbdab6de5b0c8521d4b4dbae5a292c66992974aee3d724c820d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/griquelme/tidyms
  tools:
  - Numpy
  - Python
  - Scipy
  - TidyMS
  - pyopenms
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# cwt-based-peak-detection

## Summary

TidyMS's continuous-wavelet-transform (CWT) peak-picking algorithm, parameterized for chromatographic data, detects peaks in chromatograms built from consecutive MS scans of raw mzML files — the first quantitative step of the package's untargeted metabolomics QC pipelines.

## When to use

Use when raw LC–MS mzML files must be turned into detected chromatographic peaks (TIC/EIC peaks) before feature correspondence, system suitability checks, or signal-drift evaluation in untargeted metabolomics QC.

## When NOT to use

- Input is already an extracted feature table — peak detection is a raw-scan-level step and re-running it on curated matrices is redundant.
- The goal is feature annotation or metabolite identification — the article notes TidyMS lacks feature annotation capability.
- Data are not chromatographic LC–MS scans (e.g., direct-infusion spectra), since the CWT parameters are optimized for chromatographic data and chromatogram building assumes consecutive MS scans.

## Inputs

- Raw LC–MS data files in mzML format (e.g., the 31 files from MetaboLights MTBLS1919 for NIST candidate RM 8231 and SRM 1950 plasma batches)
- Target m/z values for extracted ion chromatograms (e.g., the five system suitability solution standard compounds)
- Peak-picking parameters optimized for chromatographic data (CWT-based algorithm settings)

## Outputs

- Chromatograms built from consecutive MS scans per sample
- Detected chromatographic peaks / feature table (retention time, m/z pairs) with per-feature and per-sample descriptors stored in a TidyMS DataContainer (Pandas DataFrames)
- EIC plots for the SSS compounds rendered in reports such as system_suitability_report.png

## How to apply

Load each mzML raw data file with pyopenms into a TidyMS MSData object, then build chromatograms from consecutive MS scans (TIC or extracted ion chromatograms at target m/z values). Run the CWT-based peak-picking algorithm with parameters optimized for chromatographic data; this is the detection engine TidyMS also exposes alongside centWave-style feature detection and correspondence routines that produce the extracted data matrix. For the NIST candidate RM 8231 / SRM 1950 system suitability application, extract EICs for the five SSS standards — leucine enkephalin, lysophosphatidylcholine (0:0/18:0), L-phenylalanyl-L-phenylalanine, L-tryptophan, and alogliptin — from suitability QC injections taken before platform conditioning and at the end of the batch. Expect roughly 2 s of feature detection per sample and about 66 s total for the 31 raw files of Application 1, so substantially slower runs indicate mis-set parameters. Detected peaks then feed downstream steps (retention-time alignment, deisotoping, integration, adduct grouping) and QC filters such as the RSD < 20% in pooled/intrastudy QC and the 10-fold blank-area rule.

## Related tools

- **TidyMS** (Provides the CWT-based peak-picking algorithm, chromatogram building, and the DataContainer/Pipeline objects that hold detected features) — https://github.com/griquelme/tidyms
- **pyopenms** (Reads raw mzML files into TidyMS MSData objects prior to chromatogram building and peak detection)
- **Python** (Language runtime in which the peak-picking pipeline and parameterization are executed)

## Evaluation signals

- EIC plots in the report must show peaks for all five SSS compounds (leucine enkephalin, LysoPC(0:0/18:0), L-phenylalanyl-L-phenylalanine, L-tryptophan, alogliptin), reproducing the content of Supplementary Figures S1–S2.
- Feature detection runtime should be about 2 s per sample; processing the 31 Application 1 raw files should complete on the order of the reported 66 s.
- Retention-time and m/z values of the SSS suitability markers in pooled/intrastudy QC injections should remain stable across the analytical run.
- Per-feature and per-sample descriptors from the raw mzML input must be preserved in the DataContainer after processing, matching the paper's object-interaction design (Figure S5).
- Detected features absent from pooled/intrastudy QC samples should be removable by the downstream QC-presence filter, confirming peak detection produced QC-comparable feature intensities.

## Limitations

- No software version numbers are stated for TidyMS or Python, so reproducing exact peak-detection behavior requires pinning the repository state.
- TidyMS lacks feature annotation capability — detected peaks are not identified.
- Parameters are optimized for chromatographic data from the NIST plasma reference-material experiments; other matrices or instruments may require retuning.

## Evidence

- [abstract] The present work introduces TidyMS, a package for the Python programming language for preprocessing LC–MS data for quality control (QC) procedures in untargeted metabolomics workflows.: "The present work introduces TidyMS, a package for the Python programming language for preprocessing LC–MS data"
- [methods] LC–MS preprocessing procedure combining alignment, peak picking, and adduct grouping.: "The procedure included retention time alignment, peak picking, deisotoping, integration, and grouping together adducts derived from the same compound."
- [results] Runtime benchmark for feature detection over the Application 1 raw files.: "The running time needed to analyze the 31 raw data files used to generate the results shown in Application 1 was 66 s."
- [results] CWT peak-picking applied with chromatography-optimized parameters and QC-based filtering downstream.: "all features with an RSD larger than 20% in pooled /intrastudy QC sample injections were eliminated."
