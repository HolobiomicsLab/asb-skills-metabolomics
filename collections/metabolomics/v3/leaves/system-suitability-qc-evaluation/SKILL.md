---
name: system-suitability-qc-evaluation
description: Use when raw Waters RAW/mzML injections of a system suitability solution
  (SSS) and pooled QC samples are available before running an untargeted LC–MS metabolomics
  batch, and you must decide whether the analytical platform is adequately conditioned
  and stable before acquiring study samples.
when_to_use_negative:
- Input is already a processed feature table / extracted data matrix — this skill
  operates on raw LC–MS injections and sequence metadata, not curated features.
- The goal is downstream feature curation (RSD, D-ratio, blank filters) or statistical
  modeling of metabolomics data rather than pre-batch platform readiness assessment.
- Data were not acquired with a system suitability solution or pooled QC injections
  in the run — the conditioning metrics and EIC assessment have nothing to evaluate.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: TidyMS
  role: Python package providing the system-conditioning processor pipeline, EIC extraction,
    and QC-based system suitability check implemented for this skill
  repo: https://github.com/griquelme/tidyms
- name: pyopenms
  role: Raw file reader backend used by TidyMS to load mzML injections
- name: ProteoWizard
  role: Converts Waters RAW files to centroid-mode mzML (min distance = 0.01, snr
    = 1) before TidyMS processing
- name: Python
  role: Language in which the TidyMS system-conditioning and system-suitability pipelines
    are implemented and executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo10100416
    title: A Python-Based Pipeline for Preprocessing LC–MS Data for Untargeted Metabolomics
      Workflows
  source_content_hash: sha256:6d4694aeedc94c070f436ffef7eb3f3a87c2d683a556ac1119ef05468e090a04
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/griquelme/tidyms
  tools:
  - Numpy
  - ProteoWizard
  - Python
  - Scipy
  - TidyMS
  - pyopenms
  - scikit-learn
  - statsmodels
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# system-suitability-qc-evaluation

## Summary

Assesses LC–MS analytical-platform readiness before a study batch using TidyMS: extracts ion chromatograms for system-suitability-solution standards (Alogliptin, Phe-Phe, Leu-Enk) and applies the TidyMS system-conditioning pipeline to pooled/intrastudy QC injections to confirm a stable platform response.

## When to use

Use when raw Waters RAW/mzML injections of a system suitability solution (SSS) and pooled QC samples are available before running an untargeted LC–MS metabolomics batch, and you must decide whether the analytical platform is adequately conditioned and stable before acquiring study samples.

## When NOT to use

- Input is already a processed feature table / extracted data matrix — this skill operates on raw LC–MS injections and sequence metadata, not curated features.
- The goal is downstream feature curation (RSD, D-ratio, blank filters) or statistical modeling of metabolomics data rather than pre-batch platform readiness assessment.
- Data were not acquired with a system suitability solution or pooled QC injections in the run — the conditioning metrics and EIC assessment have nothing to evaluate.

## Inputs

- Raw Waters RAW mass-spectrometry files (e.g. from the MetaboLights MTBLS1919 deposit) or centroid-mode mzML conversions (ProteoWizard msconvert, min distance = 0.01, snr = 1)
- Batch/injection sequence metadata identifying system suitability solution (SSS) injections and pooled/intrastudy QC samples
- SSS composition table (leucine enkephalin 25 µmol/L, LysoPC 0:0/18:0 17.9 µmol/L, L-phenylalanyl-L-phenylalanine 100 µmol/L, L-tryptophan 100 µmol/L, alogliptin 50 µmol/L)

## Outputs

- conditioning_report.csv with per-injection conditioning metrics extracted by the TidyMS system-conditioning pipeline
- Regenerated system-conditioning figures showing platform response across conditioning injections
- Extracted ion chromatograms (EICs) for Alogliptin, Phe-Phe, and Leu-Enk showing detection of SSS standards
- System-suitability assessment results on pooled/intrastudy QC sample injections

## How to apply

