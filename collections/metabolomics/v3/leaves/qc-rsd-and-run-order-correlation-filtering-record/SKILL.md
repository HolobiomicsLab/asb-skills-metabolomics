---
name: qc-rsd-and-run-order-correlation-filtering-record
description: Use when processing GCxGC-TOFMS metabolomics peak tables where peaks
  must be filtered or replaced based on split/low-quality spectra, Pearson run-order
  correlation, QC sample RSD, or PCA-derived outlier values before downstream modeling.
when_to_use_negative:
- Input is already a curated feature table that has passed QC filtering (re-applying
  these thresholds would double-filter)
- The dataset lacks injected QC samples or run-order metadata, making RSD and run-order
  correlation tests inapplicable
- Downstream analysis targets unfiltered exploratory peak inspection rather than supervised
  metabolite classification
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: GCxGC-TOFMS
  role: Source instrument producing the raw two-dimensional gas chromatography time-of-flight
    mass spectrometry peak list that the QC filtering criteria are applied to
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pntd.0006215
    title: Diagnostic metabolite biomarkers of chronic typhoid carriage
  source_content_hash: sha256:3fea097f329ac6cad9d528533a541877f29ac6d7ff53a33d7cdcbde2d0c442eb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GCxGC-TOFMS
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# qc-rsd-and-run-order-correlation-filtering-record

## Summary

Records the QC-driven exclusion and missing-value replacement criteria that reduce 691 detected GCxGC-TOFMS peaks to 195 retained plasma metabolites, including run-order correlation, QC RSD, and PCA outlier thresholds.

## When to use

Use when processing GCxGC-TOFMS metabolomics peak tables where peaks must be filtered or replaced based on split/low-quality spectra, Pearson run-order correlation, QC sample RSD, or PCA-derived outlier values before downstream modeling.

## When NOT to use

- Input is already a curated feature table that has passed QC filtering (re-applying these thresholds would double-filter)
- The dataset lacks injected QC samples or run-order metadata, making RSD and run-order correlation tests inapplicable
- Downstream analysis targets unfiltered exploratory peak inspection rather than supervised metabolite classification

## Inputs

- Raw GCxGC-TOFMS peak table from plasma samples (691 detected peaks)
- Injection order metadata for run-order correlation testing
- QC sample replicate abundance matrix
- Peak quality annotations (split peak, low-quality peak, mass spectrum quality)

## Outputs

- Filtered metabolite matrix of 195 retained metabolites
- Audit record listing each exclusion/replacement criterion, threshold, action, and resulting count
- Annotation class breakdown (69 putative / 8 class-assigned / 18 uncertain / 100 unknown)

## How to apply

Parse the raw GCxGC-TOFMS peak list (e.g. from plasma cholecystectomy samples, 691 detected peaks) and apply a sequential exclusion pipeline: remove split peaks, low-quality peaks or mass spectra, column bleed artifacts and internal standards; exclude peaks whose abundance correlates with run order (Pearson |r| > 0.5); then evaluate QC sample reproducibility, excluding peaks with QC RSD > 0.5 and manually reviewing those in the 0.3–0.5 band. Retained peaks with PCA (centered, UV-scaled) values greater than 4 are inspected and values greater than 5 are treated as outliers and replaced. Record every threshold, action (exclusion or replacement), and the resulting counts so that the retained set (195 metabolites) can be audited against its annotation breakdown (69 putative, 8 class-assigned, 18 uncertain, 100 unknown).

## Related tools

- **GCxGC-TOFMS** (Source instrument producing the raw two-dimensional gas chromatography time-of-flight mass spectrometry peak list that the QC filtering criteria are applied to)

## Examples

```
Rscript peak_filter_audit.R --input gcxgc_peaks.csv --runorder order.csv --qcrsd 0.5 --pearson 0.5 --pcaReplace 5
```

## Evaluation signals

- Check that the retained metabolite count matches the reported 195 and that the annotation classes sum to 195 (69 + 8 + 18 + 100)
- Verify that every excluded or replaced peak is traceable to a named criterion (split peak, low-quality mass spectrum, run-order correlation |r| > 0.5, QC RSD > 0.5, PCA value > 5)
- Confirm that peaks with QC RSD between 0.3 and 0.5 were manually reviewed rather than automatically excluded
- Confirm that PCA values in the 4–5 range were inspected but not replaced unless they exceeded 5
- Confirm the final filtered matrix is suitable for downstream supervised pattern recognition modeling (OPLS-DA) without QC-driven artifacts

## Limitations

- The study's small sample size limits the statistical power of the QC filtering thresholds
- Peaks in the QC RSD 0.3–0.5 band require manual review, introducing operator subjectivity
- No independent validation cohort was available to confirm that filtering choices did not remove informative metabolites

## Evidence

- [methods] Filtering criteria: split/low-quality peaks, run-order correlation, QC deviation: [evidence span withheld — anchor status: not_found]
- [results] Retention breakdown of 195 metabolites: [evidence span withheld — anchor status: not_found]
- [methods] Run-order and QC thresholds: [evidence span withheld — anchor status: not_found]
- [methods] PCA outlier replacement threshold: [evidence span withheld — anchor status: not_found]
