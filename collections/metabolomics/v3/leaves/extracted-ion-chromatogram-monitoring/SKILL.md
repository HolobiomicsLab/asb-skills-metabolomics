---
name: extracted-ion-chromatogram-monitoring
description: Use when raw LC–MS injections (e.g., Waters RAW converted to mzML) include
  system suitability solution and pooled QC samples and you must verify that SSS standards
  are detected via EICs before/while assessing platform conditioning.
when_to_use_negative:
- Input is already a processed feature/peak table with no raw chromatographic (mzML)
  data, so EICs cannot be extracted
- The goal is full feature detection, correspondence, or annotation — the article
  scopes TidyMS EIC/system-suitability work to QC checks (system suitability, conditioning,
  drift), not feature annotation, which TidyMS lacks
- The batch lacks SSS or pooled QC injections, leaving no reference injections against
  which to judge platform conditioning
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: TidyMS
  role: Python package used to load mzML data (pyopenms reader), run the system conditioning
    and system suitability pipelines, and extract the EIC/conditioning metrics
  repo: https://github.com/griquelme/tidyms
- name: pyopenms
  role: Underlying reader through which TidyMS loads the mzML files for EIC extraction
- name: ProteoWizard msconvert
  role: Converts the raw Waters RAW files to centroid-mode mzML (min distance = 0.01,
    snr = 1) prior to EIC monitoring
- name: Python
  role: Implementation language for the EIC/conditioning script and export of figures
    and CSV tables
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo10100416
    title: A Python-Based Pipeline for Preprocessing LC–MS Data for Untargeted Metabolomics
      Workflows
  source_content_hash: sha256:f1c1cbf9138a82d462881c20f0d48220e89ea0a179e113616963eb1e67fd2575
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/griquelme/tidyms
  tools:
  - Numpy
  - ProteoWizard
  - ProteoWizard msconvert
  - Python
  - Scipy
  - TidyMS
  - pyopenms
  - scikit-learn
  - statsmodels
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# extracted-ion-chromatogram-monitoring

## Summary

Generates and inspects extracted ion chromatograms (EICs) for the system suitability solution (SSS) standards Alogliptin, Phe-Phe, and Leu-Enk on centroid-mode mzML data, as part of the TidyMS LC–MS system suitability check and analytical platform conditioning workflow demonstrated on the NIST plasma RM study (MetaboLights MTBLS1919).

## When to use

Use when raw LC–MS injections (e.g., Waters RAW converted to mzML) include system suitability solution and pooled QC samples and you must verify that SSS standards are detected via EICs before/while assessing platform conditioning.

## When NOT to use

- Input is already a processed feature/peak table with no raw chromatographic (mzML) data, so EICs cannot be extracted
- The goal is full feature detection, correspondence, or annotation — the article scopes TidyMS EIC/system-suitability work to QC checks (system suitability, conditioning, drift), not feature annotation, which TidyMS lacks
- The batch lacks SSS or pooled QC injections, leaving no reference injections against which to judge platform conditioning

## Inputs

- Centroid-mode mzML files converted from Waters RAW injections (MetaboLights MTBLS1919 deposit)
- Batch sequence mapping identifying SSS injections, pooled/intrastudy QC injections, and study samples
- SSS composition and standard m/z targets (Alogliptin, Phe-Phe, Leu-Enk)

## Outputs

- System-suitability EIC plots for Alogliptin, Phe-Phe, and Leu-Enk
- Regenerated conditioning figures matching the paper's Figure 2 procedure
- conditioning_report.csv with conditioning metrics extracted from pooled/intrastudy QC and SSS injections

## How to apply

First convert the raw Waters RAW files to centroid-mode mzML with ProteoWizard msconvert (min distance = 0.01, snr = 1). Load the mzML files with TidyMS (pyopenms reader) and map the batch sequence, identifying the 100-fold diluted SSS (leucine enkephalin 25 µmol/L, LysoPC 0:0/18:0 17.9 µmol/L, L-phenylalanyl-L-phenylalanine 100 µmol/L, L-tryptophan 100 µmol/L, alogliptin 50 µmol/L) injected before platform conditioning and at the end of the batch. Extract ion chromatograms for the SSS standards Alogliptin, Phe-Phe, and Leu-Enk to reproduce the system-suitability EIC assessment, then apply the system suitability procedure on pooled QC samples. The rationale is to confirm stable platform response across conditioning injections and detection of the SSS standards in the EICs, as in the Figure 2 procedure of the TidyMS paper.

## Related tools

- **TidyMS** (Python package used to load mzML data (pyopenms reader), run the system conditioning and system suitability pipelines, and extract the EIC/conditioning metrics) — https://github.com/griquelme/tidyms
- **pyopenms** (Underlying reader through which TidyMS loads the mzML files for EIC extraction)
- **ProteoWizard msconvert** (Converts the raw Waters RAW files to centroid-mode mzML (min distance = 0.01, snr = 1) prior to EIC monitoring)
- **Python** (Implementation language for the EIC/conditioning script and export of figures and CSV tables)

## Examples

```
msconvert sample.RAW --mzML --filter "peakPicking true 1-" && python run_conditioning.py --mzml_dir mzml/ --sss_standards Alogliptin,Phe-Phe,Leu-Enk --out conditioning_report.csv
```

## Evaluation signals

- Pipeline runs to completion on all MTBLS1919 injections without errors
- SSS standards Alogliptin, Phe-Phe, and Leu-Enk are detected in the generated EICs
- conditioning_report.csv contains conditioning metrics for both SSS and pooled/intrastudy QC injections
- Regenerated conditioning and system-suitability figures match the qualitative assessment reported in the paper (stable platform response across conditioning injections)
- Input mzML files are centroid mode and were converted with min distance = 0.01 and snr = 1

## Limitations

- TidyMS lacks feature annotation capability, so EIC monitoring cannot assign compound identities beyond the known SSS standards
- The workflow depends on the SSS being injected both before conditioning and at the end of the batch; batches without these injections cannot be assessed the same way
- Requires conversion of vendor Waters RAW files to centroid mzML first; profile-mode or other vendor formats need parameter-appropriate conversion (min distance = 0.01, snr = 1)

## Evidence

- [abstract] The capabilities of the package are shown with pipelines for an LC–MS system suitability check, system conditioning, signal drift evaluation, and data curation.: "The capabilities of the package are shown with pipelines for an LC–MS system suitability check, system conditioning, signal drift evaluation, and data curation."
- [intro] Two different applications are illustrated to show some of its capabilities, including a pipeline for an LC–MS system suitability check, analytical platform conditioning, and signal drift evaluation,: "Two different applications are illustrated to show some of its capabilities, including a pipeline for an LC–MS system suitability check, analytical platform conditioning, and signal drift evaluation,"
- [methods] The mass spectrometry data have been deposited to the Metabo Lights public repository [38] with the data set identifier MTBLS1919.: "The mass spectrometry data have been deposited to the Metabo Lights public repository [38] with the data set identifier MTBLS1919."
- [results] The running time needed to analyze the 31 raw data files used to generate the results shown in Application 1 was 66 s.: "The running time needed to analyze the 31 raw data files used to generate the results shown in Application 1 was 66 s."
- [methods] Source code, documentation, and installation instructions are available online at Github: https://github.com/griquelme/tidyms: "Source code, documentation, and installation instructions are available online at Github: https://github.com/griquelme/tidyms"
