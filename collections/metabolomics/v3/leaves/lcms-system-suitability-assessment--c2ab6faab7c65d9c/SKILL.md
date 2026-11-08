---
name: lcms-system-suitability-assessment
description: Use when raw LC–MS mzML files from a plasma reference material batch
  (e.g., NIST candidate RM 8231 or SRM 1950, MTBLS1919 deposit) must be checked for
  system suitability using the five SSS standard compounds and pooled/intrastudy QC
  injections.
when_to_use_negative:
- When the goal is LC–MS data curation (feature table cleaning, RSD/D-ratio/prevalence
  filters) rather than system suitability assessment — use the TidyMS data curation
  pipeline instead
- When the objective is evaluating analytical platform conditioning (conditioning
  metrics over conditioning injections) rather than suitability of the system at the
  start/end of the batch
- When the input data are already feature tables (extracted data matrices) rather
  than raw mzML chromatographic data, since this skill operates directly on raw mzML
  files via the pyopenms reader
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: TidyMS
  role: 'Primary package: loads mzML files (pyopenms reader), builds chromatograms,
    performs CWT-based peak detection, extracts EICs for SSS standards, and applies
    the system suitability pipeline on pooled/intrastudy QC injections.'
  repo: https://github.com/griquelme/tidyms
- name: Python
  role: Host language in which TidyMS runs and in which the suitability report figure
    (system_suitability_report.png) is rendered.
- name: pyopenms
  role: File reader backend used by TidyMS to load the mzML raw data files.
- name: ProteoWizard
  role: Used to convert the deposited Waters RAW files to centroid-mode mzML (min
    distance = 0.01, snr = 1) before TidyMS processing.
- name: Pandas
  role: Handles the batch/sample-type mapping and suitability metric tables for the
    pooled/intrastudy QC injections.
- name: Numpy
  role: Numeric array handling underlying chromatogram construction and peak-picking
    outputs.
- name: Scipy
  role: Supports the CWT-based peak detection implementation on the chromatographic
    data.
- name: scikit-learn
  role: Available within the Python scientific stack that TidyMS integrates with;
    used for downstream model generation after suitability evaluation.
- name: statsmodels
  role: Statistical support for suitability metric summaries on the pooled/intrastudy
    QC injections.
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.3390/metabo10100416
    title: A Python-Based Pipeline for Preprocessing LC–MS Data for Untargeted Metabolomics
      Workflows
  source_content_hash: sha256:fb9affbe1d1802410ca63fef62ec8506b77d354c89f7c8b25c288f36340696d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/griquelme/tidyms
  tools:
  - Numpy
  - Pandas
  - ProteoWizard
  - Python
  - Scipy
  - TidyMS
  - pyopenms
  - scikit-learn
  - statsmodels
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# lcms-system-suitability-assessment

## Summary

TidyMS-based LC–MS system suitability check that loads 31 mzML raw files from the NIST candidate RM 8231 / SRM 1950 plasma batches (MetaboLights MTBLS1919), extracts EICs for five system suitability solution (SSS) standards, and applies the TidyMS suitability procedure on pooled/intrastudy QC injections to assess retention-time and m/z stability.

## When to use

Use when raw LC–MS mzML files from a plasma reference material batch (e.g., NIST candidate RM 8231 or SRM 1950, MTBLS1919 deposit) must be checked for system suitability using the five SSS standard compounds and pooled/intrastudy QC injections.

## When NOT to use

- When the goal is LC–MS data curation (feature table cleaning, RSD/D-ratio/prevalence filters) rather than system suitability assessment — use the TidyMS data curation pipeline instead
- When the objective is evaluating analytical platform conditioning (conditioning metrics over conditioning injections) rather than suitability of the system at the start/end of the batch
- When the input data are already feature tables (extracted data matrices) rather than raw mzML chromatographic data, since this skill operates directly on raw mzML files via the pyopenms reader

## Inputs

- mzML raw data files for the candidate RM 8231 and SRM 1950 plasma batches (MetaboLights MTBLS1919, converted from Waters RAW via ProteoWizard msconvert)
- Batch sequence / sample-type map identifying SSS injections before platform conditioning, at the end of the batch, and pooled/intrastudy QC injections
- Five system suitability solution (SSS) standard compounds: leucine enkephalin, lysophosphatidylcholine (0:0/18:0), L-phenylalanyl-L-phenylalanine, L-tryptophan, alogliptin

## Outputs

- Extracted ion chromatograms (EICs) for the five SSS standards from system suitability QC injections
- System suitability procedure plot applied to pooled/intrastudy QC injections
- Combined suitability report figure saved as system_suitability_report.png (equivalent to Supplementary Figures S1–S2)

## How to apply

