---
name: signal-acquisition-error-rate-analysis
description: 'Decision support for choosing a signal-processing pipeline for chromatography–MS
  (GC–MS) data: when you must benchmark NPFimg against XCMS by counting erroneous
  signal acquisitions and marker identifications per dataset.'
when_to_use_negative:
- Input is already a validated marker/feature table with no raw chromatography–MS
  maps to reprocess, so no new signal-acquisition errors can occur
- Data are not chromatography–MS maps (e.g. proteomics spectra or single-cell data)
  outside NPFimg's GC–MS/LC–MS metabolomics scope
- You only need to run one pipeline and have no interest in the NPFimg-vs-XCMS reliability
  comparison
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: NPFimg
  role: Processes the two-dimensional MS map (m/z vs retention time) to identify marker
    features without conventional peak picking; its output error counts are compared
    with XCMS.
  repo: github.com/poomcj/NPFimg
- name: XCMS
  role: Conventional peak-detection and marker-identification baseline run on the
    same GC–MS data to benchmark against NPFimg's error rates.
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:38ecd17a189cb30c8dbec9ed4d73955748cdd0f4e1144e919feb0add7d3ca425
  source_papers:
  - doi: 10.1021/acs.analchem.1c03163
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NPFimg
  - XCMS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 3
---

# signal-acquisition-error-rate-analysis

## Summary

Quantifies and compares error rates of signal acquisition and marker identification between NPFimg, an image-processing/machine-learning method for two-dimensional chromatography–MS maps, and conventional peak-picking pipelines such as XCMS, to establish which method more reliably characterizes GC–MS signals.

## When to use

Decision support for choosing a signal-processing pipeline for chromatography–MS (GC–MS) data: when you must benchmark NPFimg against XCMS by counting erroneous signal acquisitions and marker identifications per dataset.

## When NOT to use

- Input is already a validated marker/feature table with no raw chromatography–MS maps to reprocess, so no new signal-acquisition errors can occur
- Data are not chromatography–MS maps (e.g. proteomics spectra or single-cell data) outside NPFimg's GC–MS/LC–MS metabolomics scope
- You only need to run one pipeline and have no interest in the NPFimg-vs-XCMS reliability comparison

## Inputs

- Chromatography–MS datasets, e.g. aroma odor and human breath GC–MS data used in the NPFimg case studies
- NPFimg marker-feature identification outputs from the two-dimensional MS map (m/z vs retention time)
- XCMS peak-detection and marker-identification outputs on the same GC–MS datasets

## Outputs

- Comparison table (e.g. xcms_vs_npfimg_error_rates.csv) with one row per method and dataset
- Signal-acquisition error rate per method per dataset (erroneous acquisition events / total evaluated signals)
- Marker-identification error rate per method per dataset (erroneous identifications / total evaluated markers)

## How to apply

Run NPFimg on the chromatography–MS datasets of interest (aroma odor and human breath GC–MS in the paper) so it discriminates analytes directly on the two-dimensional MS map (m/z vs retention time) without conventional peak picking, and run XCMS peak detection on the same data. For each method, count false/erroneous signal-acquisition events and erroneous marker identifications (e.g. parts-per-billion-level aroma/breath analytes misidentified or missed), then compute error rates as erroneous events divided by the total evaluated signals/markers. Assemble one row per method and dataset with signal-acquisition error rate and marker-identification error rate, writing the comparison to a CSV table. Judge correctness by checking that the table contains both methods, both datasets, and both metrics, and that NPFimg's error rates come out lower than XCMS's, as the paper reports.

## Related tools

- **NPFimg** (Processes the two-dimensional MS map (m/z vs retention time) to identify marker features without conventional peak picking; its output error counts are compared with XCMS.) — github.com/poomcj/NPFimg
- **XCMS** (Conventional peak-detection and marker-identification baseline run on the same GC–MS data to benchmark against NPFimg's error rates.)

## Examples

```
python npfimg_benchmark.py --method npfimg --input aroma_odor.ms --input human_breath.ms --compare xcms --out xcms_vs_npfimg_error_rates.csv
```

## Evaluation signals

- Comparison CSV contains rows for both methods (NPFimg, XCMS) and both datasets (aroma odor, human breath).
- Table includes both metrics: signal-acquisition error rate and marker-identification error rate for every method–dataset pair.
- NPFimg's signal-acquisition and marker-identification error rates are lower than XCMS's in every comparison row, matching the reported result.
- Error rates are computed as erroneous events divided by the total evaluated signals or markers, not as absolute counts.
- Marker identifications are reproducible down to the parts-per-billion analyte level in the GC–MS case studies.

## Limitations

- The paper demonstrates NPFimg on GC–MS aroma-odor and human-breath case studies; broader GC–MS/LC–MS applicability is stated as potential, not exhaustively validated.
- The README notes the repository documentation is incomplete ('For the other details, I will update soon'), so setup details may need the paper's full manual script.
- NPFimg is licensed under the MIT license on GitHub, so downstream use must respect that license.

## Evidence
