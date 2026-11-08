---
name: metabolite-annotation-class-categorization
description: Use when starting from a raw GCxGC-TOFMS detected-peak list (e.g. 691
  peaks) and you must document exclusion criteria, missing-value replacement rules,
  and the annotation-class breakdown of retained metabolites.
when_to_use_negative:
- The peak list is already filtered and annotated (e.g. a feature table with metabolite
  class assignments) - the audit would be redundant.
- The analysis goal is downstream biomarker modeling (OPLS-DA, ROC curve analysis)
  rather than documenting the filtering/annotation provenance.
- The data are from a platform whose quality-control and annotation criteria differ
  fundamentally from this GCxGC-TOFMS plasma protocol (e.g. LC-MS/MS with MS/MS spectral
  matching).
edam_operation: http://edamontology.org/operation_3561
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GCxGC-TOFMS
  role: Instrument platform whose two-dimensional gas chromatography time-of-flight
    mass spectrometry data produced the 691 detected peaks and the 195 metabolites
    to be filtered and categorized
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pntd.0006215
    title: Diagnostic metabolite biomarkers of chronic typhoid carriage
  source_content_hash: sha256:2d0e106b80afb8a8c26177bd130b016506fa07514a5b6641ae886a9dc6ae01a6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GCxGC-TOFMS
  edam_operation: http://edamontology.org/operation_3561
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# metabolite-annotation-class-categorization

## Summary

Documents the peak-exclusion and missing-value-replacement filtering that reduces a raw GCxGC-TOFMS detected-peak list (691 peaks) to a curated metabolite set (195 metabolites), and categorizes the retained metabolites by annotation class (putative annotation, assigned class, uncertain, unknown) in a reproducible audit table.

## When to use

Use when starting from a raw GCxGC-TOFMS detected-peak list (e.g. 691 peaks) and you must document exclusion criteria, missing-value replacement rules, and the annotation-class breakdown of retained metabolites.

## When NOT to use

- The peak list is already filtered and annotated (e.g. a feature table with metabolite class assignments) - the audit would be redundant.
- The analysis goal is downstream biomarker modeling (OPLS-DA, ROC curve analysis) rather than documenting the filtering/annotation provenance.
- The data are from a platform whose quality-control and annotation criteria differ fundamentally from this GCxGC-TOFMS plasma protocol (e.g. LC-MS/MS with MS/MS spectral matching).

## Inputs

- GCxGC-TOFMS detected-peak list from plasma cholecystectomy samples (691 peaks)
- Methods 'Data Processing And Metabolite Identification' text section
- S2 Table (supplementary) with metabolite filtering and annotation details
- QC sample RSD and run-order Pearson correlation metadata per peak
- PCA scores (centered/UV-scaled) for outlier detection

## Outputs

- peak_filter_audit.csv with one row per exclusion/processing criterion (criterion, threshold, action, count/percentage)
- Annotation-class breakdown table of the 195 retained metabolites (69 putative / 8 class-assigned / 18 uncertain / 100 unknown)
- Arithmetic validation report confirming 69+8+18+100=195 and percentage total = 100.0%

## How to apply

Extract the exclusion criteria from the Methods text and supplementary tables: split peaks, low-quality peaks or mass spectra, column bleed artifacts, internal standards, Pearson run-order correlation >|0.5|, QC RSD > 0.5 (with manual review of the RSD 0.3-0.5 range), and PCA centered/UV-scaled values > 4 (replaced at > 5). Record for each criterion whether it triggers exclusion or missing-value replacement, and the resulting counts: 691 detected peaks reduced to 195 retained metabolites, with 10 PCA outlier metabolites. Categorize the 195 retained metabolites into the reported annotation classes - 69/195 (35.4%) with putative annotation, 8/195 (4.1%) with an assigned metabolite class, 18/195 (9.2%) uncertain identity, and 100/195 (51.3%) unknown identity - as one row per criterion in an audit CSV. Validate arithmetically that 69 + 8 + 18 + 100 = 195 and that the percentages sum to 100.0%, flagging any inconsistency against the reported counts before saving peak_filter_audit.csv.

## Related tools

- **GCxGC-TOFMS** (Instrument platform whose two-dimensional gas chromatography time-of-flight mass spectrometry data produced the 691 detected peaks and the 195 metabolites to be filtered and categorized)

## Examples

```
python -c "import pandas as pd; counts={'putative':69,'class_assigned':8,'uncertain':18,'unknown':100}; assert sum(counts.values())==195; pd.DataFrame([counts]).to_csv('peak_filter_audit.csv')"
```

## Evaluation signals

- Sum check: 69 + 8 + 18 + 100 = 195 retained metabolites.
- Percentage check: 35.4 + 4.1 + 9.2 + 51.3 = 100.0%.
- Audit CSV contains one row for each of the exclusion criteria: split peaks, low-quality peaks/mass spectra, column bleed artifacts, internal standard, run-order correlation |r|>0.5, QC RSD > 0.5, PCA value > 4.
- Count reduction matches reported 691 detected peaks to 195 retained metabolites, with 10 PCA outlier metabolites flagged.
- Every annotation-class row carries both the fraction (e.g. 69/195) and its percentage (35.4%).

## Limitations

- The small sample size, acknowledged as 'The major limitation of this study was the small sample size', makes the filtered metabolite set fragile for downstream modeling.
- Annotation coverage is low: 100/195 (51.3%) metabolites remain unknown in identity, and the descriptive nature of the study limits physiological interpretation of the detected metabolite profiles.
- Filtering relies on manual review steps (QC RSD 0.3-0.5 range, PCA outlier inspection), which introduces operator-dependent subjectivity.

## Evidence

- [supplementary] models based on 195 metabolites generated from GCxGC-TOFMS analysis of plasma samples from patients in Nepal undergoing cholecystectomy: "models based on 195 metabolites generated from GCxGC-TOFMS analysis of plasma samples from patients in Nepal undergoing cholecystectomy"
- [discussion] The descriptive nature of the study makes it difficult to reveal what the detected metabolite profiles physiologically reflect.: "The descriptive nature of the study makes it difficult to reveal what the detected metabolite profiles physiologically reflect."
- [discussion] The major limitation of this study was the small sample size, which was difficult to overcome given the ethical, surgical, and diagnostic issues of identifying and confirming invasive Salmonella: "The major limitation of this study was the small sample size, which was difficult to overcome given the ethical, surgical, and diagnostic issues"
