---
name: qc-intermediate-precision-cv-analysis
description: Use when validating an automated peak-integration/preprocessing tool
  against manual expert preprocessing, using repeat pooled QC samples to estimate
  intermediate precision (median CV) and mutual agreement of metabolite peak areas.
when_to_use_negative:
- The dataset lacks repeat pooled QC samples injected across batches, so no intermediate
  precision (within/across-run CV) can be estimated.
- The goal is annotation fidelity or peak-picking accuracy rather than quantitative
  precision/agreement — use peak annotation accuracy metrics instead.
- Input is raw mzML data requiring preprocessing itself; this skill operates on already-extracted
  peak-area tables.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: PeakMeister
  role: Automated targeted MSI-CE-MS preprocessing tool whose QC peak areas are compared
    against manual integration for intermediate precision and mutual agreement
  repo: https://github.com/PBM-Group/PeakMeister
- name: R
  role: Statistical environment in which PeakMeister is written and used to compute
    CVs and Pearson correlations of QC peak areas
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c03513
    title: A Software Tool for Rapid and Automated Preprocessing of Large-Scale Serum
      Metabolomic Data by Multisegment Injection-Capillary Electrophoresis-Mass Spectrometry
  source_content_hash: sha256:8cb25974cf3c275316e2fb87481f3324a0da8dc64b2b8126df7b948bf53ffe00
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# qc-intermediate-precision-cv-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantify intermediate precision of an automated MSI-CE-MS preprocessing pipeline (PeakMeister) versus manual integration by computing coefficients of variation (CVs) across pooled QC replicate samples, before and after QC-based batch correction, and by cross-method Pearson correlation of peak areas.

## When to use

Use when validating an automated peak-integration/preprocessing tool against manual expert preprocessing, using repeat pooled QC samples to estimate intermediate precision (median CV) and mutual agreement of metabolite peak areas.

## When NOT to use

- The dataset lacks repeat pooled QC samples injected across batches, so no intermediate precision (within/across-run CV) can be estimated.
- The goal is annotation fidelity or peak-picking accuracy rather than quantitative precision/agreement — use peak annotation accuracy metrics instead.
- Input is raw mzML data requiring preprocessing itself; this skill operates on already-extracted peak-area tables.

## Inputs

- QC replicate peak-area table from the validation study (e.g., Table S1 supplementary) for n = 420 pooled QC samples
- Article-reported precision metrics (median CV = 16.0%; F-Phe control-chart CVs 9.2%/8.5%; post-batch-correction 7.8%/7.5%)
- Peak-area tables for 47 serum metabolites produced by PeakMeister (automated) and by manual expert integration

## Outputs

- intermediate_precision_summary.csv tabulating per-metabolite and median CVs for both methods, before and after QC-based batch correction
- Per-metabolite Pearson correlation coefficients between PeakMeister and manually preprocessed peak areas, with their median r
- Validation verdict comparing reproduced metrics against reported values (median CV 16.0%, F-Phe CVs, median r = 0.98)

## How to apply

Load the pooled QC replicate peak areas (e.g., n = 420 repeat pooled QCs from ENANI-2019 serum samples processed by PeakMeister and by manual integration) from the supplementary table using pandas in Python or R. Extract or recompute per-method precision metrics: the median CV across metabolites (reported as 16.0% for both methods) and per-metabolite control-chart CVs (e.g., F-Phe: 9.2% manual vs 8.5% PeakMeister), then recompute these after QC-based batch correction (F-Phe attenuating to 7.8% vs 7.5%). Next, compute per-metabolite Pearson correlations between PeakMeister and manually preprocessed peak areas across all 47 serum metabolites and take the median r. Tabulate all extracted CVs and the median correlation (expected r = 0.98) in a summary CSV (intermediate_precision_summary.csv); accept the validation if reproduced values match reported metrics within rounding.

## Related tools

- **PeakMeister** (Automated targeted MSI-CE-MS preprocessing tool whose QC peak areas are compared against manual integration for intermediate precision and mutual agreement) — https://github.com/PBM-Group/PeakMeister
- **R** (Statistical environment in which PeakMeister is written and used to compute CVs and Pearson correlations of QC peak areas)

## Examples

```
import pandas as pd; qc = pd.read_csv('tableS1_qc_peak_areas.csv'); cvs = qc.groupby('metabolite')['peak_area'].apply(lambda x: x.std()/x.mean()*100); print(cvs.median())
```

## Evaluation signals

- Reproduced median Pearson r between PeakMeister and manual peak areas across 47 serum metabolites matches the reported 0.98 within rounding.
- Extracted/recomputed median intermediate-precision CV equals the reported 16.0% for both methods (n = 420 pooled QCs).
- F-Phe control-chart CVs reproduce 9.2% (manual) vs 8.5% (PeakMeister) before correction and 7.8% vs 7.5% after QC-based batch correction.
- Both methods yield consistent precision (median CV = 16%), indicating the automated pipeline does not degrade reproducibility relative to expert manual integration.
- Summary CSV (intermediate_precision_summary.csv) contains all extracted CV values and the correlation metric, matching reported values within rounding.

## Limitations

- The reference workflow covers 47 serum metabolites in MSI-CE-MS serum samples; PeakMeister is not yet validated for other matrices (urine, saliva, stool).
- Low-abundance metabolites with signals near the method detection limit (recorded as '<LOD' when S/N falls below snr.threshold) can be prone to peak misannotation, affecting precision estimates.
- Late migrating serum metabolites lacking two flanking or a stable-isotope internal standard were prone to peak misannotation.

## Evidence