Download the mzML raw data files (converted from Waters RAW with ProteoWizard msconvert at centroid mode, min distance = 0.01, snr = 1) for the candidate RM 8231 and SRM 1950 plasma batches from MetaboLights (MTBLS1919). Load each file with TidyMS via the pyopenms reader, build chromatograms from consecutive MS scans, and detect peaks with the CWT-based peak-picking algorithm with parameters optimized for chromatographic data. Extract extracted ion chromatograms for the five SSS standards — leucine enkephalin, lysophosphatidylcholine (0:0/18:0), L-phenylalanyl-L-phenylalanine, L-tryptophan, and alogliptin (100-fold diluted SSS: Leu-Enk 25 µmol/L, LysoPC 17.9 µmol/L, Phe-Phe 100 µmol/L, Trp 100 µmol/L, alogliptin 50 µmol/L) — from the system suitability QC injections analyzed before platform conditioning and at the end of the batch. Then apply the TidyMS system suitability pipeline to the pooled/intrastudy QC injections to assess retention-time and m/z stability of the suitability markers across the analytical run, rendering the EIC plots and suitability figures equivalent to Supplementary Figures S1–S2. Save the combined report as system_suitability_report.png.

## Related tools

- **TidyMS** (Primary package: loads mzML files (pyopenms reader), builds chromatograms, performs CWT-based peak detection, extracts EICs for SSS standards, and applies the system suitability pipeline on pooled/intrastudy QC injections.) — https://github.com/griquelme/tidyms
- **Python** (Host language in which TidyMS runs and in which the suitability report figure (system_suitability_report.png) is rendered.)
- **pyopenms** (File reader backend used by TidyMS to load the mzML raw data files.)
- **ProteoWizard** (Used to convert the deposited Waters RAW files to centroid-mode mzML (min distance = 0.01, snr = 1) before TidyMS processing.)
- **Pandas** (Handles the batch/sample-type mapping and suitability metric tables for the pooled/intrastudy QC injections.)
- **Numpy** (Numeric array handling underlying chromatogram construction and peak-picking outputs.)
- **Scipy** (Supports the CWT-based peak detection implementation on the chromatographic data.)
- **scikit-learn** (Available within the Python scientific stack that TidyMS integrates with; used for downstream model generation after suitability evaluation.)
- **statsmodels** (Statistical support for suitability metric summaries on the pooled/intrastudy QC injections.)

## Examples

```
from tidyms import MzMLReader; reader = MzMLReader('SSS_injection.mzML'); chromatograms = reader.get_chromatograms(); features = tidyms.peak_picking.detect_peaks(chromatograms, method='cwt')
```

## Evaluation signals

- The combined report must contain EICs for all five SSS compounds — leucine enkephalin, LysoPC(0:0/18:0), L-phenylalanyl-L-phenylalanine, L-tryptophan, and alogliptin — with visible chromatographic peaks.
- The suitability procedure plot must be generated from pooled/intrastudy QC injections and show stable retention-time and m/z values of the suitability markers across the analytical run.
- The output figure must match the content of Supplementary Figures S1–S2 (EICs of SSS standards and suitability plot applied to pooled QC samples).
- Processing of the 31 raw data files should complete on the order of the reported ~66 s runtime, with feature detection taking about 2 s per sample.
- The peak-picking parameters must be the CWT-based algorithm with settings optimized for chromatographic data, and the mzML files must have been produced by ProteoWizard msconvert with centroid mode, min distance = 0.01, snr = 1.

## Limitations

- The system suitability check requires the specific SSS mixture of five standards at the reported concentrations; it cannot be inferred from plasma-only injections without the SSS injections before platform conditioning and at the end of the batch.
- The mzML conversion parameters (centroid mode, min distance = 0.01, snr = 1) are specific to this Waters RAW dataset; applying different conversion settings may alter peak shapes and suitability metrics.
- TidyMS lacks feature annotation capability, so the suitability assessment does not provide compound-level annotation beyond the known SSS standard identities.
- The study did not state software version numbers for TidyMS or the Python stack, so exact reproducibility across environments may vary.

## Evidence

- [abstract] TidyMS supports pipelines for LC–MS system suitability check, system conditioning, signal drift evaluation, and data curation.: [evidence span withheld — anchor status: not_found]
- [methods] The mass spectrometry data have been deposited to the Metabo Lights public repository with the data set identifier MTBLS1919.: "The mass spectrometry data have been deposited to the Metabo Lights public repository [38] with the data set identifier MTBLS1919."
- [results] Analyzing 31 raw data files in Application 1 took 66 s, with feature detection about 2 s per sample.: "The running time needed to analyze the 31 raw data files used to generate the results shown in Application 1 was 66 s."
- [other] Use TidyMS to load each mzML file and build chromatograms from consecutive MS scans, detecting peaks with the CWT-based peak-picking algorithm with parameters optimized for chromatographic data.: [evidence span withheld — anchor status: not_found]
- [methods] Source code, documentation, and installation instructions are available online at Github: https://github.com/griquelme/tidyms.: "Source code, documentation, and installation instructions are available online at Github: https://github.com/griquelme/tidyms"
