---
name: metabolomics-qc-pipeline-execution
description: Use when raw LC–MS injections (mzML from Waters RAW converted with msconvert)
  plus a batch sequence mapping SSS and pooled/intrastudy QC injections must be assessed
  for platform conditioning and system suitability with TidyMS.
when_to_use_negative:
- Input is already a feature table or extracted data matrix — this skill operates
  on raw mzML injections through TidyMS's reader and processor pipeline
- The goal is downstream feature annotation or statistical modeling of curated features
  — TidyMS explicitly lacks feature annotation capability, and data curation/signal-drift
  pipelines are separate skills
- Data are not QC-oriented injections (no SSS or pooled/intrastudy QC samples in the
  batch) — the conditioning assessment depends on those injection types
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: TidyMS
  role: Core Python package whose processor pipelines execute the system conditioning
    and LC–MS system suitability check on mzML files loaded via the pyopenms reader
  repo: https://github.com/griquelme/tidyms
- name: ProteoWizard
  role: Converts deposited Waters RAW files to centroid-mode mzML (min distance =
    0.01, snr = 1) before TidyMS ingestion
- name: pyopenms
  role: Reader backend TidyMS uses to load the converted mzML files
- name: Python
  role: Language in which TidyMS is implemented and in which the conditioning pipeline
    is scripted alongside Pandas/Numpy/Scipy
- name: scikit-learn
  role: Python machine-learning framework available for downstream model integration
    after TidyMS preprocessing
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo10100416
    title: A Python-Based Pipeline for Preprocessing LC–MS Data for Untargeted Metabolomics
      Workflows
  source_content_hash: sha256:8c3ef55238f26fd3536656f4a2c2fd80a92d447cf8c00dfacfae16666138d22c
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
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# metabolomics-qc-pipeline-execution

## Summary

Executes the TidyMS system-conditioning and LC–MS system suitability pipelines on raw metabolomics injections (e.g. MetaboLights MTBLS1919 plasma RM data) to regenerate QC assessment tables and figures, as demonstrated in the Riquelme et al. 2020 TidyMS paper on NIST candidate RM 8231 and SRM 1950.

## When to use

Use when raw LC–MS injections (mzML from Waters RAW converted with msconvert) plus a batch sequence mapping SSS and pooled/intrastudy QC injections must be assessed for platform conditioning and system suitability with TidyMS.

## When NOT to use

- Input is already a feature table or extracted data matrix — this skill operates on raw mzML injections through TidyMS's reader and processor pipeline
- The goal is downstream feature annotation or statistical modeling of curated features — TidyMS explicitly lacks feature annotation capability, and data curation/signal-drift pipelines are separate skills
- Data are not QC-oriented injections (no SSS or pooled/intrastudy QC samples in the batch) — the conditioning assessment depends on those injection types

## Inputs

- Raw LC–MS injections (Waters RAW files from MetaboLights deposit MTBLS1919)
- Centroid-mode mzML files converted with ProteoWizard msconvert (min distance = 0.01, snr = 1)
- Batch sequence table mapping injection order, sample type (SSS, pooled/intrastudy QC, study samples), and conditioning position

## Outputs

- conditioning_report.csv with per-injection conditioning metrics
- Regenerated system-conditioning figures (analogous to Figure 2)
- System-suitability EIC plots for Alogliptin, Phe-Phe, and Leu-Enk
- System suitability assessment applied to pooled QC samples

## How to apply

Convert Waters RAW files from the deposit to centroid-mode mzML with ProteoWizard msconvert (min distance = 0.01, snr = 1), then load them with the TidyMS pyopenms reader and map the batch sequence to locate the 100-fold diluted system suitability solution (SSS: leucine enkephalin 25 µmol/L, LysoPC 0:0/18:0 17.9 µmol/L, L-phenylalanyl-L-phenylalanine 100 µmol/L, L-tryptophan 100 µmol/L, alogliptin 50 µmol/L) injected before conditioning and at the batch end. Run the TidyMS system-conditioning processor pipeline to extract conditioning metrics from the pooled/intrastudy QC and SSS injections, following the Figure 2 procedure. Extract ion chromatograms for the SSS standards Alogliptin, Phe-Phe, and Leu-Enk to reproduce the system-suitability EIC assessment, and apply the system suitability procedure on pooled QC samples. Write the metrics to conditioning_report.csv and export the regenerated conditioning and system-suitability figures; TidyMS provides pipelines for system suitability check, system conditioning, signal drift evaluation, and data curation, and processed 31 raw files in 66 s (~2 s per sample) in Application 1.

## Related tools

- **TidyMS** (Core Python package whose processor pipelines execute the system conditioning and LC–MS system suitability check on mzML files loaded via the pyopenms reader) — https://github.com/griquelme/tidyms
- **ProteoWizard** (Converts deposited Waters RAW files to centroid-mode mzML (min distance = 0.01, snr = 1) before TidyMS ingestion)
- **pyopenms** (Reader backend TidyMS uses to load the converted mzML files)
- **Python** (Language in which TidyMS is implemented and in which the conditioning pipeline is scripted alongside Pandas/Numpy/Scipy)
- **scikit-learn** (Python machine-learning framework available for downstream model integration after TidyMS preprocessing)

## Examples

```
msconvert MTBLS1919/*.RAW --mzML --centroid --filter "peakPicking true 1" && python run_conditioning_pipeline.py --mzml mzml/ --sequence batch_sequence.csv --report conditioning_report.csv
```

## Evaluation signals

- Pipeline runs to completion on all MTBLS1919 injections without errors
- conditioning_report.csv is produced with per-injection conditioning metrics for SSS and pooled/intrastudy QC injections
- Regenerated conditioning figures qualitatively match the paper's assessment: stable platform response across conditioning injections
- SSS standards (Alogliptin, Phe-Phe, Leu-Enk) are detected in the reproduced EIC plots
- EIC extraction and msconvert parameters (min distance = 0.01, snr = 1) reproduce the centroid-mode settings stated in the workflow

## Limitations

- TidyMS lacks feature annotation capability, so the pipeline stops at QC assessment and cannot annotate detected ions
- No software version numbers are stated for TidyMS, Python, or scikit-learn, so exact reproducibility of package behavior may vary
- The demonstrated assessment relies on specific QC structures (SSS before conditioning and at batch end, pooled/intrastudy QC); batches lacking those injections cannot reproduce the Figure 2 conditioning assessment

## Evidence

- [abstract] TidyMS supports system conditioning and system suitability pipelines: "The capabilities of the package are shown with pipelines for an LC–MS system suitability check, system conditioning, signal drift evaluation, and data curation."
- [methods] Data deposited in MetaboLights under MTBLS1919: "The mass spectrometry data have been deposited to the Metabo Lights public repository [38] with the data set identifier MTBLS1919."
- [results] Runtime benchmark for Application 1 pipeline: "The running time needed to analyze the 31 raw data files used to generate the results shown in Application 1 was 66 s."
- [intro] NIST plasma reference materials exemplify the pipelines: "Both applications were exemplified with the analysis of a new suite of candidate plasma reference materials for untargeted metabolomics (candidate RM 8231)"
- [methods] TidyMS source and documentation hosted on GitHub: "Source code, documentation, and installation instructions are available online at Github: https://github.com/griquelme/tidyms"
