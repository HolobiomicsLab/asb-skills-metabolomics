---
name: longitudinal-cv-statistics-computation
description: Use when you have a run-by-feature intensity/RT matrix from repeated
  longitudinal DIA injections (e.g., a 31-day WMLD DIA series) and need to quantify
  per-precursor detection stability via CV statistics.
when_to_use_negative:
- Input is already a cleaned feature table with no missing values and only one run
  per condition (CV across runs is undefined)
- The goal is LC-MS file-level quality classification (qualified/unqualified) rather
  than per-precursor stability over time — use the iDIA-QC machine learning prediction
  workflow instead
- Data are DDA/SRM-based or from instrument types outside the supported set (timsTOF,
  TripleTOF, Orbitrap), where the DIA-NN quantification pipeline does not apply
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: iDIA-QC
  role: GUI/pipeline whose longitudinal summary outputs (html report and matrices)
    accompany the CV statistics; supports TripleTOF, Orbitrap, and timsTOF DIA files
  repo: https://github.com/guomics-lab/iDIA-QC
- name: DIA-NN
  role: Derives peptide-level quantification from converted mzXML files, applying
    1% FDR at peptide precursor and protein levels
- name: msConvert
  role: Converts vendor raw files (.wiff/.raw/.d) to mzXML before quantification and
    metric extraction
- name: TripleTOF
  role: Instrument platform that generated the 31-day longitudinal WMLD DIA runs whose
    precursor stability is assessed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-54871-1
    title: 'iDIA-QC: AI-empowered data-independent acquisition mass spectrometry-based
      quality control'
  source_content_hash: sha256:c9b9b6c174fc85aeefacee14d33580b6c10985dad1e778113cdce9561c3a3114
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/guomics-lab/iDIA-QC
  repo_url: https://github.com/guomics-lab/iDIA-QC
  tools:
  - DIA-NN
  - TripleTOF
  - iDIA-QC
  - msConvert
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# longitudinal-cv-statistics-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes per-feature coefficient of variation (CV = SD/mean × 100) across longitudinal DIA injections to quantify how stably selected peptide precursors (e.g., the 33 model-selected precursors on a TripleTOF 5600+) are detected over time, summarizing the CV distribution (median, mean, maximum, count above threshold).

## When to use

Use when you have a run-by-feature intensity/RT matrix from repeated longitudinal DIA injections (e.g., a 31-day WMLD DIA series) and need to quantify per-precursor detection stability via CV statistics.

## When NOT to use

- Input is already a cleaned feature table with no missing values and only one run per condition (CV across runs is undefined)
- The goal is LC-MS file-level quality classification (qualified/unqualified) rather than per-precursor stability over time — use the iDIA-QC machine learning prediction workflow instead
- Data are DDA/SRM-based or from instrument types outside the supported set (timsTOF, TripleTOF, Orbitrap), where the DIA-NN quantification pipeline does not apply

## Inputs

- Vendor raw DIA files from longitudinal injections (e.g., .wiff from a TripleTOF 5600+ 31-day WMLD series)
- DIA-NN (v1.7.15) peptide-level quantification with 1% FDR at precursor and protein levels
- List of model-selected peptide precursors (with retention times and intensities, no overlap between LC and MS panels)

## Outputs

- Run-by-precursor intensity/RT matrix for the selected precursors
- Per-precursor CV values (SD/mean × 100)
- CV summary (median, mean, count above threshold, maximum) written to cv_statistics.csv alongside the iDIA-QC html report and matrices

## How to apply

First obtain peptide-level quantification by converting vendor raw files (e.g., .wiff from the TripleTOF 5600+) to mzXML with msConvert and processing with DIA-NN (v1.7.15) under 1% FDR at both peptide precursor and protein levels. Clean the quantification matrix by removing features with more than 70% missing values and imputing remaining missing values with the per-feature minimum. Extract the retention times and intensities of the selected peptide precursors into a run-by-precursor intensity/RT matrix, then compute per-precursor CV (SD/mean × 100) across the longitudinal injections using pandas or R. Summarize the CV distribution (median, mean, number of precursors above threshold, maximum) and write it to a csv alongside the iDIA-QC html report and matrices.

## Related tools

- **iDIA-QC** (GUI/pipeline whose longitudinal summary outputs (html report and matrices) accompany the CV statistics; supports TripleTOF, Orbitrap, and timsTOF DIA files) — https://github.com/guomics-lab/iDIA-QC
- **DIA-NN** (Derives peptide-level quantification from converted mzXML files, applying 1% FDR at peptide precursor and protein levels)
- **msConvert** (Converts vendor raw files (.wiff/.raw/.d) to mzXML before quantification and metric extraction)
- **TripleTOF** (Instrument platform that generated the 31-day longitudinal WMLD DIA runs whose precursor stability is assessed)

## Examples

```
python -c "import pandas as pd; df=pd.read_csv('intensities.csv',index_col=0); cv=(df.std()/df.mean()*100); cv.to_csv('cv_statistics.csv')"
```

## Evaluation signals

- Reproduced median CV of ~2.9% and mean CV of ~3.5% across the 33 selected peptide precursors
- Exactly one precursor above 5.8% CV, with a maximum CV of ~11.2%
- Features with >70% missing values are removed and remaining NAs are imputed with the per-feature minimum before CV computation
- cv_statistics.csv contains median, mean, threshold exceedance count, and maximum CV, and sits alongside the iDIA-QC html report and matrices
- 1% FDR was applied at both peptide precursor and protein levels in the DIA-NN quantification feeding the CV calculation

## Limitations

- One precursor in the 33-precursor panel had a CV of 11.2%, exceeding the ~5.8% threshold — stability conclusions depend on the panel and instrument
- Generalization of the workflow to other QC and experimental samples requires further investigation, and the study could not cover all LC and MS instruments, especially newly emerging ones

## Evidence
