---
name: metabolite-peak-area-correlation-across-preprocessing-methods
description: Use when validating automated MSI-CE-MS preprocessing against a manual
  analyst workflow by comparing peak areas across metabolites and precision on repeat
  pooled QC samples.
when_to_use_negative:
- Input data are unprocessed raw spectra rather than preprocessed peak areas from
  both pipelines — correlation requires a completed peak-area table from each method
  first
- Data were not acquired by MSI-CE-MS or lack flanking/stable-isotope internal standards
  needed to compute migration time indices — PeakMeister annotation may misassign
  peaks, invalidating the comparison
- The goal is de novo feature discovery or untargeted alignment; PeakMeister is a
  targeted full-scan preprocessing tool and this skill only assesses its agreement
  with manual targeted preprocessing
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: PeakMeister
  role: Automated targeted preprocessing of mzML MSI-CE-MS data whose peak-area output
    is correlated against manually preprocessed areas
  repo: github.com/PBM-Group/PeakMeister
- name: R
  role: Statistical environment in which PeakMeister runs and in which Pearson correlations,
    CVs, and control charts are computed
- name: MSI-CE-MS
  role: High-throughput separation platform generating the multiplexed serum and pooled
    QC datasets being compared
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c03513
    title: A Software Tool for Rapid and Automated Preprocessing of Large-Scale Serum
      Metabolomic Data by Multisegment Injection-Capillary Electrophoresis-Mass Spectrometry
  source_content_hash: sha256:be3e8ea13ae4656f8f735cbbf26de444f42069ff0452d28ee2cd9d024555a01e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PBM-Group/PeakMeister
  tools:
  - MSI-CE-MS
  - PeakMeister
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# metabolite-peak-area-correlation-across-preprocessing-methods

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates an automated preprocessing pipeline (PeakMeister) against manual preprocessing by computing per-metabolite Pearson correlations of peak areas across all serum metabolites and comparing intermediate precision (median CV) of pooled QC samples, before and after QC-based batch correction.

## When to use

Use when validating automated MSI-CE-MS preprocessing against a manual analyst workflow by comparing peak areas across metabolites and precision on repeat pooled QC samples.

## When NOT to use

- Input data are unprocessed raw spectra rather than preprocessed peak areas from both pipelines — correlation requires a completed peak-area table from each method first
- Data were not acquired by MSI-CE-MS or lack flanking/stable-isotope internal standards needed to compute migration time indices — PeakMeister annotation may misassign peaks, invalidating the comparison
- The goal is de novo feature discovery or untargeted alignment; PeakMeister is a targeted full-scan preprocessing tool and this skill only assesses its agreement with manual targeted preprocessing

## Inputs

- mzML files from MSI-CE-MS runs (serum samples plus repeat pooled QC injections)
- QC replicate peak-area table (e.g., supplementary Table S1) from PeakMeister and from manual preprocessing
- PeakMeister 'Mass List and Parameters.xlsx' template with analyte m/z values, extraction.window.ppm, and internal-standard migration times
- Optional: quantified concentrations for NIST SRM-1950 reference plasma metabolites

## Outputs

- Per-metabolite Pearson correlation coefficients (r) between automated and manual peak areas, plus their median (target ~0.98)
- Intermediate precision summary: median CV across pooled QCs per method (target 16.0%)
- Control-chart CVs per tracked metabolite (e.g., F-Phe), before and after QC-based batch correction
- Mean integration bias (%) between methods (target ~-2.1%)

## How to apply

Load QC replicate peak areas (e.g., Table S1 of the validation study, n = 420 pooled QCs from ENANI-2019) into R or Python, then compute per-metabolite Pearson correlations between PeakMeister and manually preprocessed peak areas across all serum metabolites (e.g., 47 metabolites) and take the median r; the article reports r = 0.98 as the mutual-agreement benchmark. Next compute intermediate precision as the median CV across pooled QCs (article benchmark: 16.0% for both methods) and method-specific control-chart CVs for tracked metabolites such as F-Phe (9.2% manual vs 8.5% PeakMeister). Repeat CV computation after QC-based batch correction to confirm attenuation (7.8% vs 7.5% for F-Phe). Also quantify integration bias as the mean percent difference in peak areas (article benchmark: mean bias = -2.1%) and, if reference material is available, mutual agreement of absolute quantification (mean bias = -1.3% for 16 NIST SRM-1950 plasma metabolites). PeakMeister processes mzML files directly using migration time indices from 11 internal standards rather than time warping, so ensure both pipelines started from identical mzML inputs and matched parameter sheets (extraction.window.ppm ~30-35, snr.threshold for <LOD flags, minimum.peak.width.seconds) before comparing.

## Related tools

- **PeakMeister** (Automated targeted preprocessing of mzML MSI-CE-MS data whose peak-area output is correlated against manually preprocessed areas) — github.com/PBM-Group/PeakMeister
- **R** (Statistical environment in which PeakMeister runs and in which Pearson correlations, CVs, and control charts are computed)
- **MSI-CE-MS** (High-throughput separation platform generating the multiplexed serum and pooled QC datasets being compared)

## Examples

```
Rscript code  # open the PeakMeister project file and run the preprocessing script on mzML Files/, then cor(peakmeister_areas, manual_areas, method="pearson") per metabolite
```

## Evaluation signals

- Median per-metabolite Pearson r between PeakMeister and manual peak areas matches the reported 0.98 (within rounding)
- Median intermediate-precision CV across pooled QCs matches the reported 16.0% for both methods
- F-Phe control-chart CVs reproduce 9.2% (manual) vs 8.5% (PeakMeister) before correction and 7.8% vs 7.5% after QC-based batch correction
- Mean peak-integration bias falls near the reported -2.1%
- Peak annotation fidelity check: median annotation accuracy >99.9% before areas are trusted for correlation

## Limitations

- PeakMeister was validated only for serum matrix; not yet validated for urine, saliva, stool, or other sample matrices
- Late-migrating metabolites lacking two flanking or a stable-isotope internal standard were prone to peak misannotation
- Low-abundance metabolites with signals near the method detection limit were prone to peak misannotation, which would corrupt correlation and CV estimates
- Only 16 plasma metabolites were checked for absolute quantification agreement (NIST SRM-1950, mean bias -1.3%), so quantification bias is characterized on a limited panel

## Evidence

- [abstract] consistent metabolite peak integration (mean bias = -2.1%) and good mutual agreement when quantifying 16 plasma metabolites from NIST SRM-1950 (mean bias = -1.3%): [evidence span withheld — anchor status: not_found]