Convert raw Waters RAW files to centroid-mode mzML with ProteoWizard msconvert (min distance = 0.01, snr = 1), load them with TidyMS (pyopenms reader), and map the batch sequence to identify SSS injections (leucine enkephalin 25 µmol/L, LysoPC 0:0/18:0 17.9 µmol/L, L-phenylalanyl-L-phenylalanine 100 µmol/L, L-tryptophan 100 µmol/L, alogliptin 50 µmol/L) injected before conditioning and at the end of the batch. Execute the TidyMS system-conditioning processor pipeline to extract conditioning metrics from the pooled/intrastudy QC and SSS injections, and generate extracted ion chromatograms for the SSS standards Alogliptin, Phe-Phe, and Leu-Enk to reproduce the system-suitability EIC assessment. Apply the same system suitability procedure on pooled QC sample injections as in Application 1 of the paper. The rationale is that a stable response across conditioning injections, with all SSS standards detected in the EICs, indicates the LC–MS platform is fit for acquisition of the plasma reference-material samples (NIST candidate RM 8231 / SRM 1950).

## Related tools

- **TidyMS** (Python package providing the system-conditioning processor pipeline, EIC extraction, and QC-based system suitability check implemented for this skill) — https://github.com/griquelme/tidyms
- **pyopenms** (Raw file reader backend used by TidyMS to load mzML injections)
- **ProteoWizard** (Converts Waters RAW files to centroid-mode mzML (min distance = 0.01, snr = 1) before TidyMS processing)
- **Python** (Language in which the TidyMS system-conditioning and system-suitability pipelines are implemented and executed)

## Examples

```
msconvert raw.Waters.RAW --mzML --filter 'peakpicking' -o mzml/ && python run_conditioning.py --mzml mzml/ --sss-stds Alogliptin,Phe-Phe,Leu-Enk --out conditioning_report.csv
```

## Evaluation signals

- Pipeline runs to completion on all MTBLS1919 injections without errors and produces conditioning_report.csv
- Conditioning figures show a stable platform response across successive conditioning injections, qualitatively matching the assessment reported for the NIST plasma RM study
- EICs for Alogliptin, Phe-Phe, and Leu-Enk each show a detected chromatographic peak at the expected retention time
- Applying the same system suitability procedure to pooled/intrastudy QC injections yields metrics consistent with those reported in Application 1 of the paper
- Re-running the pipeline on the same mzML inputs reproduces the same conditioning metrics (TidyMS advertises rapid, reproducible preprocessing; Application 1 processed 31 files in 66 s)

## Limitations

- TidyMS is feature-annotation-incapable in this version, so the skill cannot annotate the SSS standards — EIC detection is by m/z and RT, not by spectral annotation.
- The assessment depends on the SSS composition and injection order being correctly recorded in the sequence metadata; a mis-mapped batch yields meaningless conditioning metrics.
- Validated only on the NIST candidate RM 8231 / SRM 1950 plasma acquisitions deposited as MTBLS1919; behavior on other matrices or instrument platforms is not established in the article.

## Evidence

- [abstract] pipelines for an LC–MS system suitability check, system conditioning, signal drift evaluation, and data curation: "The capabilities of the package are shown with pipelines for an LC–MS system suitability check, system conditioning, signal drift evaluation, and data curation."
- [methods] mass spectrometry data have been deposited to the Metabo Lights public repository: "The mass spectrometry data have been deposited to the Metabo Lights public repository [38] with the data set identifier MTBLS1919."
- [results] running time needed to analyze the 31 raw data files used to generate the results shown in Application 1 was 66 s: "The running time needed to analyze the 31 raw data files used to generate the results shown in Application 1 was 66 s."
- [abstract] workflow that can be used in an automated or semi-automated fashion: "The package offers a rapid and reproducible workflow that can be used in an automated or semi-automated fashion, and it is an open and free tool available to all users."
- [intro] open and free to all users to promote FAIR principles: "The tool is open and free to all users to promote Findable, Accessible, Interoperable, and Reusable (FAIR) principles for the benefit of the community [31]."
